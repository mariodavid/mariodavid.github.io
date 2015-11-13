---
layout: post
title: The generic filter in cuba platform – excel filters on steorids
description: "Starting the series of CUBA features with this blog post about the generic filter solution that the platform allows the user to filter entity lists on quite every possible facet."
modified: 2015-11-06
tags: [cuba, filtering]
---

As i promised [last](http://www.road-to-cuba-and-beyond.com/my-personal-crud-story-or-how-i-came-to-cuba/) time, i plan to go through some features of the platform, that i think are very valuable. So i’m going to do a little series here. Starting with the obvious one’s like UI, Filtering, Security to some advanced features like Web Portal, extensibility, Auditing, dynamic attributes and so on.

<!-- more -->

## CUBA features #1 – generic filter

In this blog post i want to cover is the generic filter solution. But before getting our hands dirty with the technical details, let’s get started with looking into the underling use cases that this feature addresses.


## how the user get's the data that's actually needed

We will stick with the example from last time:

<figure class="center">
	<a href="{{ site.url }}/images/my-personal-crud-story/domain-example-customer-orders-uml1.png"><img src="{{ site.url }}/images/my-personal-crud-story/domain-example-customer-orders-uml1.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/my-personal-crud-story/domain-example-customer-orders-uml1.png" title="A domain example with a customer – orders relationship">A domain example with a customer – orders relationship</a></figcaption>
</figure>


Based on this entity model, let’s think of some possible filter requirements, that users might have.

First, there are certain Filters on the entity itself and its direct attributes:

* Show all Customers in NY
* Show all Orders in 2015
* Show all Products with a price of min. 350$
* Show all Orders that are in state “COMPLETED”

Next, we have the filtering that are based on 1:1 / N:1 associations:

* List all Orders from Customer “Mario David”
* List all Customers that live in Dallas (via the Address Entity)
* Show all Products in the Category “Notebooks”

And then, we have the filters, that are based on a 1:N / M:N relationship:

* List all Customers that have at least one order in 2015
* List all Orders with at most five line items
* List all Orders where the price sum of the line items is greater then 200$

This are basically the categories of filter requirements, that fulfill perhaps about 80% of the use cases.

## A programatic way to solve these kind of problems

The way i would normally handle these kind of requirements is the following:
First of all, i would start trying to get my head around what the user actually wants to achieve with this filtering. Usually it is used to just reduce the amount of entity-instances for the current workflow, that is executed. An example of this would be “to filter for just the orders have not paid in time” – the workflow in this case would be something like “Sending overdue notices”. Another way to use filtering is if the result is a basis for reporting (this is not covered in this blog post).

Whatever the reason is, when i know what the filter criteria is, one easy solution for me as a programmer is to implement it right away. If we think of an implementation in, for example Grails i would come up with something like this in the backend:

{% highlight java %}

class OrderController {

    //...

    Date now = new Date()
    respond Order.where { dueDate > now }.list(params)

    //...
}

{% endhighlight %}

Which will do the job in a simplified manner. On the front-end, a drop-down Box or toggle button would work out. Another possibility would be to get to that data via a link, that will keep the information in it.

Whatever the implementation would look like – the point on this whole solution is, that I as the developer has to know of this filter requirement upfront, because it has to be implemented programatically.

## Generic solutions for these problems

Instead of directly implement the filter solution as needed, a more general approach to it is also quite common. In this case, the developer does not know the filter requirements upfront, but rather let the user decide what to search / filter for. To get that going, possible filter conditions have to be inferred from the underlying datatypes of the attributes.

This model can be seen as similar to excel [filtering](https://support.office.com/en-us/article/Filter-data-in-an-Excel-table-7d8e9739-2898-4bfe-9d0f-c6204e6e5c8a) mechanisms. Depending on the datatype of the current column, excel provides filter possibilities that make sense in this context. A date can be filtered to be in a certain range, a number has to be more than a given number, a string can contain a certain substring and so on. Since Excel does not [really](https://support.office.com/en-us/article/VLOOKUP-function-0bbc8083-26fe-4963-8ab8-93a18ad188a1) know about entities and relationships, it is not possible to search / filter over associations. So this filter mechanism is only valuable to a certain degree.

## What CUBA brings to the table

So here comes CUBA and tells us, that there is a “generic filter” in it, that let us filter for mostly any kind sub-data we would like. Lets have a deeper look at it.


<figure class="center">
	<a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/generic-filter-cuba.png"><img src="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/generic-filter-cuba.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/generic-filter-cuba.png" title="Example of the generic filter in CUBA for the products">Example of the generic filter in CUBA for the products</a></figcaption>
</figure>

I have created a [demo app](https://github.com/mariodavid/cuba-ordermanagement), that is an implementation of the above mentioned domain model. Here we see the list of products available in our store. Atop of the data table, you’ll notice the filter section, that lets you define your filters for this table. The link “Add search condition” looks at the underlying entity, in this case *Product* and shows all of it. Actually not only the direct attributes of the entity are shown, but also the related entities and their attributes (and their related entities and their attributes and …).

After selecting some of the available attributes, the filter section in the table is filled with corresponding condition boxes. Depending on the attribute type, the possibilities to define the condition will vary.

Here is an example of such a combination of filter conditions:

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/selected-filter-conditions-examples.png"><img src="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/selected-filter-conditions-examples.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/selected-filter-conditions-examples.png" title="Select all products that start with Apple in Category Smartphone, which have changed since 2/11/15">Select all products that start with Apple in Category Smartphone, which have changed since 2/11/15</a></figcaption>
</figure>

The filter conditions can be in different modes depending on the attribute type. A text attribute might *start with* or *contains* a given text and so on. Dates can be filtered with the corresponding date filter possibilities like *before* or *after* a given date. Enums as well as Many to One associations will be selectable via a drop down list. Condition modes for this type are *is set*, *in list*, *=* etc. I could go on describing the different datatypes and their filter modes, but i’ll leave it for now. If you want to see all the possibilities, you’ll find a pretty good documentation [here](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#gui_Filter).

What i have shown for now is pretty much just the surface of the possibilities CUBA gives the user and the developer regarding filtering. When thinking about it though, it is quite a bit of functionality that empowers the user to do the filtering on their own.

As you see, at a first glance, there is very little that prevents me as a developer from letting the user decide about the required filter possibilities instead of having to implement the different possibilities by hand.

## What do i have to do, to make this work?

OK, so the interesting question to ask might be, how much effort is it to implement this feature as a developer. To see this you’ll have to look into the UI definition [file](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/product/product-browse.xml) for the product list. Basically it goes something like this:

{% highlight xml %}

<filter id="filter" datasource="productsDs">
  <properties include=".*"/>
</filter>

{% endhighlight %}

**That’s it**. Actually not, because you’ll have do define the *productsDs* datasource definition you’ll find in the [XML descriptor](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/product/product-browse.xml) as well.

To be even more precise, you’ll often don’t write the definition yourself. Instead you’ll use CUBA [Studio](https://demo.cuba-platform.com/studio/) to do the plumbing.

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/studio-product-entity.png"><img src="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/studio-product-entity.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/studio-product-entity.png" title="CUBA Studio showing the Product Entity with the possibilites to generate screens">CUBA Studio showing the Product Entity with the possibilites to generate screens</a></figcaption>
</figure>

In this case, you’ll fire up your local studio installation (and do a git clone from the [example project](https://github.com/mariodavid/cuba-ordermanagement)), open the project, look at your product entity (as you see above) and tell it to generate standard views for it. After answering a few questions about the different options of this generation step, it will lead to the exact XML descriptor file for the list view of this entity including the filter possibilities.

The platform has solutions for these objections. I’ll cover both in the next parts of this series.