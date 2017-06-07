---
layout: post
title: From grails to CUBA
description: "In this blog post we will take a comparison between Grails and CUBA"
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---


<!-- more -->

Since i started a transition from Grails to CUBA some time ago, i thought it might be a good idea to give you a technical comparison between those two frameworks and why I finally decided to give CUBA a shot in more and more of my projects.

If we would have a gartner magic quadrant for web frameworks, technically CUBA and grails would be definitively nearby (i will not tell you in which quadrant though ;)), so it is fairly easy to compare those. Both are meta-frameworks in the JVM world. Since both are full stack frameworks we can go from layer to layer to see the differences and similarities.


## Entities and OR Mapping

Starting with the mapping between the database and the object model.


### Grails and GORM

Grails has a built in OR-Mapper called [GORM](http://gorm.grails.org/) which has been extracted from the core in recent versions. When we look at SQL based databases, GORM uses Hibernate under the covers, which is a very powerful OR-Mapper. It basically adds a lot of syntactic sugar on top of it.

GORM is definitively a place where a lot of the innovation of the framework happens. The reason is not so much the mapping to relational databases but to non-relational ones like Neo4j or MongoDB. Also GORM shifts slightly to the non-blocking IO, although this is hard at least in the JDBC world, since JDBC by nature is blocking. But this plays nicely together with the move to NoSQL databases since the driver support for asynchronous operations is much better over there (see the [RxGORM docs](http://gorm.grails.org/latest/rx/manual/index.html) for more info).

Generally you will create a domain type by executing the command line task <code>grails create-domain-class Person</code>. After that you add some properties to the person like this:

{% highlight groovy %}
class Person {
    String name
    Integer age
    Date lastVisit
}
{% endhighlight %}

With this, you are basically done. You can setup associations between entites through some keywords like this:

{% highlight groovy %}
class Person {
    static hasMany = [pets: Pet]
}

class Pet {
    static belongsTo = [person: Person]
    String name
}
{% endhighlight %}

For more information take a look at the [docs on accosiations](http://gorm.grails.org/latest/hibernate/manual/index.html#domainClasses).

### CUBA loves JPA
CUBA took a slightly different route in this regard. CUBA basically does not do anything aroung the topic of the OR-Mapper. They don't have a explicit part in the framework that deals with database access. Instead CUBA uses the Java persistence API in order to fulfill its job. Entities are just POJOs with <code>@Entity</code> annotations. An equivalent to the above Person example would look like this:


{% highlight groovy %}
@Entity
class Person {
    static hasMany = [pets: Pet]
    protected Set<Pet> pets;

    public Set<Pet> getPets();

}

@Entity
class Pet {
    static belongsTo = [person: Person]
    String name
}
{% endhighlight %}

### execute queries


### Database migrations

[Grails database migration](https://github.com/grails-plugins/grails-database-migration)

## business logic & constraints

## security

## view technology

## testing

## plugin architecture

## platform features and eco system
