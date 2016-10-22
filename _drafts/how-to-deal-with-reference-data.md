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

First of all, although everyone might have a rough understanding about the term reference data, i had to look it up myself and look especially about the differences between reference data and master data. I found [this great explenation](http://www.semarchy.com/semarchy-blog/backtobasics_data_classification/) about the different data types. Mainly there are the following categories of data types:


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

In this article we will care about the reference data, because these oftentimes have a specific need on how to manage this data in general and over time.

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

The interesting part for this blog post are the reference data. <code>CustomerType</code> is the first of this in the row. In this entity we can classify a customer with entries like *new customer*, *potential customer*, *loyal customer* etc.
