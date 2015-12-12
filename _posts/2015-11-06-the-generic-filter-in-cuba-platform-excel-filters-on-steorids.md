---
layout: post
title: The generic filter in cuba platform – excel filters on steroids
description: "Starting the series of CUBA features with this blog post about the generic filter solution that the platform allows the user to filter entity lists on quite every possible facet."
modified: 2015-11-06
tags: [cuba, filtering]
image:
  feature: 2015-11-06-the-generic-filter-in-cuba/feature.jpg
  feature_source: https://pixabay.com/de/ausbildung-langhantel-muskeln-h%C3%A4nde-603981/ 
  feature_source_2: https://www.iconfinder.com/icons/272703/excel_icon#size=128

---

As was promised [last](http://www.road-to-cuba-and-beyond.com/my-personal-crud-story-or-how-i-came-to-cuba/) time, I'm going to run through some features of the [platform](https://www.cuba-platform.com/), which I think are very valuable. I’m going to write a series of sub-articles here, starting with the elementary aspects, such as UI, filtering, security for some advanced parts like web portal, extensibility, audit, dynamic attributes and so on.

<!-- more -->

## CUBA features #1 – generic filter

In this blog post I want to cover the generic filter functionality. But before getting our hands dirty with technical details, let’s get started with looking into the underlying use cases that this feature addresses.


## How the user gets data that’s actually needed

We will stick with the example from the previous article:

<figure class="center">
	<a href="{{ site.url }}/images/2015-10-29-my-personal-crud-story/domain-example-customer-orders-uml1.png"><img src="{{ site.url }}/images/2015-10-29-my-personal-crud-story/domain-example-customer-orders-uml1.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-10-29-my-personal-crud-story/domain-example-customer-orders-uml1.png" title="A domain example with a customer – orders relationship">A domain example with a customer – orders relationship</a></figcaption>
</figure>


Based on this entity model, let’s think of some possible filter requirements that users might have.

First, there are certain filters on the entity itself and its direct attributes:

* show all customers in NY
* show all orders in 2015
* show all products with a price of at least 350$
* show all orders that are in state 'COMPLETED'

Next, we have the filtering that are based on 1:1 / N:1 associations:

* list all orders from customer 'Mario David'
* list all customers, who live in Dallas (via the address entity)
* show all products under the 'Notebooks' Category

And then, we have the filters, that are based on a 1:N / M:N relationship:

* list all customers, who have at least one order in 2015
* list all orders having at most 5 line items
* list all orders where total amount of line items is greater than 200$

These are general categories for filter requirements, that cover probably about 80% of use cases.

## The programatic way to solve these kind of problems

The way I would normally use to handle these kind of requirements is the following: To commence with, I would start trying to get my head around what a user actually wants to achieve by filtering. Usually filtering is used to reduce a number of entity-instances for the current workflow. An example of this would be ‘filter only the orders, which have not been paid in time’ – the workflow in this case would be something like ‘sending overdue notice’. Another way to use filtering is to use the result as a basis for reporting (this case is not covered in this blog post).

Whatever the reason is, when I know what the filter criteria is, the easiest solution for me, as a developer, is to code it right away. To give an implementation example, in Grails, I would come up with something like this on the backend:

{% highlight java %}

class OrderController {

    //...

    Date now = new Date()
    respond Order.where { dueDate > now }.list(params)

    //...
}

{% endhighlight %}

This piece of code might already do the job. On the frontend, a drop-down box or toggle button would work out. Another possibility would be getting data via a link, that will keep the filter information in it.

Whatever the implementation would look like – the fundamental aspect for me as the developer, is the need to know the requirements upfront, because it has to be implemented programmatically.

## Generic solutions for these problems

Instead of directly implementing the filter solution, there is a more general approach, which is also quite common. In this case, a developer does not know the filter requirements upfront, but rather lets a user decide what to search or filter for. To get that going, possible filter conditions have to be inferred from the underlying datatypes of the attributes.

This model can be seen as similar to the excel [filtering](https://support.office.com/en-us/article/Filter-data-in-an-Excel-table-7d8e9739-2898-4bfe-9d0f-c6204e6e5c8a) mechanism. Depending on the datatype of the current column, Excel provides filter possibilities that make sense in this context. A date can be filtered to be in a certain range, a number has to be ‘greater than’ or ‘lower than’, a string can contain a certain substring and so on. Since Excel does not [really](https://support.office.com/en-us/article/VLOOKUP-function-0bbc8083-26fe-4963-8ab8-93a18ad188a1) know about entities and relationships, it is not possible to search or filter over associations. So, this filter mechanism is only applicable to a certain degree.

## What CUBA brings to the table

So, here comes CUBA and tells us: there is a ‘generic filter’, which lets you filter for mostly any kind of sub-data set we would think of. Now, to take a deeper look at it:


<figure class="center">
	<a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/generic-filter-cuba.png"><img src="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/generic-filter-cuba.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/generic-filter-cuba.png" title="Example of the generic filter in CUBA for the products">Example of the generic filter in CUBA for the products</a></figcaption>
</figure>

I have created a [demo app](https://github.com/mariodavid/cuba-ordermanagement), that implements the domain model I mentioned above. Here we see the list of products available in our store. Atop of the data table, you’ll find the filter section, that enables you to define your filters for this table. On clicking the ‘Add search condition’ link, CUBA filter retrieves and shows all the related attributes of the underlying entity, in this case Product. Actually, not only the local attributes of an entity comes up, the filter mechanism also drills down to the related entities and their attributes (and their related entities and their attributes and …).

After selecting some of the available attributes, the filter section in the table is filled with corresponding condition boxes. Depending on the attribute types, ways to define conditions vary.

Here is an example showing the combination of different filter conditions:

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/selected-filter-conditions-examples.png"><img src="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/selected-filter-conditions-examples.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/selected-filter-conditions-examples.png" title="Select all products that start with Apple in Category Smartphone, which have changed since 2/11/15">Select all products that start with Apple in Category Smartphone, which have changed since 2/11/15</a></figcaption>
</figure>

The filter conditions can be in different modes dependent on the attribute type. A text attribute filter could be for example: *start with* or *contains* a given text and so on. Dates can be filtered with the corresponding date filter possibilities, like *before* or *after* a given date.  Enums as well as N:1 associations will be selectable via a drop down list. Condition modes for this type are: *is set*, *in list*, *=* etc. I could go on and on describing the different data types and their filter modes, but I’ll leave it for now. If you want to see the significant power of these filters, you’ll find a pretty good documentation [here](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#gui_Filter).

What I have shown for now, is pretty much just the surface of the facilities, that CUBA provides to a user and a developer regarding filtering. When thinking about it though, it is also quite a bit of functionality that empowers a user to do filtering on their own.

As you can see, there is nothing that prevents me, as a developer, from letting a user decide about the required filter conditions, instead of having to implement those by hand.

## What do i have to do, to make this work?

Ok, so an interesting question to ask might be, ‘How much effort is it to implement this feature as a developer?’  To see this you’ll have to look into the UI definition [file](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/product/product-browse.xml) for the list of products screen. Basically it goes something like this:

{% highlight xml %}

<filter id="filter" datasource="productsDs">
  <properties include=".*"/>
</filter>

{% endhighlight %}

**That’s it**. Actually not, because you’ll have to define the productsDs datasource, which you can find in the [XML descriptor](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/product/product-browse.xml) as well.

To be even more precise, you often don’t need to write the definition yourself. Instead you can use the CUBA [Studio](https://demo.cuba-platform.com/studio/) to do scaffolding.

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/studio-product-entity.png"><img src="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/studio-product-entity.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-06-the-generic-filter-in-cuba/studio-product-entity.png" title="CUBA Studio showing the Product Entity with the possibilites to generate screens">CUBA Studio showing the Product Entity with the possibilites to generate screens</a></figcaption>
</figure>

In this case, you’ll need to [download](https://www.cuba-platform.com/download) and install the CUBA Studio, do a git clone from the [example project](https://github.com/mariodavid/cuba-ordermanagement)), open the project, look at your *Product* entity (as you see above) and tell the studio to generate standard screens for it. After answering a few questions about the different options of this generation step, it will lead to the exact XML descriptor file for the list browser of this entity including filtering functionality.

## What’s about the real heavy stuff?

After going through this, two things came to my mind. First, this is just for kind of ad-hoc filtering scenarios. How can I pre-define these filters, so that my users do not have to pick them up by themselves over and over again? The second thing is that there are often filtering requirements that go beyond the described possibilities. How is this tackled by CUBA?

The platform has solutions for these objections as well. I’ll cover both in the next part of this series.