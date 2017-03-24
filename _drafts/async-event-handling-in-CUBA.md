---
layout: post
title: Asynchronous event handling in CUBA
description:
modified: 2017-03-15
tags: [cuba, events, spring, notifications]
image:
  feature:
---

Event handling in an asynchronous fashion oftentimes leads to an architecture that is more flexible and decoupled.
In this article we will have a look at how to implement a CUBA application mostly communicates through messages.

<!-- more -->

## The life before messages
To start of with this whole topic, we can have a look at a sample application. We will use (once again) an ordermanagement application. A customer can have orders that might have order lines etc.

Let's look at the following requirement:

> "When a customer is created, a user account should get generated for this customer as well so that the person can have a look at their own orders".

Another requirement might be

> "When a customer is created, detailed information should be logged into a particular file that get's displayed on a monitor on the wall of the office"

The list might go on, but we will leave it for now. The traditional approach in most applications (and in a CUBA app as well) is to have an implementation of the following kind:

{% highlight groovy %}
public class CustomerServiceBean implements CustomerService {

    @Inject DataManager dataManager;
    @Inject UsermanagementService usermanagementService;
    @Inject OfficeWallService officeWallService;

    @Override
    public void createCustomer(Customer customer) {

        // 1. persist customer in DB
        dataManager.commit(customer);

        // 2. create user account for customer
        usermanagementService.createUserForCustomer(customer);

        // 3. log customer information into the office wall file
        officeWallService.logCustomerinformation(customer);

       // ...
    }
}
{% endhighlight %}

The CustomerService gets called from the UI and will do all necessary things that have to be done when a customer is created.

So the problem with this is the coupling between the different parts of the system. When thinking about you might notice that for the CustomerService it does not really need to know about the effects that it's action: "to create a customer" has.

If you don't care about it, the CustomerServiceBean is going to be the God object of your system in the long run. It will violate the single responsibility priciple, because instead of changing the class because of the one reason "the storate of a customer" you will end up in a situation where changing the class because of one of any reasons that are related to the customer storage is the actual situation.

## Event based architectures to the rescue

Because of this, we want to have a look at one option for us to improve the design with an event based architecture. A lot has been written about this stuff and it is already been around for a very long time.

There are a lot of examples of event based architectures in totally different granularities and time frames. Starting with something like UI events in visual basic, over to event driven architectures in a SOA context or with something more modern: reactive programming.

One major difference between the example above and the event based approach is the removal of the dependencies between the <code>CustomerService</code> as the caller to the <code>OfficeWallService</code> as the callee goes away. Instead both services depend on the "contract": <code>CustomerCreatedEvent</code>.

<figure class="center">
	<a href="{{ site.url }}/images/async-event-handling-in-cuba/sync-to-async.png"><img src="{{ site.url }}/images/async-event-handling-in-cuba/sync-to-async.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/async-event-handling-in-cuba/sync-to-async.png" title="Direct dependency (synchnous way - left) --> no dependency between services (asynchronous way - right)">Direct dependency (synchnous way - left) --> no dependency between services (asynchronous way - right)</a></figcaption>
</figure>

With this approach we are more closely to fulfilling the single responsiblity principle. Addinally, testing get way easier, because the need to mock the different collaborators will go down dramatically. We will close of the theoretical (nevertheless important) part and have a look at how we can do this sort of things in CUBA.

## Application events in CUBA

CUBA has already a lot of events that are created from the framework. On the persistence layer there is the feature of [Entity Listeners](https://doc.cuba-platform.com/manual-6.4/entity_listeners.html), where you can create classes that react to certain actions during the persistence process. Then there are a lot of UI related events that can be catched and act on your UI needs. But these are all framework --> application events.

What we want take a look at instead is the possibility to create custom application events that can be send and received within your CUBA application. This is more like application --> application.

### Spring application events

Luckily CUBA as a meta-framework sits on top of very major and feature rich frameworks that themselfs have a lot of stuff to offer. One of these frameworks is Spring. The Spring framework has an event-mechanism baked in from the very beginning (which is a fairly long time actually). The main purposes for an application developer is to get notified when certain technical things in the running application occur (like <code>ContextStartedEvent</code>).

Nevertheless custom application events are supported as well. In fact, in recent versions Spring improved the creation and handling of these events even futher (see the [docs](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#context-functionality-events-annotation) for more information).

So basically it boils down to the three major parts of the actual event object, the event sender and the event receiver. Let's take a closer look at those.

#### 1. Event object

You have to create a PO(J\|G)O class that represents an event like this:

{% highlight groovy %}
// Application event when a order is created
class OrderCreatedEvent {
    String source
    Order order
}
{% endhighlight %}


#### 2. Event publisher

Having that, we need a publisher of an event. Here's an example of a service that does that:

{% highlight groovy %}
import org.springframework.context.ApplicationEventPublisher

class ApplicationEventProducerService {

    @Inject
    ApplicationEventPublisher publisher;

    void produceApplicationEvent(Object event) {
        publisher.publishEvent(event)
    }

    void produceOrderCreatedEvent(Order order, String source) {
        publisher.publishEvent(new OrderCreatedEvent(order: order, source: source))
    }
}
{% endhighlight %}

The <code>ApplicationEventProducerService</code> is responsible for pushing an event into the atmosphere so that other parts of the system can see it fly and catch it if they want. The service (which acts as an example for your business logic that creates events) uses <code>org.springframework.context.ApplicationEventPublisher</code> to fulfil its purpose. This bean is the point in the Spring framework which lets us publish application events. As you might have notices, the method <code>publishEvent</code> can take any object to publish, it does not require a certain class type to do that (anymore...).

#### 3. Event receiver

The last part is the receiving part - which is called EventListener. An event listener qualifies itself by having a particular method annotation that should handle a certain event. Here's an example:

{% highlight groovy %}
@Component
class CustomerUserCreator {

    private final Logger log = LoggerFactory.getLogger(getClass());

    @Async
    @EventListener
    void handleCustomerCreated(CustomerCreatedEvent event) {
        User user = createUserFromCustomer(event.customer)
        log.warn("The user ${user.login} was created...".toString())
    }
{% endhighlight %}

<code>CustomerUserCreator</code> creates a user account in our application when a Customer has been created. The <code>handleCustomerCreated(CustomerCreatedEvent event)</code> method is the actual handler of the event due to the <code>@EventListener</code> annotation.

The parameter in the method definition qualifies which event is handled in this method. The qualification is normally done on a class level (through the type of the parameter).

If you want to define even further for what events of a particular times, this method should get called you can either do a big if statement within the method, or you can set the <code>condition</code> attribute like this:<code>@EventListener(condition = "someCondition")</code> (while <code>someCondition</code> has to be a valid [SpEL expression](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)).

One important point is, that in order to have multiple actions in your application that have to be executed when a customer is created, you just have to create another event listener bean. So going back to our example from above, we would have another bean called <code>OfficeWallService</code> that would look like this:

{% highlight groovy %}
@Component
class OfficeWallService {

    @Inject
    OfficeWall officeWall

    @Async
    @EventListener
    void handleCustomerCreated(CustomerCreatedEvent event) {
        officeWall.renderText("${event.customer.name} is our new and hopefully the best customer ever...")
    }
{% endhighlight %}

With this, everything is in place to create a event based application. There are many more options to consider, but as I will not mirror the Spring docs, I will leave it at this point. You can read more about this in the Spring [docs](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#context-functionality-events). Another very good article is the following: [Better application events in Spring Framework 4.2](https://spring.io/blog/2015/02/11/better-application-events-in-spring-framework-4-2).

#### Sync or Async - depending on your needs
As you might have seen, I used the <code>@Async</code> annotation in the handlers. The reason for that is, that I want to execute the stuff that happens when is user is created independently of the thread where it is created. There are some situations where you want a synchronous execution, but in this example I want the execution in another thread.

To make that happen, one thing is to add the above mentioned annotation. Another thing is, that there needs to be a configuration in the spring.xml that defines futher information about the asynchronous execution.

{% highlight xml %}
<beans>
    <task:annotation-driven executor="scheduler"/>
    <bean id="exceptionHandler" class="com.company.ceae.listener.application.ExecutorExceptionHandler"/>
</beans>
{% endhighlight %}

The "scheduler" reference is an existing CUBA bean ([CubaThreadPoolTaskScheduler](https://github.com/cuba-platform/cuba/blob/master/modules/global/src/com/haulmont/cuba/core/sys/CubaThreadPoolTaskScheduler.java)) which is an implementation of Springs ThreadPoolTaskScheduler.

The <code>task:annotation-driven</code> tag defines that Spring will scan the classpath for beans for the corresponding annotations.

### CUBA application event example
Let's have a look at how we can create a running example of this approach in CUBA. The complete example can be found on github: [mariodavid/cuba-example-application-events](https://github.com/mariodavid/cuba-example-application-events).

The example does the following things:

<figure class="center">
	<a href="{{ site.url }}/images/async-event-handling-in-cuba/cuba-example-activities.png"><img src="{{ site.url }}/images/async-event-handling-in-cuba/cuba-example-activities.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/async-event-handling-in-cuba/cuba-example-activities.png" title="Activity graph in the CUBA example">Activity graph in the CUBA example</a></figcaption>
</figure>

First, the user creates a customer through the standard editor for the customer entity. After that, the registered [CustomerCreatedEntityListener](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/core/src/com/company/ceae/listener/persistence/CustomerCreatedEntityListener.java) which is a normal CUBA entity listener,  creates a CustomerCreatedEvent through the above mentioned [ApplicationEventProducerService](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/core/src/com/company/ceae/service/ApplicationEventProducerServiceBean.groovy). The events gets fired aynchronously, so the DB transaction of storing the customer is finished independently of everthing else that might happen with te CustomerCreatedEvent.

The event is catched by multiple event handlers. The [CustomerUserCreator](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/core/src/com/company/ceae/listener/application/CustomerUserCreator.groovy) which creates a corresponding user for the customer so that the customer is capable of login to the system and see it's own orders. The next event handler is the [CustomerCreatedUserNotificator](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/core/src/com/company/ceae/listener/application/CustomerCreatedUserNotificator.groovy), which creates a notification for every user that subscribed on these events. This leads to the UI that updates itself for the corresponding user and shows a new notification message. As the last part has some more business logic, let's have a look at it more closely.

#### Users can subscribe to application events

One example of what can be done as a useful event handler is to notify users through the UI that certain things happend in the application.

<p class="information">This example of notifiing users does not have a direct correlation between itself and the fact that we use application events. Although the play nicely together, these things are orthogonal. We could implement a notification system without using an event based architecture as well as using an event based architecture without showing these notifications on the UI.</p>


In order to allow the user to subscribe to certain events, we have to create an entity that stores these subscriptions. The user can define the class of the nofitication and a condition that has to be true in order to get a notification.

<figure class="center">
  <img style="padding: 10px" src="{{site.url}}/images/async-event-handling-in-cuba/subscription-editor.png">
</figure>

The [CustomerCreatedUserNotificator](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/core/src/com/company/ceae/listener/application/CustomerCreatedUserNotificator.groovy) will pick up all subscriptions of every user where the event fulfils the entity class criteria as well as the condition. For each of those subscriptions it will create a new [Notification](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/global/src/com/company/ceae/entity/Notification.java) entity for the corresponding user. It will additionally link the entity where the notification comes from in the entity, so that the user can hit a link to go directly to this entity instance.

After the notifications have been persisted in the database, we can have a look at how to get the information in the users UI.

#### Update the UI through polling

The [MainWindow](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/web/src/com/company/ceae/web/screens/ExtAppMainWindow.groovy) of the app uses the recently added [SideMenu](https://doc.cuba-platform.com/manual-6.4/gui_SideMenu.html) component. I decided to do a polling of the database through the trigger option we have in CUBA for UI screens. The timer calls [updateCounters](https://github.com/mariodavid/cuba-example-application-events/blob/master/modules/web/src/com/company/ceae/web/screens/ExtAppMainWindow.groovy#L58) which will ask the database for notifications for the current user, that are unread and update the badge text of the menu item accordingly.

The resulting user interface looks like this:

<figure class="center">
	<a href="{{ site.url }}/images/async-event-handling-in-cuba/user-notifications.png"><img src="{{ site.url }}/images/async-event-handling-in-cuba/user-notifications.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/async-event-handling-in-cuba/user-notifications.png" title="User notification screen with unread notifications">User notification screen with unread notifications</a></figcaption>
</figure>


This might be a reasonable solution for the problem. We could have done it differently, e.g. when there are a lot of users and the polling will result in a large amount of load for the database. In this case, it might be worth to find the corresponding sessions and set a session attribute with the counter. But I think that this shows the basic functionality. Everything else is up to the specific application.

I hope you got a basic idea of what application events can bring on the table and if they are suitable for your needs. If you have any questions or comments - please let me know.
