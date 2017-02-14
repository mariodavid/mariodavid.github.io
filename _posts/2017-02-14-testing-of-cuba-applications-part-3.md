---
layout: post
title: Testing of CUBA applications
subtitle: Part 3 - Services & Integration Tests
description:
modified: 2017-02-14
tags: [cuba, Testing]
image:
  feature: testing-of-cuba-applications-part-3/feature.jpg
---


The next part of this series is designated towards the other elements of the middleware.
We'll look at how to test services and other middleware components. While doing that we will try to climb
the testing pyramid and try to create integration tests as an alternative to the unit tests we saw in the second part.

<!-- more -->

# Testing services

The first question that comes up when thinking about how to test services might be: what is different to
testing Entities? The basic answer is simple: nothing. When we want to test services in a unit test, there is nothing special about it. In fact it is even a little bit easier to do so, because Dependency Injection in services works like a charm.

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

So let's have another look on how it would work if we want to retrieve the data from the database and let the database do the job.

The implementation of this case would probably look like this:

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

The <code>DataManager</code> from CUBA is used to trigger the database. Instead of iterating over the orders collection in groovy,
we would create the equivalent SQL string and create a <code>LoadContext</code> for this that will be passed to the <code>dataManager</code> instance.

In this case we definitely need to adjust the unit test. When we run the test again, we see the following:

{% highlight java %}
java.lang.NullPointerException
	at com.haulmont.cuba.core.global.AppBeans.get(AppBeans.java:61)
	at com.haulmont.cuba.core.global.LoadContext.<init>(LoadContext.java:88)
	at com.haulmont.cuba.core.global.LoadContext.create(LoadContext.java:63)
	at com.company.ceta.service.OrderInformationServiceBean.findLatestOrder(OrderInformationServiceBean.groovy:22)
	at com.company.ceta.service.OrderInformationServiceBeanSpec.findLatestOrder returns the order with the latest orderDate(OrderInformationServiceBeanSpec.groovy:30)
{% endhighlight %}

It is pretty much the same situation as we've seen in the [last](https://www.road-to-cuba-and-beyond.com/testing-of-cuba-applications-part-2/) blog post where we tried to use <code>AppBeans.get()</code> directly in the code of the Customer.

### Mocking through subclass and dependency injection
So what can we do in this case? Upps, did I above said: DI works like a charm in services? Right, so for <code>dataManager</code> this is true, but we have another dependency here: <code>LoadContext.create()</code>. This one is not created through DI, therefore we are in the same situation as in the entities case.

To solve this, we can use both approaches: Mock through DI (the <code>dataManager</code>) as well as the Subclass (the <code>LoadContext</code>).

In order to allow that, we have to change the implementation slightly to enable useful and minimally invasive subclassing.


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

With this change, we create a subclass that will mock the <code>createOrderLoadContext</code> method:

{% highlight groovy %}
class OrderInformationServiceBeanWithMockableDependencies extends OrderInformationServiceBean {

    LoadContext orderLoadContext

    @Override
    protected LoadContext<Order> createOrderLoadContext() {
        orderLoadContext
    }
}
{% endhighlight %}

I created a few unit tests that will check different scenarios in the OrderInformationServiceBean and its interaction with the <code>loadContext</code> and <code>dataManager</code>. You can find the complete tests [here](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/d9d9b09287a8b7a38254bb671e2aad2ae607eb3a/modules/core/test/com/company/ceta/service/OrderInformationServiceBeanSpec.groovy). Let's look at one example:

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

I mocked the loadContext through the subclass. In the test I can define expectations on the <code>loadContext</code> instance. In the code the loadContext will get passed a particular Query object. In this case, I used a spock feature to define an expectation in my mock. Instead of defining that the query object is a particular instance, I define a groovy closure that is invoked and the real instance is passed to it. The closure has to return a boolean value that describes if the instance is right or wrong.

So it pretty much reads like this:
"the setQuery method of the loadContext gets called once with a query parameter, that has a query string which looks like this: 'select ...'"

When you look at the [whole test class](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/d9d9b09287a8b7a38254bb671e2aad2ae607eb3a/modules/core/test/com/company/ceta/service/OrderInformationServiceBeanSpec.groovy) you will notice that the tests are of totally different kind than we saw in the first example. Here are just the descriptions of the test cases:

{% highlight groovy %}
def "findLatestOrder passes the correct SQL query to the loadContext"()  { /*...*/ }
def "findLatestOrder sets the parameter customerId in the query"() { /*...*/ }
def "findLatestOrder sets the maxResults to one, to just get the first order of the SQL query"() { /*...*/ }
def "findLatestOrder uses the loadContext to pass it into the dataManager"() { /*...*/ }
{% endhighlight %}

Instead of checking anything about the customer and it's order, I created interaction descriptions of the OrderInformationService with its dependencies.

### The problem with too much mocking
When you look at the tests descriptions, it actually reads not that well. Everything I described there is true, but it is on a different level compared to the first test that we created before doing it via the database: <code>findLatestOrder returns the order with the latest orderDate</code>. This test, from its description, was more like a black-box test, where the implementation is totally irrellevant to the test.

The unit tests with the <code>DataManager</code> are more like a white-box tests where the test is totally aware of the internals of the implementation of the service. Generally it is a good idea not to care about the internals of the implementation in the test. But are they really internals? Some of them are, but a lof of them are actually interactions with other dependencies.

What it boils down to is somewhat related to the distinction between [Chicago and London style of TDD](http://softwareengineering.stackexchange.com/questions/123627/what-are-the-london-and-chicago-schools-of-tdd). I will not go into details of this, but it basically says: Where the Chicago style prefers more the hidden implementation part (simplified), the London style sees the interaction of the dependencies as the description of the behavior of the system. You can think of a OO program as a set of cells that are interacting with each other. The description of the cell system can be derived from the communication between the cells, meaning that your tests can describe the messages between the objects instead of the result of a certain cell.

Nontheless, for now we assume that these tests are too much detailed and don't fulfil a particular purpose they were intended to, which is: [unit test should serve as documentation](http://softwareengineering.stackexchange.com/questions/154615/are-unit-tests-really-used-as-documentation) (because the test descriptions tell how the method works, but not what it does). So how can we get out of it?

# Integration tests in middleware
One way of going back to a test description that reads more like the first one is to create integration tests for that purpose. Why? Because then we will set up the infrastructure beforehand, then we create the real customer instance etc. and run the system much more like in a real world scenario, where the <code>dataManager</code> is in place and does its job against a real database.

So let's take a look on how we can create an Integration Test in CUBA.

In a middleware integration test in CUBA the following things are available:

* access to a real database
* CUBA platform spring beans as well as your own spring beans and services are available

#### Create a integration test runtime environment

First of all we need a little runtime environment that starts the Spring application context and creates the database.
The [official docs on middleware integration testing](https://doc.cuba-platform.com/manual-6.4/integration_tests_mw.html) go into this in detail, but we will go through that to some extend.

In our tests, we create an object called <code>container</code> that is responsible for the runtime environment. This container is a subclass of [TestContainer](https://github.com/cuba-platform/cuba/blob/master/modules/core/test/com/haulmont/cuba/testsupport/TestContainer.java). I called it [IntegrationTestContainer](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/master/modules/core/test/com/company/ceta/service/integration/container/IntegrationTestContainer.groovy) and it basically looks like this:

{% highlight groovy %}
class IntegrationTestContainer extends TestContainer {

    // ...

    IntegrationTestContainer() {
        super()
        appPropertiesFiles = Arrays.asList(
                // List the files defined in your web.xml
                // in appPropertiesConfig context parameter of the core module
                'cuba-app.properties',
                'app.properties',
                'ceta-test-app.properties'
        )

        dbDriver = 'org.hsqldb.jdbcDriver'
        dbUrl = 'jdbc:hsqldb:mem:cetaTestDb'
        dbUser = 'sa'
        dbPassword = ''
    }

    // ...
}
{% endhighlight %}

It is just a configuration of the application properties files that I want to include as well as the database configuration. In this case I choosed to create an in-memory hsqldb instead of a persistent database as described in the docs. It removes the burden of having to call <code>gradlew createTestDb</code> before running the test, but on the other hand it is not possible to have a look at the database after test execution for debugging purposes.


#### Common integration test superclass

Next, in the actual test case we need to create an instance of this container. In order to not create a new container for every test case, I used the idea of the Singleton pattern of the docs. The reference of that container instance is shared across all specs in spock via the <code>@Shared</code> annotation (see also the [whole class](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/master/modules/core/test/com/company/ceta/service/integration/container/ContainerIntegrationSpec.groovy)):

{% highlight groovy %}
class ContainerIntegrationSpec extends Specification {

    @Shared
    IntegrationTestContainer container = IntegrationTestContainer.Common.INSTANCE

    // ...

}
{% endhighlight %}


The <code>ContainerIntegrationSpec</code> is going to be our superclass of our test cases to reduce boilerplate code in the tests.

So let's have a look at an [actual integration test](https://github.com/mariodavid/cuba-example-testing-artifacts/blob/master/modules/core/test/com/company/ceta/service/integration/OrderInformationServiceBeanIntegrationSpec.groovy):

{% highlight groovy %}
class OrderInformationServiceBeanSpec extends ContainerIntegrationSpec {

    OrderInformationService service

    def setup() {
        service = AppBeans.get(OrderInformationService)
    }

    def "findLatestOrder returns the order with the latest orderDate"() {
        given:
        Customer customer = metadata.create(Customer)
        def todaysOrder = metadata.create(Order)
        customer.setOrders([todaysOrder] as Set)

        when:
        Order latestOrder = service.findLatestOrder(customer)
        then:
        latestOrder == todaysOrder
    }

}
{% endhighlight %}


What we see here is that we are pretty close to the actual unit test implementation. The only difference is that we use the entity manager to persist the instances and instead of creating the service via new, the real service instance is used.

With this, we are able to generally execute code in a fairly real world like scenario. The execution time of the integration tests is obviously higher, because it has to create the spring application context and start the database. But as we use the Singleton approach for our container instance, it will only affect the first integration test that gets executed, so it will take something like 100ms instead of 1ms to run it. The next integration test though are usually run within something like 1-10ms depending on what is done in the test.


##  The right balance between Unit and Integration tests
Since integration tests seem more appropriate in this case, it is not always the case. Do you remember the [testing pyramid from the first blog post](https://www.road-to-cuba-and-beyond.com/testing-of-cuba-applications-part-1/)? I talked about the ratio between the different test phases a little. Before we go and test everything that we have to do with the database in an integration test, it might be valuable to look at the alternatives. The above described problem, that we needed too much mocking in the unit test, might be a sign that we have a design problem.

It may be (and in fact I would argue in this case it is), that we need an abstraction layer between the CUBA APIs and our own business logic. Michael Feathers talks about this in his book "Working effectively with legacy code" about the fact that "my whole application is all API calls". The idea to prevent that situation of too much mocking entirly is to create an abstraction of the interface with some kind of a [wrapper](http://svengrand.blogspot.de/2008/05/new-xunit-test-pattern.html) which additionaly gets optimised for unit testing.

Let's have a look at what that would look like. The wrapper might have the following imaginary interface:

{% highlight groovy %}
public interface AppDataManager {

    String NAME = "ceta_AppDataManager";

    <E extends BaseUuidEntity> E loadByReference(Class<E> entityClass, String propertyPath, BaseUuidEntity reference, Map<String, String> orderBy);
}
{% endhighlight %}

The method <code>loadByReference</code> not only wrapps the API, but also it puts a common use case on top of it. With this, the the <code>OrderInformationService</code> implementation implodes like this:

{% highlight groovy %}
@Service(OrderInformationService.NAME)
public class OrderInformationServiceBean implements OrderInformationService {

    @Inject AppDataManager appDataManager

    @Override
    Order findLatestOrder(Customer customer) {
        appDataManager.loadByReference(Order, 'customer', customer, ['orderDate': 'desc'])
    }

}
{% endhighlight %}

Now it is very easy to go back to a unit test, that will mock the <code>appDataManager</code> and track its call.
With that we can push the integration test part into the underlying db access layer (AppDataManager) and with that are not forced to do integration test for the whole business logic.

This solution might seem to be a little overkill, and probably it is, but I hope you got the point of what I'm trying to say.

With that we have gone through the most of the middleware integration test scenarios as well as how to test services in general. I hope you enjoyed it. If you have any questions or different opinions around that topic, just leave a comment below.
