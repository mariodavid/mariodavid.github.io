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
