---
layout: post
title: Asynchronous event handling in CUBA
description:
modified: 2017-03-15
tags: [cuba, messaging, spring]
image:
  feature:
---

Event handling in an asynchronous fashion oftentimes leads to an architecture that is more flexible and decoupled.
In this article we will have a look at how to implement a CUBA application mostly communicates through messages.

<!-- more -->

## The life before messages
To start of with this whole topic, we can have a look at a sample application. We will use (once again) an ordermanagement application. A customer can have orders taht might have order lines etc.

Let's look at the following requirement:

> "When a customer is created, a user account should get generated for this customer as well so that the person can have a look at their own orders".

Another requirement might be

> "When a customer is created, detailed information should be logged into a particular file that get's displayed on a monitor on the wall of the office"

The list might go on, but we will leave it for now. The traditional approach in most applications (and in a CUBA app as well) is to have some kind of implementation:

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

With this, we will close of the theoretical (nevertheless important) part and have a look at how we can do this sort of things in CUBA.

## Application events in CUBA

CUBA has already a lot of events that are created from the framework. On the persistence layer there is the feature of [Entity Listeners](https://doc.cuba-platform.com/manual-6.4/entity_listeners.html), where you can create classes that react to certain actions during the persistence process. Then there are a lot of UI related events that can be catched and act on your UI needs. What i mainly want to show to you is the possibility to create custom application events that can be send and received within your CUBA application. 

### Spring application events

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
