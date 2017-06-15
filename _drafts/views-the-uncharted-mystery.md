---
layout: post
title: Views - the uncharted mystery
description: "The views approach is "
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: grails-vs-cuba/feature.png
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

### Food on demand - *lazy loading*
You sit in a restaurant and are so hungry you could literally eat a horse. After you've scanned the first pages of the menu you have this feeling that you can't really decide what to eat because everything looks so great. You are so hungry, that you don't really bother about the prices.

After the waiter asks you if you want to have something to drink and a starter (which you gladly accept), you finally decide that you will take the pasta first and if you are still hungry you are going to order another piece of pasta and the fish you have seen in the first page of the menu.

The waiter brings the drink, five minutes later the starter and another ten minutes later the pasta. You know that you are going to be hungry afterwards with this, so you add another pasta and the fish. Additionally you order two sandwiches (although you only really like the bread, not so much the content) and for dessert you'll take the ice cream with chocolate, raspberry and vanilla. You know that you will only eat the raspberry one, but as it could not be ordered on its own, that's the way it is.

The waiter takes a couple of additional round trips to bring you all this stuff. You eat a bit of everything, but it turns out your eyes were bigger than your belly. So you see not eaten food all over the place. You had eaten so much, you aren't really able to move, when the waiter brings you the bill. This is when you realize that letting the belly decide what to order is not the best idea.


### arrange your meal on your own - *eager fetching*

Now imagine the following other situation. Setting is the same, but you are at home and have to decide what to eat. The fridge is empty, so if you decide to eat something, you have to take your bike (since your car is broken), go to the supermarket, buy the food you want to cook, do the cooking and have a nice meal one hour later.

In this scenario, although you are very hungry, you decide to only take pasta and leave the fish because you know it is going to be fast to prepare. On your way out of the supermarket you'll grab a piece of raspberry ice cream. This means you have to drive faster so that it wouldn't melt, but you feel bad anyway, so that's not a big of a deal.
After you are ready eating, you are full up as well. Not so much you can't move anymore but its enough for now.

This was a fairly long story, but it pretty much sums up the situations when it comes to the question when what data will be loaded of your entities.

## N+1 query problem

The N+1 query problem occurs when using lazy loading without activly thinking about it. To illustrate that, let's have a look at a snippet of Grails code. This does not mean that in Grails everything is lazy loaded (its actually up to you to decide). In Grails your database requests will return instances of the Entity, with all attributes of the table loaded with it. It basically makes a "SELECT * FROM Pet". When you want to traverse a relationship between entities you do that afterwards. Here's an example:

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

It is a single line of code that will do the traversal here: <code>it.owner.name</code>. Owner is the relationship that has not been loaded in the first request (<code>Pet.findAll</code>). So for each call of this line, GORM will something like "SELECT * FROM Person WHERE id='...'". This is called *lazy loading*.

As a application developer you probably don't really notice this, because you might think that you will only traverse the object graph.


CUBA instead has the notion of "views". Views are basically a definition of what attributes have to be fetched and will be loaded in the entity instances. This would be a "SELECT name FROM Pet".

The real difference comes into play when it comes to relations between entities.

## Only pay for what you really need


## The _minimal views and the instance name


## create multiple views for different purposes
