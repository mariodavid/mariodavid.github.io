---
layout: post
title: Views - the uncharted mystery
description: "The views approach is "
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: views-the-uncharted-mystery/feature.jpg
---

Views are a concept of CUBA that is not that widespread in the web framework world and to understand it means to prevent running into weired issues around not-loaded data and applications that therefore stop working. Let's look at the idea behind views and why they are actually pretty neat.

<!-- more -->

## The problem of not loaded data

To get an easy start into topic, lets take the following example. Imagine, you have a <code>Customer</code> entity that references a <code>CustomerType</code> entity as a N:1 association, meaning that for a customer you can reference a type that describes the customer like "cash cow", "boor dog" etc. The CustomerType has a name attribute where the actual name is stored.

One of issues probably most of CUBA starters (and advanced users) have run into is the following issue:

> IllegalStateException: Cannot get unfetched attribute [type] from detached object com.rtcab.cev.entity.Customer-e703700d-c977-bd8e-1a40-74afd88915af [detached].

<figure class="center">
	<a href="{{ site.url }}/images/views-the-uncharted-mystery/unfetched-attribute-error.png"><img src="{{ site.url }}/images/views-the-uncharted-mystery/unfetched-attribute-error.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/views-the-uncharted-mystery/unfetched-attribute-error.png" title="Cannot get unfetched attribute error in the CUBA UI">Cannot get unfetched attribute error in the CUBA UI</a></figcaption>
</figure>


Did you ever get some kind of this error message? I certainly have in a ton of different situations. In this article we will take a look at the root cause of the problem, why it exists and how to resolve it.

To give you a little introduction about the topic, let's look at the concept of a view.

## What is a view?

A view in CUBA is basically a set of database columns that should be loaded together in one query.

Lets imagine we want to create a UI that has a table of customers where the first column is the name of the customer and the second column is the name of the customerType (like in the picture above). Taking the entity model from above, you end up with two database tables (one for Customer and one for CustomerType). If you do a <code>SELECT * from CEV_CUSTOMER</code> you are only able to get the data within this table (like the name attribute e.g.). To get data from other tables you have to use [SQL JOINS](https://www.w3schools.com/sql/sql_join.asp) in order to get data from multiple tables.

In SQL when using JOIN, the hierachy of the associations is flatted into a list instead of a graph.

In JPQL (what is used by CUBA through JPA), the graph of data can be preserved, so that the java code can easily work with its entity representations and the graph relationship between them.

Nevertheless, the data has to be fetched from the database somehow and transformed into the entity graph. To do that, in the object-relational mapping space (which JPA is) there are two major approaches to query the database.


## Lazy loading vs. eager fetching

Lazy loading and eager fetching are two strategies to retrieve the desired data from the database. They distingluish themselves in the question, *when* the data of referenced tables are loaded. To understand what that means, imagine the following situations:



<div class="hobbit-scene">Do you remember the scene from the book "The Hobbit" where the expedition group of the dwarves together with Gandalf and Bilbo tries to get a bed in Beorns house for a couple of days? Gandalf told the dwarves to only come one at a time after he already talked to Beorn carefully and introduced them one by one, to not make Beorn freak out about the fact that he has to take care of 15 guests.
<br /><br />
<figure class="center">
	<img src="{{ site.url }}/images/views-the-uncharted-mystery/feature2.jpg" alt="">
	<figcaption>Gandalf with the dwarves at Beorns house...</figcaption>
</figure>

This is not the most obvious way to think about the <i>lazy loading</i> and <i>eager fetching</i>, but it certainly has the same characteristics. Gandalf was very wise in doing so, because he knew the constraints. He basically actively decided to do <i>lazy loading</i> because he new that loading all the data from the database would be too heavy for the database. After the 8th dwarf he switched to <i>eager fetching</i> to get the last four dwares in the room, because he noticed that querying the database too often will not lead to the goal either.
</div>


The takeaway of this is, that both options have their own strengths and weaknesses. It is up to you, to decide which one is more accurate in which situation.

## N+1 query problem

The N+1 query problem oftentimes occurs when using lazy loading all over the place without actively thinking about it. To illustrate that, let's have a look at a snippet of Grails code. This does not mean that in Grails everything is lazy loaded (its actually up to you to decide). In Grails your database requests will return instances of the Entity, with all attributes of the table loaded with it. It basically makes a "SELECT * FROM Pet". When you want to traverse a relationship between entities you do that afterwards. Here's an example:

{% highlight groovy %}
function getPetOwnerNamesForPets(String nameOfPet) {
  def pets = Pet.findAll(sort:"name") {
       name == nameOfPet
  }
  def ownerNames = []
  pets.each {
    ownerNames << it.owner.name
  }

  return ownerNames.join(", ")
}
{% endhighlight %}

It is a single line of code that will do the traversal here: <code>it.owner.name</code>. Owner is the relationship that has not been loaded in the first request (<code>Pet.findAll</code>). So for each call of this line, GORM will something like "SELECT * FROM Person WHERE id='...'". This is called *lazy loading*. When you count the SQL queries you will end up at N (the person for each invocation of <code>it.owner</code>) + 1 (for the initial <code>Pet.findAll</code>). If you traverse your entity graph further, you will most likely pushing your database to the edge of what is possible.

As a application developer you probably don't really notice this, because you might think that you will only traverse the object graph.

This implicity with a single line of code hitting the database really hard is what makes lazy loading somewhat dangerous.

## Eliminating N+1 queries through CUBA views

In CUBA the N+1 query problem most likely never occurs, because the framework decided to not implicitly do lazy loading. Instead CUBA has the notion of "views". Views are basically a definition of what attributes have to be fetched and will be loaded in the entity instances. This would be something like: <code>SELECT pet.name, person.name FROM Pet pet JOIN Person person ON pet.owner == person.id</code>

A view on the one hand represents the column that gets fetched from the own table (Pet) (instead of everything via *), on the other hand represents the columns that have to be loaded via a JOIN.

You can think of a CUBA view as a [SQL view](https://www.w3schools.com/sql/sql_view.asp) for the OR-Mapper since it acts pretty much in the same way.

In CUBA you cannot make a SQL call through the [DataManager](https://doc.cuba-platform.com/manual-6.5/dataManager.html) without using a view. Let's look at the example from the docs for this:

{% highlight java %}
@Inject
private DataManager dataManager;

private Book loadBookById(UUID bookId) {
    LoadContext<Book> loadContext = LoadContext.create(Book.class)
            .setId(bookId).setView("book.edit");
    return dataManager.load(loadContext);
}
{% endhighlight %}

In this case we want to load a book via its id. The method <code>setView("book.edit")</code> in the Load context creation defines what view to use when fetching the database. In case you don't define the view, the data manager will use one of the two standard views that exists for every entity: the <code>_local</code> view. Local means that every attribute that is not a reference to another table will be loaded, nothing else.

## Solving the IllegalStateException with views

To get back to our example from above with the knowledge about views, let's take a look how to resolve the issue.

The error message <code>IllegalStateException: Cannot get unfetched attribute [] from detached object</code> just means, that there is an attribute that you want to display which is not part of the view that you are using for this entity. When we look at the [browse screen](https://github.com/mariodavid/rtcab-cuba-example-views/blob/1-unfetched-attribute-in-table/modules/web/src/com/rtcab/cev/web/customer/customer-browse.xml#L11) i used the <code>_local</code> view - which is exactly the problem:

{% highlight xml %}
<dsContext>
    <groupDatasource id="customersDs"
                     class="com.rtcab.cev.entity.Customer"
                     view="_local">
        <query>
            <![CDATA[select e from cev$Customer e]]>
        </query>
    </groupDatasource>
</dsContext>
{% endhighlight %}


To get rid of the error message, the first thing we need to do is to include our customer type into our view. Since we cannot change the <code>_local</code> view, we can create our own. You can either do it via studio like this (right click on the entity > create view):

<figure class="center">
	<a href="{{ site.url }}/images/views-the-uncharted-mystery/view-definition-in-studio.png"><img src="{{ site.url }}/images/views-the-uncharted-mystery/view-definition-in-studio.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/views-the-uncharted-mystery/view-definition-in-studio.png" title="Cannot get unfetched attribute error in the CUBA UI">Cannot get unfetched attribute error in the CUBA UI</a></figcaption>
</figure>

or directly in the [views.xml](https://github.com/mariodavid/rtcab-cuba-example-views/blob/2-using-view-to-display-referenced-data/modules/global/src/com/rtcab/cev/views.xml#L5) of the application:

{% highlight xml %}
<view class="com.rtcab.cev.entity.Customer"
      extends="_local"
      name="customer-view">
    <property name="type"
              view="_minimal"/>
</view>
{% endhighlight %}

After that, you can change the view reference in the [browse screen](https://github.com/mariodavid/rtcab-cuba-example-views/blob/2-using-view-to-display-referenced-data/modules/web/src/com/rtcab/cev/web/customer/customer-browse.xml#L11) like this:

{% highlight xml %}
<groupDatasource id="customersDs"
   class="com.rtcab.cev.entity.Customer"
   view="customer-view">
    <query>
        <![CDATA[select e from cev$Customer e]]>
    </query>
</groupDatasource>
{% endhighlight %}

This resolved the issue and referenced data is loaded in the browse screen as well.

## The _minimal views and the instance name

The next thing that is relevant when talking about views is the <code>_minimal</code> view. For the local view there is a straight forward definition: all attributes of an entity that are direct attributes of the table.

For the minimal view it is not so obvious, but actually straight forward as well.

In CUBA there is a term called "instance name". The instance name is pretty much the same as the [toString](https://stackoverflow.com/questions/3615721/how-to-use-the-tostring-method-in-java) method in a plain old Java program. It is the representation of an entity on the UI and for referencing it. The instance name is defined through the usage of the [NamePattern Annotation](https://github.com/cuba-platform/cuba/blob/d77b25a67dbd2e25fbaebdda0157beef6f091c70/modules/global/src/com/haulmont/chile/core/annotations/NamePattern.java).

It is used like this: <code>@NamePattern("%s (%s)|name,code")</code>. This will lead to two distinct things:

### The instance name defines the UI representation

First and foremost it will determine what things in which order will be displayed if the entity is referenced by another entity (e.g. the CustomerType by the Customer) and displayed on the UI.

In this case, the Customer Type should be represented as the name of the CustomerType instance followed by the code in brackets. If no instance name is shown it will display the entity class name as well as the ID, which is mostly not what anyone wants to look at in the UI. See the before / after image for an example:

<figure class="center">
	<a href="{{ site.url }}/images/views-the-uncharted-mystery/reference-without-defining-instance-name.png"><img src="{{ site.url }}/images/views-the-uncharted-mystery/reference-without-defining-instance-name.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/views-the-uncharted-mystery/reference-without-defining-instance-name.png" title="Reference without defining an instance name">Reference without defining an instance name</a></figcaption>
</figure>


<figure class="center">
	<a href="{{ site.url }}/images/views-the-uncharted-mystery/reference-with-defining-instance-name.png"><img src="{{ site.url }}/images/views-the-uncharted-mystery/reference-with-defining-instance-name.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/views-the-uncharted-mystery/reference-with-defining-instance-name.png" title="Reference with defining an instance name">Reference with defining an instance name</a></figcaption>
</figure>

### The instance name define the attributes of the minimal view

The second thing that gets defined through the Annotation is that every attribute that is mentioned after the Pipe in the annotation value *is* the minimal view. This seems somewhat obvious because somehow the data has to be displayed in the UI and therefore has to be loaded through the database. But at least for me, i oftentimes don't really thing about that fact.

Another thing that is very important is, that the "minimal" view, compared to "local", can contain references to other entities. In the example from above i defined the instance name of the Customer entity by using one local attribute of the Customer (name) and one attribute that is a reference (type): <code>@NamePattern("%s - %s|name,type")</code>
<figure class="center">
	<img style="width: 400px;" src="{{ site.url }}/images/views-the-uncharted-mystery/recursive-usage-of-minimal-view.png" alt="">
	<figcaption>The minimal view can be used recursively (Customer [Instance Name] --> CustomerType [Instance Name])</figcaption>
</figure>


## Summary

To summarize this topic, let's recap what is important. What get's loaded from the database is very explicit in CUBA. It uses view that define what gets loaded in an eager fashion compared to lazy loading what a lot of other frameworks do.

Views are a little bit cumbersome, but they mostly turn out well in the long run.

I hope, i could clarify your thoughts on what views actually are. There are are some advances usages as well as gotchas and pitfalls with views in general and the minimal view in particular, that i will shift to the next blog post.




<style type="text/css">
article.hentry {
  background-color:#314016;
}

div.hobbit-scene {
  background-color:#314016;
  border: 0px;
  margin-left: -100px;
  margin-right: -100px;
  padding: 40px;
  color: white;


}

div.hobbit-scene figcaption {
  color: white !important;
}
</style>
