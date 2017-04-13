---
layout: post-dark
title: CUBA Security Subsystem Distilled - Part 2
description: "In this blog post, we will discover the insights of CUBA's Security subsystem and how different business security requirements can be achieved."
modified: 2017-04-15
tags: [cuba, security]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

CUBA Security Inc. is still around and just recently wants to benefit from the CUBA even further. The ERP-software should support different requirements regarding security constrains. In order to fulfil reguilatory requirements the software needs to do audit logging. In this blog post we will try to implement the requirements and with this have a look at how the parts of the security subsystem work in CUBA.

<!-- more -->


<img style="float: left; margin-left:-280px; margin-top:-450px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-7.jpg">


### Picking up where we left off

In the [last blog post](https://www.road-to-cuba-and-beyond.com/cuba-security-subsystem-distilled/) about the whole security subsystem in CUBA i wrote that i might write another blog post to describe additional examples of CUBAs security relates features. Although it took quite some time, it finally arrived. This time, it is mostly about interacting with the security system in a (semi-)programmatic way. Additionally  we will take a slight look at the auditing options of the platform.

If you are not fimilar with the basic building blocks of the security features of CUBA, i would encourange you to read about it in the [first part](https://www.road-to-cuba-and-beyond.com/cuba-security-subsystem-distilled/).


<img style="float: right; margin-right:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-6.jpg">



## Extended business-related security constraints

Currently the security constraints that are applied in the software are mostly done through CUBAs Role system as well as the constraints that get applied the different access groups.

Besides the existing ones there are a number of additional requirements. We will go through the list one by one and try to implement them. Mostly this is possible not through coding but a semi-programmatic way where we use the feature of the access group constraints and combine it with custom groovy scripts. Let's take a look.

You can find all examples in the example application: [cuba-example-security-constraints](https://github.com/mariodavid/cuba-example-security-constraints) on github.

### Security at build time vs. runtime

To get a slow start into this topic, let's take a look at what solutions are possible in case we don't know anything about the platform security features. We will use the first requirement (that we will see below in more detail) as a basis for that:

*All users can only edit "not closed" orders*

One obvious solution would be in the order editor controller, to do a check in the <code>preCommit</code> action. Here is an example of such a solution:

{% highlight groovy %}
class OrderEdit extends AbstractEditor<Order> {

    // ...

    @Override
    protected boolean preCommit() {
        if (item.closed) {
            showNotification("No no, you cannot do that", Frame.NotificationType.ERROR)
            false
        }
        else {
            showNotification("That works out quite well", Frame.NotificationType.TRAY)
            true
        }
    }
}
{% endhighlight %}

This works. Here are a few of the benefits:

* code is at the place you expect it to be - a error message on a screen is defined in the corresponding screen controller
* embedded in the source code of the software - runtime configurations can't accidentily disable security constaints
* fairly easy to test in isolation

Unfortunately some major drawbacks as well:

* security logic in UI screens will not prevent other system parts to violate the rule (e.g. the REST API)
* no seperation of concerns - UI button logic will sit right next to security constraint logic
* embedded in the source code of the software - not changeable on a per installation basis


We will care about all those drawbacks in this blog post, but let's start with the first one. Although the UI might be a big player in terms of data entry, in most cases it is not the only one.

Sometimes you have a REST API (like the CUBA built-in REST API) where you can add data to the system. Or you do a batch import of some kind through another screen. Or think of the [BulkEditor](https://doc.cuba-platform.com/manual-6.4/gui_BulkEditor.html) in CUBA - all of those possibilities are not aware of your logic in the controller. Therefore the security constraints will not get applied.

For some of those you can copy over the logic (e.g. other screens) to replicate the behavior, but other you just can't (e.g. BulkEditor) because they are part of the platform and there are no extension points.

Basically this is a show stopper for most of the applications. So lets think about another solution that is a little bit more lower level: *Bean Validation*.

#### Bean Validation as a global way of enforcing security / business rules

In the 6.4 release CUBA added support for [Bean Validation](http://beanvalidation.org/). When you look at the website of this, it says: *"Constrain once, validate everywhere"* - that's what we want right? Fine.

So it basically is a Java standard that gives you the possibility to define validation rules on Java classes. CUBA executes those validation rules in the UI and in other parts of the system (e.g. the REST API). With this we have a much better position to do those kinds of checks globally.

So how can a solution to this look like? First we have to create a custom constraint and apply it in the order entity. Detailed information on that can be found in the [docs](https://doc.cuba-platform.com/manual-6.4/bean_validation_constraints.html).

In this example, we will create an annotation called <a href="https://github.com/mariodavid/cuba-example-security-constraints/blob/2976edfb041bd4a1330a099e7cb334b4f7ad9d87/modules/global/src/com/company/cesc/entity/validation/CheckOrderNotClosed.java"><code>CheckOrderNotClosed</code></a> with a corresponding validation class called <a href="https://github.com/mariodavid/cuba-example-security-constraints/blob/2976edfb041bd4a1330a099e7cb334b4f7ad9d87/modules/global/src/com/company/cesc/entity/validation/OrderNotClosedValidator.java"><code>OrderNotClosedValidator</code></a>.

The Order entity is annotated with the new Annotation like this:

{% highlight groovy %}
@CheckOrderNotClosed(groups = {Default.class, UiCrossFieldChecks.class})
@Entity(name = "cesc$Order")
public class Order extends StandardEntity {
  //...
}
{% endhighlight %}

The actual Validator with the security logic looks like this:

{% highlight groovy %}
public class OrderNotClosedValidator implements ConstraintValidator<CheckOrderNotClosed, Order> {

    @Override
    public boolean isValid(Order value, ConstraintValidatorContext context) {
        return !value.getClosed();
    }
}
{% endhighlight %}

### 1. All users can only edit "not closed" orders

The first requirements deals with the fact that although orders can be updated only when the order is not already in status "closed".

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

### 2. "walter" can only edit his own orders in Northeast

In this example, we want the user walter to constraint in such a way, that the set of orders he is allowed to edit will reduce futher. Currently he can see (and edit) all orders that are placed by customers in the Northeast area. Now we want that, although he can see all of those orders, editing is only possible for a subset of those. In this case, he should be allowed to edit only orders that created by himself.

Here's the corresponding constraint for that requirement:

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

In the groovy script, certain variables get injected. One of those is <code>userSession</code> which allows getting information about the current user. In this case we need the login string to compare it to the currents entites createdBy attribute.

### 3. "walter" can only create non-invoice based orders for new customers

The next example is a little bit more complex. As the company is not willing to trust walter in the same way it trusts other employees, it is required to to treat him in a special way. In this case, the following constraint should be applied:

When walter creates an order with a customer that has the customer type "NEW", it should be only allowed to create this order with particular payment types. New Customers are not allowed to place an order that will be payed via invoice.

Here is the corresponding constraint:

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
com.haulmont.cuba.core.global.PersistenceHelper.checkLoaded(__originalEntity__, 'paymentMethod')
com.haulmont.cuba.core.global.PersistenceHelper.checkLoaded(__originalEntity__.customer, 'type')

def newCustomer = com.company.cesc.entity.CustomerType.NEW
if ({E}.customer.type == newCustomer) {
    if ({E}.paymentMethod != com.company.cesc.entity.PaymentMethod.INVOICE) {
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

To make this work, the view of the order in the order editor has to match with the things that are used in the groovy script (e.g. <code>{E}.customer.type</code>). Here we see, that there is an implicit correlation between the editor screen definition and the constraint through the need for a particular view. This is somewhat problematic. Because of this, we need to program a little bit more defensive and check in our script.

To check that the attributes are actually in the view (through <code>PersistenceHelper.checkLoaded</code>) on the entity it is required to use the magic variable <code>__originalEntity__</code> that gets passed into the script via <code>SecurityConstraintExtensionImpl</code>.

We will check that the attributes are loaded via the view in order to not produce false positive results in the script. <code>PersistenceHelper.checkLoaded</code> will throw an exception if the attribute is not loaded, therefore the script will evaluate to false.


In case the attributes are loaded we will check the attributes for the above mentioned conditions.


### 4. "saul" can create orders for new customers independent of the payment method

This is a really easy one. As the security constraint only gets applied to the Northeast access group, saul will not be effected by this restriction. Therefore he is capable of creating orders independent of the customers customer type.

### 5. All users can only "deliver" orders (via a button in the editor of the order), if there is at least one lineItem

In order to achieve this requirement, we need to do a little bit of glue code in the editor of the order. First we will add an appropriate Button and action to the order-edit.xml screen definition like this:

{% highlight xml %}
    <!-- ... -->

    <actions>
        <action id="deliverOrderAction"
                caption="msg://deliverOrder"
                invoke="deliverOrder"/>
    </actions>
    <!-- ... -->

    <buttonsPanel id="buttonsPanel">
        <button id="deliverOrderBtn"
                action="deliverOrderAction"
                icon="font-icon:SEND"/>
    </buttonsPanel>

    <!-- ... -->
{% endhighlight %}

Lets have a look at the <code>deliverOrder</code> method. It will use the [Security platform interface](https://doc.cuba-platform.com/manual-6.4/security.html), that allows certain requests against the security subsystem of CUBA in a programmatic fashion.

In this case we use the the variant to check if an entity has a particular permission on a custom constraint code. There are other options like checking for screen permissions etc. On [github](https://github.com/cuba-platform/cuba/blob/master/modules/global/src/com/haulmont/cuba/core/global/Security.java) you'll find the full fletched interface of it, but for this case, we will use the custom constraint code like this:

{% highlight groovy %}


// ...
import com.haulmont.cuba.core.global.Security
// ...

class OrderEdit extends AbstractEditor<Order> {

    @Inject Security security

    void deliverOrder() {
        if (security.isPermitted(item, "deliverOrder")) {
            item.status = OrderStatus.DELIVERED
            showNotification("Order delivered...", Frame.NotificationType.TRAY)
            commitAndClose()
        }
        else {
            showNotification("Not allowed...", Frame.NotificationType.ERROR)
        }
    }
}
{% endhighlight %}

After that little bit of clue code, we need to define the actual constraint like before. But this time, we will pick the check type "Custom". This option is relevant for constraint operations that are not directly related to the CRUD operations. In this case we will call it "deliverOrder".

Constraint:
{% highlight json %}
{
  "accessGroup": "Company",
  "entityName": "cesc$Order",
  "operationType": "Custom",
  "operationConstraintCode": "deliverOrder",
  "checkType": "Check in memory",
  "groovyScript": "see below"
}
{% endhighlight %}

Groovy Script:
{% highlight groovy %}
if ({E}.lineItems.size() > 0) {
    return true
}
else {
    return false
}
{% endhighlight %}

The groovy script itself just checks if there are line items in place. In this case it will the operation is allowed. One could argue that this requirement is probably somewhat unrelated and probably this is true, but as this example should just show the dfferent techniques that can be used it doesn't really matter for now.

### 6. All users can only "send invoice" (via a button in the order browser), if the orders payment method is "invoice" and the orderState is "payed"

This last example is fairly similar to the one above. We need to create a little bit of code in our application to make this work. In this case we will go another route in the implementation. The screen definition basically creates button in the buttons panel that uses the sendInvoiceAction of the table.

{% highlight xml %}
<button id="sendInvoiceBtn"
        action="ordersTable.sendInvoiceAction"/>
{% endhighlight %}

In the controller, i created a SendInvoiceAction class that gets added to the orders table. SendInvoiceAction is a subclass of  ItemTrackingAction, which enables the feature of setting constraint code. The ItemTrackingAction will check the security constraint and enable / disable the action (and with the the button) accordingly.

In the class definition of the SendInvoiceAction i set the constraint code and soem other stuff like icon and caption.

{% highlight groovy %}

class OrderBrowse extends AbstractLookup {

    @Inject InvoiceService invoiceService
    @Inject Table<Order> ordersTable

    @Override
    void init(Map<String, Object> params) {
        ordersTable.addAction(new SendInvoiceAction())
    }

    protected class SendInvoiceAction extends ItemTrackingAction {

        SendInvoiceAction() {
            super("sendInvoiceAction");

            setConstraintOperationType(ConstraintOperationType.CUSTOM)
            setConstraintCode("sendInvoice")
            setIcon("font-icon:SEND_O")
            setCaption(formatMessage("sendInvoice"))
        }

        @Override
        void actionPerform(Component component) {
            invoiceService.sendInvoice(ordersTable.singleSelected)
            showNotification("Invoice send", Frame.NotificationType.TRAY)
        }
    }
}
{% endhighlight %}

The <code>actionPerform</code> method is the actual handler of the action. It will delegate the business logic to the InvoiceService.
