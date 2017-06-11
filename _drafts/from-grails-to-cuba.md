---
layout: post
title: From Grails to CUBA
subtitle: Round 1 - differences in data access
description: "In this blog post series we'll compare the two JVM based web frameworks: Grails and CUBA. This time, starting with data access"
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: grails-vs-cuba/feature.png
---

Since i started a transition from Grails to CUBA some time ago, i thought it might be a good idea to give you a technical comparison between those two frameworks and why I finally decided to give CUBA a shot in more and more of my projects.

<!-- more -->

If we would have a gartner magic quadrant for web frameworks, technically CUBA and grails would be definitively nearby (i will not tell you in which quadrant though ;)), so it is fairly easy to compare those. Both are meta-frameworks in the JVM world. Since both are full stack frameworks we can go from layer to layer to see the differences and similarities.



<!--
<img style="margin-left:-80px; width:1200px" src="{{site.url}}/images/grails-vs-cuba/example.png">
-->

## Entities and OR Mapping

Starting with the mapping between the database and the object model.


### Grails and GORM

Grails has a built in OR-Mapper called [GORM](http://gorm.grails.org/) which has been extracted from the core in recent versions. When we look at SQL based databases, GORM uses Hibernate under the covers, which is a very powerful OR-Mapper. It basically adds a lot of syntactic sugar on top of it.




GORM is definitively a place where a lot of the innovation of the framework happens. The reason is not so much the mapping to relational databases but to non-relational ones like Neo4j or MongoDB. Also GORM shifts slightly to the non-blocking IO. This is hard at least in the JDBC based database connections, since JDBC is blocking by nature. But this plays nicely together with the move to NoSQL databases since the driver support for asynchronous operations is much better over there (see the [RxGORM docs](http://gorm.grails.org/latest/rx/manual/index.html) for more info).

Generally you will create a domain type by executing the command line task <code>grails create-domain-class Person</code>. After that you add some properties to the person class. You'll end up with something like this:

{% highlight groovy %}
class Person {
    String name
    static hasMany = [pets: Pet]
}

class Pet {
    static belongsTo = [owner: Person]
    String name
}
{% endhighlight %}

### CUBA loves JPA
CUBA took a slightly different route in this regard. CUBA basically does not do much around the topic of the OR-Mapper. It doesn't have an explicit part in the framework that deals with database access. Instead CUBA uses the Java persistence API in order to fulfill its job. Entities are just POJOs with <code>@Entity</code> annotations. An equivalent to the above Person example would look like this:

{% highlight groovy %}
@Table(name = "PERSONPET_PERSON")
@Entity(name = "personpet$Person")
public class Person extends StandardEntity {
    @Column(name = "NAME")
    protected String name;

    @Composition
    @OneToMany(mappedBy = "person")
    protected List<Pet> pets;

    // getters and setters...
}

@Table(name = "PERSONPET_PET")
@Entity(name = "personpet$Pet")
public class Pet extends StandardEntity {
    @Column(name = "NAME")
    protected String name;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "PERSON_ID")
    protected Person person;

    // getters and setters...
}
{% endhighlight %}

So it is basically the same. CUBA is a little bit more noisy in this regard, but from a features perspective they are pretty much the same.

### Data access patterns

Executing queries against the datastore is a little more different.

First of all, the differ in the way the data retrieval gets called. Grails uses a pattern called [Active record](https://www.martinfowler.com/eaaCatalog/activeRecord.html) in order to fetch data from the database. Active record means that the entity class itself is responsible for fetching the data. <code>Person.list()</code> will in this case get all the entities of the Person table.

In JPA there is the <code>EntityManager</code> that has to be used for data retrieval, while the Entity object are more data access objects (DAO) that act more in a passive manner. You can see it more or less as a [repository pattern](https://martinfowler.com/eaaCatalog/repository.html). CUBA additionally has some more options to access data. Normally instead of using the <code>EntityManager</code> CUBA has a thin layer on top of that called <code>DataManager</code>. The DataManager cares about all the stuff that JPA is not really aware of but is part of CUBA like row level security or views (see this [diff](https://doc.cuba-platform.com/manual-6.5/dm_vs_em.html) for details).

Both patterns have some advantages and disadvantages. Active model sometimes leads to a slightly messier code when you have a big application because the entities have more responsibilities, but in smaller applications it is easier to read.

<div class="information">In the upcoming Grails (GORM 6.1) version, there is also native support for <a href="http://gorm.grails.org/6.1.x/hibernate/manual/index.html#dataServices">DataServices</a> (kind of repository like) which will give you the option to chose between those to worlds</div>

### Making data access calls

In CUBA the main data access call would look like this:

{% highlight groovy %}

@Inject
DataManager dataManager

// ...

LoadContext<Pet> loadContext = LoadContext.create(Pet.class)
            .setQuery(LoadContext.createQuery("select p from demo$Pet p where p.name = :petName")
                .setParameter("petName", "Long John"))
dataManager.loadList(loadContext);

// ...

{% endhighlight %}

You use the <code>dataManager</code> and pass it a <code>loadContext</code>. The load context defines what has to be loaded. In this case we use a JPQL expression to define that we want all Pets with a certain name.

When it comes to the UI, CUBA has an additional concept called "datasource" that will do the heavy lifting to get the data in the UI components.

Grails has different options to access data. Let's look at the outstanding ones. As the example we want to fetch all pets that have a certain name. You can use dynamic finders in Grails like this: <code>Pet.findAllByName("Long John")</code>. Pretty easy, right? Actually the method findAllByName is created dynamically and interpreted by the execution engine do create the correct SQL statement.

As these methods are only suitable for simple queries, there is the opportunity to create a so called 'where query' that does the same thing:

{% highlight groovy %}
Pet.findAll(sort:"name") {
     name == 'Long John'
}
{% endhighlight %}

Where queries have compile time checks, which is a huge win, compared to what we have seen in CUBA with plain JPQL.

### Lazy loading vs. eager fetching

One additional thing that is very important and is something where both frameworks have a different approach is the question what data gets loaded and when this is done.

Imagine the following situation: You sit in a restaurant and are so hungry you could literally eat a horse. After you've scanned the first pages you have this feeling that you can't really decide what to eat because everything looks so great. You are so hungry, that you don't bother really about the prices.

After the waiter asks you if you want to have something to drink and a starter (which you gladly accept), you finally decide that you will take the pasta first and if you are still hungry you are going to order another piece of pasta and the fish you have seen in the first page of the menu.

The waiter brings the drink, five minutes later the starter and another ten minutes later the pasta. You know that you are going to be hungry afterwards with this, so you add another pasta and the fish. Additionally you order two sandwiches (although you only really like the bread, not so much the content) and for dessert you'll take the ice cream with chocolate, raspberry and vanilla. You know that you will only eat the raspberry one, but as it could not be ordered on its own, that's the way it is.

The waiter takes a couple of additional round trips to bring you all this stuff. You eat a bit of everything, but it turns out your eyes were bigger than your belly. So you see not eaten food all over the place. You had eaten so much, you aren't really able to move, when the waiter brings you the bill. This is when you realize that letting the belly decide what to order is not the best idea.

Now imagine the following other situation. Setting is the same, but you are at home and have to decide what to eat. The fridge is empty, so if you decide to eat something, you have to take your bike (since your car is broken), go to the supermarket, buy the food you want to cook, do the cooking and have a nice meal one hour later.

In this scenario, although you are very hungry, you decide to only take pasta and leave the fish because you know it is going to be fast to prepare. On your way out of the supermarket you'll grab a piece of raspberry ice cream. This means you have to drive faster so that it wouldn't melt, but you feel bad anyway, so that's not a big of a deal.
After you are ready eating, you are full up as well. Not so much you can't move anymore but its enough for now.

This was a fairly long story, but it pretty much sums up the situations when it comes to the question when what data will be loaded of your entities.

With Grails the situation is pretty much like you sitting in the restaurant. In Grails your database requests will return instances of the Entity, with all attributes of the table loaded with it. It basically makes a "SELECT * FROM Pet". When you want to traverse a relationship between entities you do that afterwards. Here's an example:

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

> Lazy loading is one of GORMs greatest strength and probably one of its greatest weaknesses

As a application developer you probably don't really notice this, because you might think that you will only traverse the object graph.



CUBA instead has the notion of "views". Views are basically a definition of what attributes have to be fetched and will be loaded in the entity instances. This would be a "SELECT name FROM Pet".

The real difference comes into play when it comes to relations between entities.





### Database migrations

[Grails database migration](https://github.com/grails-plugins/grails-database-migration)

## business logic & constraints

## security

## view technology

## testing

## plugin architecture

## platform features and eco system


<style type="text/css">
article.hentry {
  background-color:#ac43ba;
}
</style>
