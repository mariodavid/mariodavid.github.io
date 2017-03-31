---
layout: post-dark
title: CUBA Security Subsystem Distilled - Part 2
description: "In this blog post, we will discover the insights of CUBA's Security subsystem and how different business security requirements can be achieved."
modified: 2017-04-15
tags: [cuba, security]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---



<!-- more -->


<img style="float: left; margin-left:-280px; margin-top:-450px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-7.jpg">


### From Albuquerque to Havanna


Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

<img style="float: right; margin-right:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-6.jpg">



## Constraint examples

### All users can only edit "not closed" orders
1. Solution:

Constraint:
  entityName: cesc$order"
  operationType: "Update"
  CheckType: Check in memory
  Groovy Script: ""{E}.status != com.company.cesc.entity.OrderStatus.CLOSED"

Order-browse.xml:
              <action id="edit" constraintOperationType="update"/>

This works. The already closed orders a no longer capable of beeing edited. Problem: An order, that is currently not closed can't be changed to "orderStatus: closed", because when trying to change the instance to closed, the constraint gets applied, because the constraint only checks what is currently in memory. It would need some kind of that: old({E}.status != OrderStatus.CLOSED)

2. Solution:
- create another attribute: "closed: boolean" to the order class.
- create a OrderEntityListener that looks like this:

{% highlight groovy %}
@Component("cesc_OrderEntityListener")
public class OrderEntityListener implements BeforeInsertEntityListener<Order>, BeforeUpdateEntityListener<Order> {


    @Override
    public void onBeforeInsert(Order entity, EntityManager entityManager) {

        if (entity.getStatus() == OrderStatus.CLOSED) {
            entity.setClosed(true);
        }
    }


    @Override
    public void onBeforeUpdate(Order entity, EntityManager entityManager) {

        if (entity.getStatus() == OrderStatus.CLOSED) {
            entity.setClosed(true);
        }
    }


}
{% endhighlight %}

- create an Constraint:
  entityName: cesc$order"
  operationType: "Update"
  CheckType: Check in memory
  Groovy Script: ""!{E}.closed"

Here, we switched the check to the closed boolean flag. There the above problem does not occur.

### "walther" can only edit the orders created by himself in the Northeast area, not the ones created by "saul" in Northeast

Constraint:
  Access Group: Northeast
  entityName: cesc$order"
  operationType: "Update"
  CheckType: Check in memory
  Groovy Script: "{E}.createdBy == userSession.user.login"

### "walther" can only create orders for new customers if payment method is "CREDIT_CARD" or "PAYPAL"  
--> wizard generat:
(({E}.customer.type == value(com.company.cesc.entity.CustomerType.class, 'NEW') && {E}.paymentMethod in CREDIT_CARD,PAYPAL) || {E}.customer.type != value(com.company.cesc.entity.CustomerType.class, 'NEW'))

--> richtig wäre:
(
    {E}.customer.type == com.company.cesc.entity.CustomerType.NEW
    &&
    (
        {E}.paymentMethod com.company.cesc.entity.OrderStatus.CREDIT_CARD ||
        {E}.paymentMethod com.company.cesc.entity.OrderStatus.PAYPAL
    )
) 
||
{E}.customer.type != com.company.cesc.entity.CustomerType.NEW


### "saul" can create orders for new customers independent of the payment method

### All users can only "deliver" orders (via a button in the editor of the order), if there is at least one lineItem

### All users can only "send invoice" (via a button in the order browser), if the orders payment method is "invoice" and the orderState is "payed"
