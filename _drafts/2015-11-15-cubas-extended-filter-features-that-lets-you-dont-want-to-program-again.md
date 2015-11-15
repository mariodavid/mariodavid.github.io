---
layout: post
title: CUBA's extended Filter features that lets you don't want to program again
description: "..."
modified: 2015-11-13
tags: [cuba, filtering]
---

Last time, i wrote about the generic filter possibilities that CUBA provides out of the box. I left off with two requirements as a developer, that are necessary to be confident that the generic solution does not fall down with real world problems.

<!-- more -->



## Going from fast food to a real healthy meal

After going through the filter possibilities for myself the first time, it felt a little strange. This first thing i thought about it, was something like *"ok, so WTF did i spent my time on the last few years?"*. Because in the first place it is a little too good to be true. Don't need to write this simple, boring and similar filter mechanisms by yourself?


* *all orders that are in state X*
* *all products that are in category Y*
* *all orders that have been placed between t<sub>0</sub> and t<sub>1</sub>*
* *yada yada yada*

From a developer point of view this is just boring stuff. As you'll get the schema behind it, it becomes pointless. On the other hand, to wrap this generic schema into code that is efficient in terms of SQL statements and slick from UX perspective is actually pretty hard. This is why oftentimes the developer does not go from the specific problem to a more general implementation that can then be *used* instead of *developed*.

Going back to the solution we have at hand from the last blog post. After the first attempts to do some stuff with the filtering mechanism, i wasn't aware what to make of it. At this point it feels a little like *fast food*.

> OK at the exact moment. But after eating it, you often have a queasy conscience. First of all you probably know that it's going to bite you in the long run when buying a big mac every second day. Additionally you know that two hours later you probably will be hungry again.

The question basically is: *Is this is healthy well tasting meal?*. Meaning: Are you on the right track with this solution, or will this tool fall off after the first non-obvious filter problems?

For this to figure out i will go through the two objections from the last blog post.



## 1. pre-define filter combinations in behalf of the user

Many times there is a need for the developer to pre-define filter possibilities on behalf. You can think of it as different categories in terms of intervene through the user as well as the developer. Below you'll see a diagram that shows this classification.

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/pre-defied-filter-options.png"><img src="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/pre-defied-filter-options.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/pre-defied-filter-options.png" title="Different categories for pre-defined filter implementations">Different categories for pre-defined filter implementations</a></figcaption>
</figure>

Next, i'll go through the different categories of possibilities to pre-defined filters.

### 1.1 pre-defined generic filters

The first option that CUBA gives the user, it that a filter which is defined by the user as we saw in the last blog post, can be saved within the application. To do this, the Filter section has a little *Config* icon on the left, that lets the user choose between different possibilities to save a pre-defined filter.


<figure class="center half"><img src="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/save-filters-cuba.png" alt="">
	<figcaption>Configuration options for saving filters in CUBA</figcaption>
</figure>

CUBA lets you as a user decide between different options to save a filter, that you have created via the UI. The options differentiate in different use cases that should be fulfilled with it. 

With *Save / Save as* only the filter conditions (not the given values) will be saved for the current user. Next there is a possibility to save a filter as a *Search folder*. In this case, the entry will be shown in the side panel of the application. It stores the conditions as well as the input values and can be shared with other users. There is more stuff like *Sets*, which will let you save the results insted of the filter definition. 

I've just screached the surface of the different options - more information about the different possibilities will be found [here](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#gui_Filter).


### 1.2 custom UI with generic filters

The first option that i showed above, are from a UI perspective mostly the same. Oftentimes the need for a pre-defined filter comes from the wish to be more efficient as a user of the software to achieve the filter execution. If you as the developer want to enable the user to be even more efficient in the execution of its business process, a custom filter UI can be created that does exactly this.

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/custom-ui-entity-log-filter.png"><img src="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/custom-ui-entity-log-filter.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/custom-ui-entity-log-filter.png" title="Custom filter UI defined in the Entity Log view">Custom filter UI defined in the Entity Log view</a></figcaption>
</figure>

When you go through the [example app](https://github.com/mariodavid/cuba-ordermanagement), especially over the *administration section* of the app, you'll find different platform screen, where the generic filter solution is not used, but a custom UI for filtering instead. Above you see an example of the *Entity Log* view. 

As you see, there is actually no limit in customization of the UI. You can use all available [input elements](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#gui_components) as well as [layout](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#gui_layouts) possibilities to design the perfect UI for the execution of the users business process.

Alongside with the customization of the UI elements, there comes the need for the developer, to handle UI events that execute the filter. This is the trade-off that a developer has to make this kind of UI possible. 


### 1.3 programatically filtered datasources

The third option that you as a programmer have, is to adjust the underlying datasource of the filter. A [datasource](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#datasources) is a concept that is resonsible for the data that is provided to UI components. Of course you have programatic access to these datasources. But often is totally ok, to just configure these datasources through the declarative definition via the XML descriptors. An example of this definition you'll find in the [product-browse view](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/product/product-browse.xml) in the [ordermanagement app](https://github.com/mariodavid/cuba-ordermanagement):

{% highlight xml %}

<dsContext>
    <collectionDatasource id="productsDs"
                          class="com.company.ordermanagement.entity.Product"
                          view="product-view">
        <query>
            <![CDATA[select e from om$Product e]]>
        </query>
    </collectionDatasource>
</dsContext>
{% endhighlight %}

This datasource definition is used via the id attribute of the collectionDatasource XML Tag in the definition of the filter ui element. In this definition you have the option to first change the query string that should be executed. As this is a plain old JPQL Query, there is obviously a lot of freedom. A few examples can be found in the [docs](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#datasource_query). 


## 2. support for complex filters that go beyond attribute values of the or related entities

The second objection that i brought up last time, is the question about the support for filters that go beyond "simple" attribute based filterings. That is even more important for the decision to use the filter mechanisms as the basis for filter requirements for my application.

So let's have a look about what CUBA is capable of in this regard. Because it's a little bit harder to categories these advanced filter requirements, i'll go through a few examples, that should give an overview of what is possible out of the box.

### Products that have created within the last week

The first example that shows the dynamism of the filter feature is the possibility to add a custom condition. We'll define this condition with a criteria that uses a JQPL macro.

> Lets assume we want to filter for all products, that have created within the last week. For these products we want to track the success of the orders to do a marketing campaign if necessary

First of all we create a *new condition* via the UI (Product browser > Add search condition > Create new...). The name can be something like "from last week".

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/from-last-week-condition.png"><img src="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/from-last-week-condition.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/from-last-week-condition.png" title="Custom filter UI defined in the Entity Log view">Custom filter UI defined in the Entity Log view</a></figcaption>
</figure>

In the *Where* condition we'll use the JPQL macro @between to define, that the attribute *createdTs* has to be in the time frame (now-5) and now. *{E}* is the current entity (in this case *Product*).

	




