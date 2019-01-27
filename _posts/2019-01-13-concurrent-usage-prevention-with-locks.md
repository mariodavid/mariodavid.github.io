---
layout: futurama
title: Concurrent Usage prevention with Locks
description: "In this blog post we will go over an example that deals with preventing concurrent usage or particular resources like entities through the different CUBA mechanisms of locking."
modified: 2019-01-13
tags: [cuba, locking]
image:
  dir: concurrent-usage-prevention-with-locks
  feature: concurrent-usage-prevention-with-locks/feature.png
---

In this blog post we will go over an example that deals with preventing concurrent usage of particular resources like entities through the different CUBA mechanisms of locking.

<!-- more -->

The complete example can be found at Github:

{% include github-example.html repository="cuba-example-concurrent-usage-prevention" %}

Sometimes certain resources of an application like a particular entity or even more general a particular part of the application should only be accessed by one particular user at a time in order to prevent the following situation:

## Scenario: Lost update of Customer data for Zapp Brannigan


{% include image-center.html image="zapp-love.png" class="futurama-style" width="500px" %}

1. <code class="clock">10:30</code>: Customer <code>Zapp Brannigan</code> sends an email in order to inform about his marriage with <code>Kif Kroker</code>. Therefore he wants to update his last name to <code>Kroker-Brannigan</code>
2. <code class="clock">10:35</code>: User <code class="leia">leia</code> reads the email and opens the Customer screen from <code>Zapp Brannigan</code>. Directly after that her boss - <code>Jabba the Hutt</code> calls her in for an urgent task. She leaves the customer details screen open.
3. <code class="clock">10:40</code>: <code>Zapp</code> calls the hotline, because the marriage did not last long and is already gone. But as he found this lovely girl <code>Amy Wong</code> - he now wants to update his name to <code>Wong</code> because they are in love. User <code class="luke">luke</code> accepts the request as he does not know anything about the previous email, opens the Customer screen and changes the name to <code>Zapp Wong</code>.
4. <code class="clock">10:50</code>: User <code class="leia">leia</code> is back on her desk and wants to finish her task of updating <code>Zapp Brannigan</code> to <code>Bareny Kroker-Brannigan</code>. As she has the customer screen already open, she just changes the name and saves it.

Let's look at the data that has been stored in the system:

1. <code class="clock">10:30</code> - Customer <code>Zapp Brannigan</code>
2. <code class="clock">10:40</code> - Customer <code>Zapp Wong</code>
3. <code class="clock">10:50</code> - Customer <code>Zapp Kroker-Brannigan</code>

As we have seen from the interactions with <code>Zapp</code>, what is finally stored in the system is wrong and does not reflect the real world as the final name should be <code>Zapp Wong</code> instead of <code>Zapp Kroker-Brannigan</code>.

In order to solve this scenario, there are some established techniques. CUBA offers the following options:

* optimistic locking
* pessimistic locking
* entity log

In this example we will go through the different locking options.

## Example Overview

Here is an overview of the functionality of the different examples for the different locking solutions within a CUBA application:

{% include hover-image.html image="overview.gif" class="overview" description="Example overview" %}


## Optimistic locking

Optimistic locking is enabled by default for all entities in CUBA and is the standard solution to this problem. The way it works looks like this:

In this example it is implemented for the entity: <code>Product</code>. <code>Product</code> contains an attribute called <code>version</code> which is a counter. It is incremented each time a particular entity is created / updated.

When a user opens the product details screen in order to update a particular product, the current value of the <code>version</code> attribute is also received in the screen. In case of the above example with the Customer entity, an optimistic locking would look like this:

1. before <code class="clock">10:30</code>: Customer <code>Zapp Brannigan</code> - version <code>1</code>
2. at <code class="clock">10:35</code>: <code class="leia">leia</code> reads the customer with version <code>1</code>
3. at <code class="clock">10:40</code>: <code class="luke">luke</code> reads the customer with version <code>1</code> and stores the customer. The version attribute is updated to value <code>2</code>.
4. at <code class="clock">10:50</code>: <code class="leia">leia</code> wants to write the customer with the one she read at <code class="clock">10:35</code> with version <code>1</code>. On storing the new data for the Customer CUBA will throw a <code>OptimisticLockException</code>, because the version in the DB is already <code>2</code> (from <code class="clock">10:35</code>) and now <code class="leia">leia</code> wants to store information with a current version of <code>1</code>. This prevents <code class="leia">leia</code> from storing the values based on old data.

The resulting UI for an optimistic locking exception looks like this:

{% include hover-image.html image="automatic-optimistic-entity-lock.png" description="Automatic optimistic entity lock" %}

The user has to manually reload the customer and based on the new data decide which action to take and ultimately which data to persist.

The reason why it is called <code>optimistic</code> is because this strategy is optimistic in the sense that concurrent reads are treated as a normal interaction. Only at the last possible point it throws an exception to prevent potential wrong data storage.



The difference to the pessimistic locking is that optimistic locking allows to open different reads of the resource and also leads the user to believe
that concurrent updates would be possible by the fact that CUBA shows the normal screen editor for the entity without any restrictions. 


{% include image-left.html image="bender-hit-face.gif" class="futurama-style" width="150px" %}

Only if the second user actually does a concurrent change, the system will prevent this. For the user though, it oftentimes feels like Bender in this gif. 

The problem is that once the system returns this error, the user is not expecting such behavior and now is in the situation to figure out which person has already done another change without having more detailed information about it.

This is why optimistic locking sometimes can be an undesired behavior.

More information on optimistic locking can be found here:

* [optimistic concurrency control (wikipedia)](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)
* [JPA optimistic locking (baeldung.com)](https://www.baeldung.com/jpa-optimistic-locking)

## Pessimistic locking

{% include image-left.html width="200px" image="zapp-brannigan.png" class="futurama-style" %}

Sometimes the optimistic locking approach is not appropriate because the system should not lead the user to believe that such operations are possible. In this case the pessimistic locking approach is better suited.

The user is informed upfront that another user is currently doing the wished operation (like changing the customer). This enforces the user to deal with a potential problem upfront.

In the above <code>Zapp Brannigan</code> scenario, <code class="luke">luke</code> would have been informed, when opening the Customer editor screen, with the information that <code class="leia">leia</code> is already trying to change the customer, and would prevent <code class="luke">luke</code> from changing the customer all together.


This approach is a little more "secure" in the sense that it notifies the user upfront. But also leads to a lot of "false negatives". 


{% include image-right.html width="200px" image="fry.webp" class="futurama-style" %}

This oftentimes leads to the situation where you have a lot of frustrated users that actually just wanted to look at the data and then decide if something needs to be changed. But for all cases, the system informs them about the read-only mode. So it is wise to use pessimistic locking where necessary in order to not let your users feel like Fry here.


### Automatic pessimistic locking for entity editors

CUBA offers automatic pessimistic locking for entity editors. It sets the editor in a read-only mode so that no changes to the entity are possible once one user opened the editor.

In order to activate this behavior, a runtime configuration needs to be in place on a per-entity basis. CUBA offers a management UI under <code>Administration > Locks > Setup</code>:


{% include hover-image.html image="current-locks.png" description="Current locks" %}


The resulting UI of the automatic pessimistic locking approach for the customer editor looks like this:

{% include hover-image.html image="automatic-pessimistic-entity-lock.png" description="Automatic pessimistic entity lock screen" %}


### Overview of existing locks




{% include image-right.html image="happy-wait-dr-zoidberg.gif" class="futurama-style" %}

Normally locks are released once the operation is done. For the automatic pessimistic locking of entity editors in CUBA happens when the user closes the editor screen. Unfortunately releasing the lock is not always possible, because the user does not close the editor and instead  e.g. closes the laptop completely. This leads to open locks, that are not released.

Therefore the lock configuration has to have a timeout configured. After that the system will automatically resolve the locks in order to prevent locking of particular data forever.

CUBA additionally allows the administrator of the system to look at the current locks and also release them manually in case such scenario occurs. This is useful when the timeout of the lock is not yet over, but the user wants to get access to the data.

{% include hover-image.html image="current-locks.png" description="Currently active locks" %}


### Custom pessimistic locking

It is also possible to use custom pessimistic locks programmatically. This is sometimes necessary when a lock should not be based on an entity instance. It also requires to configure a lock via the CUBA management UI. In this example it is configured for the name <code>customer-support-ticket</code>.


{% include hover-image.html image="current-lock-configurations.png" description="Lock configuration" %}

For programmatically accessing the pessimistic locking functionality of CUBA, there are two interaction points: <code>LockManagerAPI</code> for the backend as well as <code>LockService</code> for the web layer (e.g. UI controllers). In the <a href="https://github.com/mariodavid/cuba-example-concurrent-usage-prevention/blob/master/modules/web/src/com/rtcab/cecup/web/customer/CustomerSupportTicket.java">CustomerSupportTicket</a> screen, the <code>LockService</code> is used in order to prevent creating support tickets for the same customer. It is done via the custom pessimistic lock with the name <code>customer-support-ticket</code>: <code>lockService.lock(CUSTOMER_SUPPORT_LOCK_NAME, customerId(customer))</code>.

{%highlight java%}
public class CustomerSupportTicket extends AbstractWindow {

  private final String CUSTOMER_SUPPORT_LOCK_NAME = "customer-support-ticket";

  @Inject
  protected LockService lockService;

  @WindowParam
  Customer customer;


  @Override
  public void ready() {

    LockInfo customerSupportTicketLock = tryToAcquireCustomerSupportTicketLock(customer);

    if (customerSupportTicketIsAlreadyLocked(customerSupportTicketLock)) {
      showCustomerSupportTicketLockedWarning(customerSupportTicketLock);
      close(CLOSE_ACTION_ID, true);
    }
  }

  private void showCustomerSupportTicketLockedWarning(LockInfo lockInfo) {
    String currentlyUserHavingLock = lockInfo.getUser()
        .getInstanceName();
    String message = "Customer Support Ticket is already in use by: " + currentlyUserHavingLock;
    showNotification(message, NotificationType.WARNING);
  }

  private boolean customerSupportTicketIsAlreadyLocked(LockInfo customerLock) {
    return customerLock != null;
  }

  private LockInfo tryToAcquireCustomerSupportTicketLock(Customer customer) {
    return lockService.lock(CUSTOMER_SUPPORT_LOCK_NAME, customerId(customer));
  }

  private void unlockCustomerSupportTicket(Customer customer) {
    lockService.unlock(CUSTOMER_SUPPORT_LOCK_NAME, customerId(customer));
  }

  @Override
  protected boolean preClose(String actionId) {
    unlockCustomerSupportTicket(customer);

    return true;
  }


  public void printInteraction() {
    showNotification("Ticket send to printer...");
  }

}
{% endhighlight %}



The resulting UI in case of simultaneous access looks like this:

{% include hover-image.html image="custom-lock.png" description="Custom lock error message" %}


## Summary

This example should show the different options that are available when it comes to preventing the concurrent use of resources. CUBA supports both optimistic and pessimistic locking.

Optimistic locking is the default behavior and will notify the user once an actual attempt to write the concurrent changes by multiple users occurs.

For pessimistic locking, a configuration has to be set. After that the default CUBA editor screen will go into a read-only mode in case multiple users are opening the same entity editor concurrently.

CUBA also offers API to programmatically use pessimistic locking so that it can be used in non-entity scenarios.


Which one of these two solutions to use highly depends on the use case, how the data is structured, how the UI looks like etc. With that article, you hopefully have enough information about the pros and cons of both approaches.

{% include image-center.html width="400px" image="summary.webp" class="futurama-style" %}

