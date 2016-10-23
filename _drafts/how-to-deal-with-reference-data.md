---
layout: post
title: How to deal with reference data
description:
modified: 2016-09-25
tags: [cuba, reference data]
image:
  feature: how-to-deal-with-reference-data/feature.png
---

Reference or master data is an important topic for every application. Let's have a look what build in options we have for creating these kinds of data when working with a CUBA application and what possibilities we have to extend this features to our own needs.

<!-- more -->

## Different categories of data

First of all, although everyone might have a rough understanding about the term reference data, i had to look it up myself and look especially about the differences between reference data and master data. I found [this great explanation](http://www.semarchy.com/semarchy-blog/backtobasics_data_classification/) about the different data types. Mainly there are the following categories of data types:


<figure style="float:right; padding: 10px; width: 300px;">
	<a href="{{ site.url }}/images/how-to-deal-with-reference-data/data-types.png"><img src="{{ site.url }}/images/how-to-deal-with-reference-data/data-types-small.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/how-to-deal-with-reference-data/data-types.png" title="Data categories as defined in the data management space">Data categories as defined in the data management space </a></figcaption>
</figure>

* reporting data (e.g. aggregated information about sales)
* transactional data (e.g. order information)
* master data (e.g. customer information that is used by transactional data)
* reference data (e.g. order status, so data that gets referenced and used)
* metadata (e.g. created timestamp information)

<img style="float:left; padding: 5px; margin-left:-85px; width:100px;" src="{{site.url}}/images/how-to-deal-with-reference-data/cubes/cube-je.png">

In this article we will care about the reference data, because these have oftentimes a specific need on how to manage this data in general and over time.

<br />

## Problems with reference data

I will not go into much detail about the theory behind master data management, first of all because it is a very broad field and secondly i'm not 100% into it :). Instead link you to a [starting point](https://en.wikipedia.org/wiki/Master_data_management) on data management. But to get a basic understanding around that topic, let's look at some instances of reference data and what potential problems arise when working with them.

So here's the example we will go through in this blog post. Let's imagine we have just another order management system. To manage orders we generally have the following data types that categories themselvs into the above mentioned groups:

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


The next example is the <code>TaxRate</code> entity. Tax rates not only have a name and a code, but a *rate* value as well. Additionally, as the tax rates are only valid for a certain time period, there is a need to define validities for this type of data. In this case, when working with this data, only certain entries are allowed to select depending on a particular point in time.

## Alphabet Inc. Ordermanagement

In order to have a look on how we can handle these kinds of situations within a CUBA application, i created an [example](https://github.com/mariodavid/cuba-example-temporal-reference-data) for the ordermanagement of the Alphabet Inc. With this we can have next to the normal reference data requirements a look at another interesting (imaginary) requirement: For certain users groups, only particlar entries of the refernce data should be visible.

First of all, let's take a look at the underlying domain model:

<figure class="center">
	<a href="{{ site.url }}/images/how-to-deal-with-reference-data/domain-model-temporal-reference-data.png"><img src="{{ site.url }}/images/how-to-deal-with-reference-data/domain-model-temporal-reference-data.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/how-to-deal-with-reference-data/domain-model-temporal-reference-data.png" title="Class diagram of the Alphabet Inc. ordermanagement system">Class diagram of the Alphabet Inc. ordermanagement system</a></figcaption>
</figure>

You'll find most of the above described entites. All reference entites the common  base class <code>ReferenceEntity</code>. In case of a reference entity with temporal validity information it is specialization <code>TemporalReferenceEntity</code>.
