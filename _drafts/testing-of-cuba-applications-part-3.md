---
layout: post
title: Testing of CUBA applications
subtitle: Part 3 - Services & other middleware components
description:
modified: 2016-12-19
tags: [cuba, Testing]
image:
  feature: testing-of-cuba-applications-part-3/feature.jpg
---


The next part of this series is designated towards the other elements of the middleware.
We'll look at how to test services and other beans. While doing that we will try to climb
the testing pyramid and try to create integration tests as an alternative to the unit tests we saw in the second part.

<!-- more -->

# Testing services

The first question that comes up when thinking about how to test services might be, what is different to
testing Entities? The basic answer is simple: nothing. When we want to test services in a unit test, there is nothing special about it. In fact it is even a little bit easier to do so, because Dependency Injection in services works like a charme.

Let's look at a [simple case](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/88ed9db13ed37698de751b3d60df6046adb7e958/modules/core/test/com/company/ceta/service/OrderInformationServiceBeanSpec.groovy):


{% highlight groovy %}

@Service(OrderInformationService.NAME)
public class OrderInformationServiceBean implements OrderInformationService {

    @Override
    Order findLatestOrder(Customer customer) {
        customer.orders?.max {it.orderDate}
    }
}

class OrderInformationServiceBeanSpec extends Specification {

    OrderInformationService service

    def setup() {
        service = new OrderInformationServiceBean()
    }

    def "findLatestOrder returns the order with the latest orderDate"() {

        given: "there are two orders"
        def today = new Date()
        def yesterday = today - 1
        Order yesterdaysOrder = new Order(orderDate: yesterday)
        Order todaysOrder = new Order(orderDate: today)

        and: "there is one customer that holds those orders"
        Customer customer = new Customer(orders: [todaysOrder, yesterdaysOrder])

        when: "we search for the latest Order"
        Order latestOrder = service.findLatestOrder(customer)

        then: "we get back the todays order as the latest one"
        latestOrder == todaysOrder
    }
}

{% endhighlight %}

This implementation is a simple case, because the service doesn't do anything special. It has no dependencies. Therefore it's easy to test. Instead of fetching the order information from the Database it assumes that the orders are already filled within the Customer instance.

So let's have another look how it would work if we want to retrieve the data from the database and let the database do the job.

The implementation of this would probably look like this:

{% highlight groovy %}

@Service(OrderInformationService.NAME)
public class OrderInformationServiceBean implements OrderInformationService {

    @Inject
    DataManager dataManager

    @Override
    Order findLatestOrder(Customer customer) {

        def sqlQueryString = 'select e from ceta$Order e where e.customer.id = :customerId order by e.orderDate desc'

        LoadContext loadContext = LoadContext.create(Order)
                .setQuery(
                LoadContext.createQuery(sqlQueryString)
                        .setParameter('customerId', customer.id)
                        .setMaxResults(1)
        )

        dataManager.load(loadContext)
    }
}

{% endhighlight %}

The `DataManager` from CUBA is used to trigger the database. Instead of iterating over the orders collection in groovy,
we would create the equivalent SQL string and create a `LoadContext` for this that will be passed to the `dataManager` instance.

In this case we definitivly need to adjust the unit test. When we run the test again, we see the following:

{% highlight java %}
java.lang.NullPointerException
	at com.haulmont.cuba.core.global.AppBeans.get(AppBeans.java:61)
	at com.haulmont.cuba.core.global.LoadContext.<init>(LoadContext.java:88)
	at com.haulmont.cuba.core.global.LoadContext.create(LoadContext.java:63)
	at com.company.ceta.service.OrderInformationServiceBean.findLatestOrder(OrderInformationServiceBean.groovy:22)
	at com.company.ceta.service.OrderInformationServiceBeanSpec.findLatestOrder returns the order with the latest orderDate(OrderInformationServiceBeanSpec.groovy:30)
{% endhighlight %}

It is pretty much the same situation as we've seen in the [last](https://www.road-to-cuba-and-beyond.com/testing-of-cuba-applications-part-2/) blog post where we tried to use `AppBeans.get()` directly in the code of the Customer.

### Mocking through subclass and dependency injection
So what can we do in this case? Upps, did i above said: DI works like a charme in services? Right, so for `dataManager` this is true, but we have another dependency here: `LoadContext.create()`. This one is not created through DI, therefore we are at the same situation as in the entites case.

To solve this, we can use both approaches: Mock through DI (the `dataManager`) as well as the Subclass (the `LoadContext`).

In order to allow that we have to change the implementation slightly to enable useful and minimally invasive subclassing.


{% highlight groovy %}

@Override
Order findLatestOrder(Customer customer) {

    LoadContext.Query query = createCustomerQuery(customer)
    LoadContext loadContext = createOrderLoadContext().setQuery(query)

    dataManager.load(loadContext)
}

protected LoadContext.Query createCustomerQuery(Customer customer) {
    def sqlQueryString = 'select e from ceta$Order e where e.customer.id = :customerId order by e.orderDate desc'
    LoadContext.createQuery(sqlQueryString)
            .setParameter('customerId', customer.id)
            .setMaxResults(1)
}

protected LoadContext<Order> createOrderLoadContext() {
    LoadContext.create(Order)
}
{% endhighlight %}

With this change, we create a subclass that will mock the `createOrderLoadContext` method:

{% highlight groovy %}
class OrderInformationServiceBeanWithMockableDependencies extends OrderInformationServiceBean {

    LoadContext orderLoadContext

    @Override
    protected LoadContext<Order> createOrderLoadContext() {
        orderLoadContext
    }
}
{% endhighlight %}

I created a few unit tests that will check different scenarios in the OrderInformationServiceBean and its interaction with the `loadContext` and `dataManager`. You can find the complete tests [here](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/d9d9b09287a8b7a38254bb671e2aad2ae607eb3a/modules/core/test/com/company/ceta/service/OrderInformationServiceBeanSpec.groovy). Let's look at one example:

{% highlight groovy %}
class OrderInformationServiceBeanSpec extends Specification {

    OrderInformationService service
    LoadContext loadContext
    DataManager dataManager

    def setup() {
        loadContext = Mock(LoadContext)
        dataManager = Mock(DataManager)
        service = new OrderInformationServiceBeanWithMockableDependencies(
                dataManager: this.dataManager,
                orderLoadContext: loadContext
        )
    }

    def "findLatestOrder passes the correct SQL query to the loadContext"() {

        when:
        service.findLatestOrder(new Customer())

        then: 'the querString has been set in the query'
        1 * loadContext.setQuery({ LoadContext.Query query ->
            query.queryString == 'select e from ceta$Order e where e.customer.id = :customerId order by e.orderDate desc'
        })
    }
}
{% endhighlight %}

I mocked the loadContext through the subclass. In the test i can define expectations on the `loadContext` instance. In the code the loadContext will get passed a particular Query object. In this case, i used a spock feature to define an expectation in my mock. Instead of defining that the query object is a particular instance, i define a groovy closure that is invoked and the real instance is passed to it. The closure has to return a boolean value that describes if the instance is right or wrong.

So it pretty much reads like this:
"the setQuery method of the loadContext gets called once with a query parameter, that has a query string which looks like this: 'select ...'"

When you look at the [whole test class](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/d9d9b09287a8b7a38254bb671e2aad2ae607eb3a/modules/core/test/com/company/ceta/service/OrderInformationServiceBeanSpec.groovy) you will notice that the test are totally different kind than we saw in the first example. Here are just the descriptions of the test cases:

{% highlight groovy %}
def "findLatestOrder passes the correct SQL query to the loadContext"()  { /*...*/ }
def "findLatestOrder sets the parameter customerId in the query"() { /*...*/ }
def "findLatestOrder sets the maxResults to one, to just get the first order of the SQL query"() { /*...*/ }
def "findLatestOrder uses the loadContext to pass it into the dataManager"() { /*...*/ }
{% endhighlight %}

Instead of checking anything about the customer and it's order, i created interaction descriptions of the OrderInformationService with its dependencies.

### The problem with too much mocking
When you look at the tests descriptions, it actually reads no that well. Everything i described there is true, but it is on a different level compared to the first test that we created before doing it via the database: `findLatestOrder returns the order with the latest orderDate`. This test, from it's description, was more like a black-box test, where the implementation is totally irrellevant to the test. The unit tests with the `DataManager` are more like a white-box tests where the test is totally aware of the internals of the implementation of the service.
