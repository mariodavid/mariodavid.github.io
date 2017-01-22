---
layout: post
title: Testing of CUBA applications
subtitle: Part 2 - Testing of CUBA Entites
description:
modified: 2017-01-15
tags: [cuba, Testing, Entity]
image:
  feature: testing-of-cuba-applications-part-2/feature.jpg
  feature_source: https://pixabay.com/de/kamel-w%C3%BCste-pyramide-sonne-wolken-1574449/
---
In the second part of the Testing article series, we'll look at the CUBA side of things. We will go through different artifacts of the platform and check how we can test these artifacts. In this blog post we will start with Entities and encounter Mocking as we go through the different examples.

<!-- more -->

After we saw how basic unit testing works in the first part, we are basically able to write unit tests. But the production code that wasn't the hardest code we could imagine. To recap, here's the Listing of the method once again:

{% highlight groovy %}
class Customer {

    // ...
    List<Order> orders = []

    boolean isGoodCustomer() {
        orders?.size() > 0
    }
}
{% endhighlight %}

When you look at, it seems to be a fairly easy method - and indeed, it is. There are not a lot of if / else statements that complicate the code, no loops. The Cyclomatic complexity is 2 which is fairly low.

Additionally the method only has one external dependency that it uses in order to fulfill its purpose - which is [List](https://docs.oracle.com/javase/7/docs/api/java/util/List.html). Although it is a list of orders, nothing of the Order class is used to achieve the goal, so we will not count it here.

Generally the following situations make a class or a method harder to test:

* external dependencies and interactions with those
* static method calls
* creating object instances through <code>new</code>
* expecting of a lot of object setup before executing
* ...

We will encounter different strategies for these situations as we go with the examples. So let's dive into it.

When we look at a typical CUBA application, we will see different artifact types. These types occur due to different business logic that we want to implement. Beginning with the Backend which contains the Entities as well as corresponding entity listeners. In the next articles we will go through the other artifact types. Services that are normally executed in the backend as well. On the frontend we have Controllers that are responsible for the UI logic. So let's start with the easiest ones: Entities.

## Testing CUBA Entities

Testing Entities is easy and problematic at the same time. It is easy because normally there is not much to test in Entities. On the other hand, since there is no dependency injection available for Entities, it can sometimes be a little harder to extract out the usages of other beans.

Let's look at how a normal Entity looks like when it get's generated via studio:

{% highlight java %}
public class Customer extends StandardEntity {
    private static final long serialVersionUID = 3748864199667741996L;

    @Column(name = "NAME")
    protected String name;

    @Column(name = "FIRST_NAME")
    protected String firstName;

    @Column(name = "TYPE_", nullable = false)
    protected String type;

    @Composition
    @OnDelete(DeletePolicy.CASCADE)
    @OneToMany(mappedBy = "customer")
    protected Set<Order> orders;

    public void setOrders(Set<Order> orders) { this.orders = orders;}
    public Set<Order> getOrders() { return orders; }

    public void setName(String name) {this.name = name;}
    public String getName() {return name;}

    public void setFirstName(String firstName) {this.firstName = firstName;}
    public String getFirstName() {return firstName;}

    public void setType(CustomerType type) {
        this.type = type == null ? null : type.getId();
    }

    public CustomerType getType() {
        return type == null ? null : CustomerType.fromId(type);
    }
}
{% endhighlight %}

### Don't test generated code

So when we look at what we can test in this class, the answer is: nothing. Why? Well, it's all generated code. You can assume that anyone from the CUBA team has already tested this code generator and it works. If not you can't do anything about it either. Additionally the code consists mostly out of getters and setters. We don't really want to test this kind of code anyway ([on SO](http://stackoverflow.com/questions/6197370/should-unit-tests-be-written-for-getter-and-setters) is an explanation why).

### The first piece of business logic
Let's put that code beside and make a real example. In this case, we add a <code>getCaption</code> method which is used as the NamePattern of the Customer entity. <code>getCaption</code> combines <code>firstName</code> and <code>name</code> and additionally puts the customer type at the end of it.


{% highlight java %}
@NamePattern("#getCaption|firstName,name,type")
@Table(name = "CETA_CUSTOMER")
@Entity(name = "ceta$Customer")
public class Customer extends StandardEntity {

    // ...

    public String getCaption() {
        return getFullName() + " (" + type + ")";
    }

    private String getFullName() {
        return Joiner.on(", ").skipNulls().join(name, firstName);
    }
}
{% endhighlight %}

When we look at the method we want to test, we can come up with the following test cases:

* Happy Path: getCaption combines the first name and the last name as well as the type
* getCaption only uses name if first name is not present
* getCaption only prints type if it is present

Having looked at the examples from the first blog post, you should be able to create a test like this:

{% highlight groovy %}
class CustomerSpec extends Specification {

    Customer customer

    void setup() {
        customer = new Customer()
    }

    def "getCaption combines the first name and the last name"() {

       given:
       customer.firstName = "Jack"
       customer.name = "Jones"

       expect:
       customer.getCaption().startsWith "Jones, Jack"
   }

   def "getCaption prints the type if set after the name in parentheses"() {

       given:
       customer.firstName = null
       customer.name = "Jones"
       customer.type = CustomerType.LOYAL

       expect:
       customer.getCaption().endsWith "(LOYAL)"
   }

   def "getCaption only uses name if first name is not present"() {

       given:
       customer.firstName = null
       customer.name = "Jones"
       customer.type = CustomerType.LOYAL

       expect:
       customer.getCaption() == "Jones (LOYAL)"
   }
}
{% endhighlight %}

The first test checks the happy path "getCaption combines the first name and the last name as well as the type". It sets the name and firstName, which is the setup phase. After that it executes the getCaption method and verifies the output. In this case it should contain the firstName and the lastName separated by a comma.

### One assertion per test

As you might have noticed, instead of checking the whole String "Jones, Jack (LOYAL)" I decided to split the assertions into two tests. The first checks if the names have been combined correctly and the second test case checks if the type is printed in parentheses at the end of it.

It could have been done in one test, but I like the idea of doing [one assertion in one test](http://programmaticallyspeaking.com/one-assertion-per-test-please.html), because these assertion contains two different parts.

Imagine what would happen if used the initial stated test case "getCaption combines the first name and the last name as well as the type" which would look like this:


{% highlight groovy %}
def "getCaption combines the first name and the last name as well as the type"() {

    given:
    customer.firstName = "Jack"
    customer.name = "Jones"
    customer.type = CustomerType.LOYAL

    expect:
    customer.getCaption() == "Jones, Jack (LOYAL)"
}
{% endhighlight %}



Let's imagine in the ongoing implementation the customer changes their mind and wants to see something more along those lines: "Jones, Jack - LOYAL". You change the code accordingly and with this break the tests. When we now look at the test report, we will see the following output:

<figure class="center">
	<a href="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-broken-dash.png"><img src="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-broken-dash.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-broken-dash.png" title="Breaking the test does not directly point to the problem becase of multiple responsibilities">Breaking the test does not directly point to the problem becase of multiple responsibilities</a></figcaption>
</figure>

The problem here is that although the description points you slightly in the right direction, it doesn't do it as specific as it could. Why is this test failing? Because it doesn't use the first name? Does the firstName does not get combined with the name? Or is the type missing? Additionally the message doesn't even say anything about parentheses, because when we would describe in this way, the text would get longer and longer.

This not being as precise as it could be will increase the maintenance costs of that change.

So instead, let's create the two distinguished test cases with the stricter defined name and assertion (like in the original code above). When we look at the test result in case we put the different responsibilities into two tests we will see the difference.

<figure class="center">
	<a href="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-broken-dash-two-tests.png"><img src="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-broken-dash-two-tests.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-broken-dash-two-tests.png" title="Breaking the test directly points you to the problem with the parentheses and excludes the part with first name and last name">Breaking the test directly points you to the problem with the parentheses and excludes the part with first name and last name</a></figcaption>
</figure>

It has certain additional benefits to do one assertion per test, but it's more of a personal taste than a must have. So let's put this somewhat advanced topic beside for now.

### Unit tests will reveal bugs
When we implement the third test case "getCaption only prints type if it is present", we will encounter a bug in our software. Have a look at this test:

{% highlight groovy %}
def "getCaption doesn't contain type with parentheses if the type is missing"() {

    given:
    customer.name = "Jones"
    customer.type = null

    expect:
    customer.getCaption() == "Jones"
}
{% endhighlight %}

This test will be red, because the implementation doesn't yet support this behavior. Good that we have tests, isn't it?

Let's fix this with the following if statement:

{% highlight java %}
public String getCaption() {

    String caption = getFullName();
    if (type != null) {
        caption += " (" + type + ")";
    }
    return caption;
}
{% endhighlight %}

This is pretty common. The reason is that when you think about the edge cases in the test case creation phase, you probably do more so compared to the implementation phase. This is another indicator why TDD is a good thing, but that's another story.

With this, we have seen a fairly common method in an entity. But sometimes it's not only a calculation on local attributes. Therefore let's expand the requirement so that it will get a little bit more complicated.

### Using localized ENUM values in the output

Instead of having the source code representation of the customer type (which is "LOYAL", "NEW" etc.), we want to see the localized version of it. So in case of English it would be "loyal", "new" etc.

In order to achieve this, CUBA has an article in the docs about the topic of [enum localization](https://doc.cuba-platform.com/manual-6.3/enum_localization.html). We need the <code><a href="https://github.com/cuba-platform/cuba/blob/master/modules/global/src/com/haulmont/cuba/core/global/Messages.java#L90">Messages</a></code> interface and especially the <code>String getMessage(Enum caller);</code> method.

#### Using external dependencies in Entities

So how do we get a reference to an instance of a messages interface? Normally we would use dependency injection like this (as it states in the docs about [bean usage](https://doc.cuba-platform.com/manual-6.3/managed_beans_usage.html)):

{% highlight java %}
public class Customer extends StandardEntity {
    @Inject
    protected Messages messages;

    public String getCaption() {
        //messages.doYourStuff()
    }

}
{% endhighlight %}

Unfortunately, dependency injection doesn't work for entities. So we have to use the other approach suggested by the docs: <code>AppBeans.get(MyDependency)</code>. Below you'll see the code with the localized type String which is fetched from the Messages interface.

{% highlight java %}
public class Customer extends StandardEntity {

  public String getCaption() {
      String caption = getFullName();
      if (type != null) {
          String localizedType = AppBeans.get(Messages.class).getMessage(getType());
          caption += " (" + localizedType + ")";
      }
      return caption;
  }

}
{% endhighlight %}

The resounding slap is not far away:

<figure class="center">
	<a href="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-execution-app-beans.png"><img src="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-execution-app-beans.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/testing-of-cuba-applications-part-2/customer-spec-execution-app-beans.png" title="AppBeans.get() will break some existing tests with a NullPointerException">AppBeans.get() will break some existing tests with a NullPointerException</a></figcaption>
</figure>

When we look at Line 61 of AppBeans where the NullPointerException occurs, we see the problem:
<code>AppContext.getApplicationContext().getBean(name, beanType)</code>

#### The missing spring application context in unit tests

The reason is we have no application context. In a unit test environment the Spring application context is not created. Therefore we can't use the AppBeans in the unit test. We could go on, read the docs about [middleware integration testing](https://doc.cuba-platform.com/manual-6.3/integration_tests_mw.html) and use the approach to start up the Spring application context for our unit test, but let's step back a little bit for now.

What we actually encountered is a dependency to an external class. The <code>Messages</code> interface is a external provider of information that it is out of scope of the Customer class. But in the last blog post we saw that a unit test tries to test units in isolation, without using the collaborators but instead cut the dependencies in order to control the environment for the test case.

So just spinning up the Spring application context would lead us to an integration test. That might be valuable for other reasons, but we would cover these for now. Instead, let's to try to treat the unit test as it is and deal with the "limitations".

But then the question is how can we make the test run? This is where we get into the topic we mentioned in the last blog post: Stubbing and Mocking.

### Mocking of external dependencies

In order to create a controllable dependency for the Messages interface we have to exchange the real implementation with something that we can control.

To do that, generally we have to express the dependency as something that can be placed from outside of the object into it. With this the Customer class in our case is no longer responsible for creating / pulling the reference from somewhere, but instead relies on the fact that for itself the reference is already there, regardless of who has placed the reference to it.

This is the very nature of Dependency Injection. The dependencies get injected into the objects in order to not let the objects deal with the problem.

#### Dependency Injection to the rescue?

In the [original groovy blog post](https://www.road-to-cuba-and-beyond.com/groovify-cuba-app-integrate-with-cuba/) I created the following example about mocking:

{% highlight groovy %}

class Customer {

    String firstName
    String lastName
    List<Order> orders = []

    @Inject
    OrderPlacementService orderPlacementService

    void placeOrder(Order order) {
        orders << order
        orderPlacementService.placeOrder(order)
    }

}

class Order {
    int amount
}

// ...

def 'When an order is placed, the order placement system will get notified'() {

    given: 'a mocked order placement system'
    def orderPlacementServiceMock = Mock(OrderPlacementService)

    and: 'a customer'
    def mario = new Customer(
        firstName: "Mario",
        lastName: "David",
        orderPlacementService: orderPlacementServiceMock
    )

    and: 'an order'
    def order = new Order(amount: 100)

    when: 'the order is placed'
    mario.placeOrder(order)

    then: 'the order placement system will get notified'
    1 * orderPlacementServiceMock.placeOrder(order)
}   
{% endhighlight %}

In the constructor of the customer, the orderPlacementServiceMock got injected which is a <code>Mock(OrderPlacementService)</code>.

<div class="information">Note that the customer class in this example is not the same as we use in the earlier examples. As I mentioned above, DI in CUBA Entities doesn't work. In this example it can be seen as any POJO that would have DI enabeld in the running application.</div>

Using <code>Mock(MyDependency)</code> creates a Proxy through the Spock framework in order to capture interactions with this object. More information about Mocks can be found in the [interaction-based testing docs](http://spockframework.org/spock/docs/1.0/interaction_based_testing.html) of spock. The example above expressed our expectation, that the customer class sends a particular message to the orderPlacementServiceMock exactly once like this:
<code>1 * orderPlacementServiceMock.placeOrder(order)</code>. If this expectation is fulfilled, the test will pass, otherwise it will fail.

<div class="information">When you want more information about what I just said about messages that get's send, take a look this <a href="http://softwareengineering.stackexchange.com/questions/140602/what-is-message-passing-in-oo">Question on StackExchange</a>. A really good book about this topic is <a href="https://www.amazon.com/Practical-Object-Oriented-Design-Ruby-Addison-Wesley/dp/0321721330/ref=sr_1_1?s=books-intl-de&ie=UTF8&qid=1483526200&sr=1-1&keywords=practical+object-oriented+design+in+ruby">Practical Object-Oriented Design in Ruby</a> by Sandi Metz. It goes in much more detail on this and related topics.</div>

So although this information might help us in the case where dependency injection is available, how does it help us with our Entity? Not directly. But as we need Mocking in the next blog posts, it's good to know about it.

#### Encapsulate dependencies and subclass

Right know, we have to add another trick that I learned from the book <a href="https://www.amazon.com/Working-Effectively-Legacy-Robert-Martin/dp/0131177052/ref=sr_1_1?ie=UTF8&qid=1483527057&sr=8-1&keywords=legacy+code">Working effectively with legacy code</a>. It's called <a href="https://blog.pivotal.io/labs/labs/test-after-in-java-subclass-and-override">Subclass and override method</a>.

To do that we have to encapsulate the dependency in a method that we will override in the test later.


{% highlight java %}
public class Customer extends StandardEntity {
    // ...
    public String getCaption() {
        String caption = getFullName();
        if (type != null) {
            String localizedType = getMessages().getMessage(getType());
            caption += " (" + localizedType + ")";
        }
        return caption;
    }

    protected Messages getMessages() {
        return AppBeans.get(Messages.class);
    }

}
{% endhighlight %}

With this change we are able to create subclass that we will override the getMessages method. I called it <code>CustomerWithInjectableDependencies</code>:

{% highlight java %}
public class CustomerWithInjectableDependencies extends Customer {

    private Messages messages;

    @Override
    protected Messages getMessages() {
        return messages;
    }
}
{% endhighlight %}

This class can now be used in the test so that we are able to inject a dependency into the Customer class manually.

{% highlight groovy %}

class CustomerSpec extends Specification {

    Customer customer
    Messages messagesStub

    void setup() {

        messagesStub = Stub(Messages)
        customer = new CustomerWithInjectableDependencies(messages: messagesStub)
    }

    // ...

    def "getCaption prints the type if set after the name in parentheses"() {

        given:
        customer.firstName = null
        customer.name = "Jones"
        customer.type = CustomerType.LOYAL

        and: "the messages stub is called for getMessage with the LOYAL customerType"
        messagesStub.getMessage(CustomerType.LOYAL) >> "local"

        expect:
        customer.getCaption().endsWith "(local)"
    }

    // ...

}
{% endhighlight %}

First we create a Stub for Messages and use it to pass it into the messages attribute of the CustomerWithInjectableDependencies constructor. Next, we have to define in the test, how the Stub should behave. We want that if it gets a method call on <code>getMessage</code> with the parameter CustomerType.LOYAL, then it should respond with the String "local" (which is expressed through the ">>" arrows).

That's it. With this we injected the messages object into the Customer class although it might otherwise would be able to do so.

You may asked, why I used a Stub instead of a Mock. Well, Mocks are usually used to test interactions between the object (like in the example with the OrderPlacementService). When you just want to stub out certain functionality, you'll probably use Stubs. More information about the distinction can be found at Martin Fowlers article about [Stubs and Mocks](http://martinfowler.com/articles/mocksArentStubs.html).


With this I would like to close this topic for now. As you have seen, it is quite easy to test entities in CUBA (and probably in JPA in general). Only the last part was a little bit more advanced where we used mocking in order to Stub out certain functionality of the Customer class, in order to test the actual method.

In the next article we will focus on the next artifacts.

I hope you enjoyed this testing article. If so, you can write me a mail or post a comment. If not, you can and should do it as well :)
