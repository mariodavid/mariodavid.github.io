---
layout: post
title: How to deal with reference data
description:
modified: 2016-09-25
tags: [cuba, reference data]
image:
  feature: how-to-deal-with-reference-data/feature.png
---

Reference or master data is an important topic for every application. Let's have a look at what build-in options we have for creating these kinds of data when working with a CUBA application. Additionally: what possibilities do we have to extend this features to our own needs?

<!-- more -->

## Different categories of data

First of all, although everyone might have a rough understanding about the term reference data, i had to look it up myself and look especially about the differences between reference data and master data. I found [this great explanation](http://www.semarchy.com/semarchy-blog/backtobasics_data_classification/) about the different data types. Mainly there are the following categories of data types:

<img style="float:right; padding: 5px; margin-right:-85px; width:100px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-je.png">

* reporting data (e.g. aggregated sales data)
* transactional data (e.g. order information)
* master data (e.g. customer information)
* reference data (e.g. order status)
* metadata (e.g. created timestamps)

In this article we will care about the reference data, because these have oftentimes a specific requirements.



<figure class="center">
	<a href="{{ site.url }}/images/how-to-deal-with-reference-data/data-types.png"><img src="{{ site.url }}/images/how-to-deal-with-reference-data/data-types-small.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/how-to-deal-with-reference-data/data-types.png" title="Data categories as defined in the data management space">Data categories as defined in the data management space </a></figcaption>
</figure>


## Problems with reference data

I will not go into much detail about the theory behind master data management, first of all because it is a very broad field and secondly i'm not 100% into it :). Instead i'll link you to a [starting point](https://en.wikipedia.org/wiki/Master_data_management) on master data management. But to get a basic understanding around that topic, let's look at some instances of reference data and what potential problems arise when working with them.

So here's the example we will go through in this blog post. Let's imagine we have just another order management system. To manage orders we generally have the following data types that can be categorized into the above mentioned groups:

<img style="float:right; padding: 10px; margin-right:0px; width:150px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-cm.png">

* **transactional data**
  * Order
* **master data**
  * customer
* **reference data**
  * customer type
  * payment method
  * tax rates
  * tenant information

The interesting part for this blog post are the reference data. <code>CustomerType</code> is the first of this in the row. In this entity we can classify a customer with entries like *new customer*, *potential customer*, *loyal customer* etc. These classifications of customers are there for e.g. reporting purposes. The general problem with reference data is that it is very sensible towards changes.

Imagine what happens when the entry with the name "new customer" is changed to "Impulsive customer". In this case not only new customer entries will be able to select this entry as the customer type, but all existing customers that had been previously classified as "new customer" are now "Impulsive". Is this really what the reference data changer thought of when doing the change? Probably not.

<img style="float:right; padding: 10px; margin-right:-90px; width:150px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-pk.png">


The next example is the <code>TaxRate</code> entity. Tax rates not only have a name and a code, but a *rate* value as well. Additionally, as the tax rates are only valid for a certain time period, there is a need to define validities for this type of data. In this case, when working with this data, only certain entries are allowed to be selected depending on a particular point in time.

## Alphabet Inc. Ordermanagement

In order to have a look on how we can handle these kinds of situations within a CUBA application, i created an [example](https://github.com/mariodavid/cuba-example-temporal-reference-data) for the ordermanagement of the Alphabet Inc. With this example we will care about the three requirements:

1. Handle deactivated / deleted ReferenceEntity references
2. Filter only for currenty valid TemporalReferenceEntities
3. Display only allowed CustomerTypes per tenant

First of all, let's take a look at the underlying domain model:

<figure class="center">
	<a href="{{ site.url }}/images/how-to-deal-with-reference-data/domain-model-temporal-reference-data.png"><img src="{{ site.url }}/images/how-to-deal-with-reference-data/domain-model-temporal-reference-data.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/how-to-deal-with-reference-data/domain-model-temporal-reference-data.png" title="Class diagram of the Alphabet Inc. ordermanagement system">Class diagram of the Alphabet Inc. ordermanagement system</a></figcaption>
</figure>

You'll find most of the above mentioned entites. All reference entites are subclasses of the common  base class <code>ReferenceEntity</code>. In case of a reference entity with temporal validity information it is the specialization <code>TemporalReferenceEntity</code>. <code>PaymentMethod</code> and <code>TaxRate</code> have validity information while <code>CustomerType</code> is a normal reference entity.


### Handle deactivated / deleted ReferenceEntity references

The first thing that we have to be aware of is the fact, that reference entites can change over time. New entries get created and other entries get obsolete. Let's look at the problematic situation of a reference entity that gets removed.

The immediate question that comes up then: what happens to the transactional data that has a reference to the ReferenceEntity instance. Let's look at the Customer entity for this example: In this case, the reference data would be the CustomerType. What happens if the CustomerType gets removed from the system?

<img style="float:left; margin-right: 50px;padding: 10px; margin-left:-90px; width:150px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-xz.png">


**There are the main options:**

1. Remove the customers from the system that have this CustomerType
2. Remove the reference to the CustomerType for all these customers
3. Soft-delete the CustomerType and let the customer references untouched

For transactional data, it might not even be a big deal, but if you start thinking about statistics and historical information it is absolutely crucial that these transactional data stay intact.

When you have a statistic that gives you a pie chart of the turn over share for all customers in 2015 separated by their CustomerType. In 2016 you remove the main CustomerType with all its customers. Now, when you execute the 2015 statistic again, the pie chart will change significantly. Therefore removing all customers like in 1. will not work out.

Depending on the type of statistic you want to make the second option would not work as well. Let's imagine we want total turn overs only for a certain customer region. In this case, the statistics be infected by the change as well.

So both options are probably not what we want for historical data.

Let's take a look at the third option. In this case the data will not deleted at all. Instead, the they will be tagged as deleted (e.g. via <code>deleteTs</code> and  <code>deleteBy</code> attributes). With this the transactional data will stay intanct and the statistics will remain the same, doesn't matter how often and when they are executed.


#### Soft deletion in CUBA

So. let's try to implement this one in CUBA. Soft deletion is a feature of the platform that is already inplace if we use <code>StandardEntity</code> as a base class for our entities. In our case CustomerType is soft deletable.

The default behavior of CUBA in the case of a deletion of a soft deletable entity is as follows:

* The instance will not show up in the browse screens of the entity anymore
* references from the other entities will be displayed normally (probably the instance name)
* In case of the editor of the entity with the reference, when using the PickerField, in the looup screen the option will not be available anymore

Basically this is exactly what we want in this case. The deleted CustomerType "new Customer" is stopped from using it in the future (new customers or changing existing customers), but existing entries remain correct (because customer "Juan Perkins" has be classified as a "new Customer" before). You can see the result in the pictures below.

<table><tr><td><img style="float:left; padding: 10px; width:300px;" src="{{site.url}}/images/how-to-deal-with-reference-data/customer-list-without-new-customer.png"></td><td><img style="float:left; padding: 10px; width:300px;" src="{{site.url}}/images/how-to-deal-with-reference-data/customer-with-deleted-customerType.png">
</td></tr></table>

After this is working out, the next requirement that we can take a look at is how references are handled that are only valid within a certain time area.

### Filter only for currenty valid TemporalReferenceEntities
The main difference to the example above goes like this: When i edit a entity, that has a reference to another entity - i should be able to select the instances that have been valid at this point in time.

An example of this would be: An order has a payment method. These payment methods may change overtime, because new payment methods are added as the time goes on. But when i want to change the payment method for an existing order, i should only be allowed to see the payment methods that have been available at the time of the order.


<img style="float:right; padding: 10px; margin-right:-90px; width:150px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-je.png">

To get that going, the <code>PaymentMethod</code> is a subclass of <code>TemporalReferenceEntity</code>, that additionally has the two major attributes <code>validFrom</code> and <code>validUntil</code>. Instead of deleting an entity instance, it will be invalidated via setting the validUntil date.

This means, that we have to filter our default lookup list of the payment methods to see only the entries that are valid at a given reference date (which in case of the order is the orderDate).

For this to work i created a common superclass for the controllers of the browse screen: [TemporalReferenceEntityBrowse](https://github.com/mariodavid/cuba-example-temporal-reference-data/blob/master/modules/web/src/com/company/cetrd/web/reference/TemporalReferenceEntityBrowse.groovy). It takes a Date as a window parameter that get used to filter the entries like this:

{% highlight groovy %}
    @WindowParam
    Date validReferenceDate

    @Override
    void init(Map<String, Object> params) {
        if (validReferenceDate) {
            String datasourceQuery = 'select e from ' + getEntityName() + ' e where @dateBefore(e.validFrom, :param$validReferenceDate) and (@dateAfter(e.validUntil, :param$validReferenceDate) or e.validUntil is null)'
            getDatasource().setQuery(datasourceQuery)
        }
    }

    abstract CollectionDatasource<TemporalReferenceEntity, UUID> getDatasource();
    abstract String getEntityName();
{% endhighlight%}

With this, the datasource filter query changed to show only the entries that are valid at the given reference data. The subclasses of this are only required to define the datasource and the entity name with the corresponding abstract methods (see [PaymentMethodBrowse](https://github.com/mariodavid/cuba-example-temporal-reference-data/blob/master/modules/web/src/com/company/cetrd/web/reference/paymentmethod/PaymentMethodBrowse.groovy) for more details).

To pass the order data as the reference data to the lookup screen, take a look at the [OrderEditor](https://github.com/mariodavid/cuba-example-temporal-reference-data/blob/master/modules/web/src/com/company/cetrd/web/order/OrderEdit.groovy#L33), which does exactly that:

{% highlight groovy %}
protected void activatePickerField(PickerField pickerField, Date validReferenceDate) {

    PickerField.LookupAction lookupAction = (PickerField.LookupAction) pickerField.getAction("lookup")
    lookupAction.lookupScreenParams = [validReferenceDate: validReferenceDate]
    pickerField.enabled = true
    pickerField.description = ""
}
{% endhighlight%}


With this, the exact same behavior from above is achieved.
When a new order gets created, it will show only the payment methods that are valid at the given order date. If the reference data change (an entry gets deactivated), it will remain in the reference for existing orders.

Depeding on the use case, the reference data might not be a particular data within an entity. There is another example of this in the application: The customer can have a preffered payment method that should be used. In this case, there is no reference date like the orderDate, so in this case the current date is used to get only the currently valid options (see the [CustomerEditor for details](https://github.com/mariodavid/cuba-example-temporal-reference-data/blob/master/modules/web/src/com/company/cetrd/web/customer/CustomerEdit.groovy#L33))

### Display only allowed CustomerTypes per tenant

As a third example, that is only partly related but nevertheless very interesting is the following situation:

Let's imagine we are in a multi-tenant environment (like in the [cuba-sample-saas](https://github.com/cuba-platform/sample-saas) example). In this case, we have the ordermanagement for Alphabet Inc. Since this holding has different companies that are part of it, the application needs to have distinct ordermanagement systems for each company (tenant). A tenant on it's own is allowed to define which customer types are relavant in it's company.

<img style="float:right; padding: 10px; margin-right:-90px; width:150px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-pk.png">


The first option to solve this issue is to let every company create their own entries for CustomerType - this would definitivly get the job done. CustomerType would be a subclass of <code>TenantEntity</code> and there we go.

But let's imagine, there are so many CustomerTypes and it is required that it is possible to make statistics over the customers of all companies within the holding. In this case, let every company define their own CustomerTypes wouldn't work.

So we take another approach. We will share the CustomerTypes for the whole holding. The companies are only allowed to blacklist some of the entries so that they will not get displayed since it might not make sense in their business to have such customer types.

*Here's how we can achieve it:*

As described in the entity model from above, we create a class [TenantCustomerType](https://github.com/mariodavid/cuba-example-temporal-reference-data/blob/master/modules/global/src/com/company/cetrd/entity/reference/tenant/TenantCustomerType.java) that is responsible for holding the blacklist of every tenant for the CustomerTypes.

Next, we have will create a Security Group constraint that will kick out every CustomerType via security mechanisms that is in the list of the CustomerType blacklist like this:


<figure class="center">
	<a href="{{ site.url }}/images/how-to-deal-with-reference-data/tenant-customerType-constraint.png"><img src="{{ site.url }}/images/how-to-deal-with-reference-data/tenant-customerType-constraint.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/how-to-deal-with-reference-data/tenant-customerType-constraint.png" title="Security constraint to disable all blacklisted customer types per tenang">Security constraint to disable all blacklisted customer types per tenang</a></figcaption>
</figure>


<img style="float:left; padding: 10px; margin-left:-90px; margin-top:150px; width:150px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-da.png">

Now, when we login as a specific tenant (google:google or fiber:fiber), we will only see and be able to assign references to non-blacklisted CustomerTypes.

With this, we have finished our three requirements for the Alphabet Inc. Ordermanagement system regarding reference data. 
