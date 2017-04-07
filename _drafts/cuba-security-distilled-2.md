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

### 1. All users can only edit "not closed" orders

#### 1.1. Solution:

Create a constraint:
{% highlight json %}
{
  "entityName": "cesc$Order",
  "operationType": "Update",
  "checkType": "Check in memory",
  "groovyScript": "{E}.status != com.company.cesc.entity.OrderStatus.CLOSED"
}
{% endhighlight %}

Order-browse.xml:
{% highlight xml %}
  <action id="edit" constraintOperationType="update"/>
{% endhighlight %}

This works. The already closed orders a no longer capable of beeing edited.

Problem: An order, that is currently not closed can't be changed to "orderStatus: closed", because when trying to change the instance to closed, the constraint gets applied, because the constraint only checks what is currently in memory. It would need some kind of that: <code>old({E}).status != OrderStatus.CLOSED</code> (see 3. Solution for this)

#### 1.2. Solution:
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

Constraint:
{% highlight json %}
{
  "entityName": "cesc$Order",
  "operationType": "Update",
  "checkType": "Check in memory",
  "groovyScript": "!{E}.closed"
}
{% endhighlight %}

Here, we switched the check to the closed boolean flag. There the above problem does not occur.



#### 1.3. Solution:
- reload the entity from db, to get the current persistent entity

Constraint:
{% highlight json %}
{
  "entityName": "cesc$Order",
  "operationType": "Update",
  "checkType": "Check in memory",
  "groovyScript": "see below"
}
{% endhighlight %}


Groovy Script:
{% highlight groovy %}
def dataManager = com.haulmont.cuba.core.global.AppBeans.get(com.haulmont.cuba.core.global.DataManager)
def currentPersistedEntity = dataManager.reload({E},"_local")
def closedStatus = com.company.cesc.entity.OrderStatus.CLOSED

if ({E}.status != closedStatus) {
    return true
}
else if (currentPersistedEntity.status != closedStatus && {E}.status == closedStatus) {
    return true
}
else if (currentPersistedEntity.status == closedStatus && {E}.status != closedStatus) {
    return true
}
else {
    return false
}
{% endhighlight %}

In this case, we reloaded the entity in order to be able to check more conditions: not only the current state of the entity is important, but also the values that were in the database before.

### 2. "walther" can only edit the orders created by himself in the Northeast area, not the ones created by "saul" in Northeast

Constraint:
{% highlight json %}
{
  "accessGroup": "Northeast",
  "entityName": "cesc$Order",
  "operationType": "Update",
  "checkType": "Check in memory",
  "groovyScript": "{E}.createdBy == userSession.user.login"
}
{% endhighlight %}

### 3. "walther" can only create orders for new customers if payment method is "CREDIT_CARD" or "PAYPAL"  

Constraint:
{% highlight json %}
{
  "accessGroup": "Northeast",
  "entityName": "cesc$Order",
  "operationType": "Create",
  "checkType": "Check in memory",
  "groovyScript": "see below"
}
{% endhighlight %}

Groovy Script:
{% highlight groovy %}
def newCustomer = com.company.cesc.entity.CustomerType.NEW
com.haulmont.cuba.core.global.PersistenceHelper.checkLoaded(__originalEntity__, 'paymentMethod')
com.haulmont.cuba.core.global.PersistenceHelper.checkLoaded(__originalEntity__.customer, 'type')
if ({E}.customer.type == newCustomer) {
    if ({E}.paymentMethod == com.company.cesc.entity.PaymentMethod.CREDIT_CARD || {E}.paymentMethod == com.company.cesc.entity.PaymentMethod.PAYPAL) {
        return true
    }
    else {
        return false
    }
}
else {
    return true
}
{% endhighlight %}

In this groovy script we check the above mentioned condition.

To make this work, the view of the order in the order editor has to match with the things that are used in the groovy script (e.g. {E}.customer.type).
To check that the attributes are actually in the view (through <code>PersistenceHelper.checkLoaded</code>) on the entity it is required to use the magic variable <code>__originalEntity__</code> that gets passed into the script via <code>SecurityConstraintExtensionImpl</code>.


### 4. "saul" can create orders for new customers independent of the payment method

As the security constraint only gets applied to the Northeast access group, saul will not be effected by this restriction. Therefore he is capable of creating orders independent of the customers customer type.

### All users can only "deliver" orders (via a button in the editor of the order), if there is at least one lineItem

### All users can only "send invoice" (via a button in the order browser), if the orders payment method is "invoice" and the orderState is "payed"
