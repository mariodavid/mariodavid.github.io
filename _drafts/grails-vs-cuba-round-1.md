---
layout: post
title: Grails vs CUBA
subtitle: Round 1 - Data access and business logic
description: "In this blog post series we'll compare the two JVM based web frameworks: Grails and CUBA. This time, starting with data access"
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: grails-vs-cuba/feature.png
---

Since I started a transition from Grails to CUBA some time ago, I thought it would be a good idea to give you a technical comparison between those two frameworks. In this blog post series I go though different aspects, starting with data access and business logic in this post, to conclude why I finally decided to give CUBA a shot in more and more of my projects.

<!-- more -->


If we would have a gartner magic quadrant for web frameworks, technically CUBA and grails would be definitively nearby (I will not tell you in which quadrant though ;)), so it is fairly easy to compare those. Both are meta-frameworks in the JVM world. Since both are full stack frameworks we can go from layer to layer to see the differences and similarities.




## Entities and OR Mapping

Lets get started with the mapping between the database and the object model. This is an important part for a lot of backend developers, so I'll go into a little more detail about the approaches both frameworks take.


### Grails and GORM

Grails has a built in OR-Mapper called [GORM](http://gorm.grails.org/) which has been extracted from the core in recent versions. When we look at SQL based databases, GORM uses Hibernate under the covers, which is a very powerful OR-Mapper. It basically adds a lot of syntactic sugar on top of it.

<img style="float: right; padding-left: 10px; margin-right:-80px; width:400px" src="{{site.url}}/images/grails-vs-cuba/grails.jpg">

GORM is a place where a lot of the innovation of the framework happens. The reason is not so much the mapping to relational databases but to non-relational ones like Neo4j or MongoDB. It supports a variety of NoSQL databases to map it to the domain types in a form that makes sense depending on the database you are working with.

Also GORM shifts slightly to non-blocking IO when it comes to the actual access of the database. Turns out to be quite hard for the relational databases, since JDBC is blocking by nature. But NoSQL databases have not the restriction of the JDBC standard and therefore the driver support for asynchronous operations is much better over there (see the [RxGORM docs](http://gorm.grails.org/latest/rx/manual/index.html) for more info).

Generally you will create a domain type by executing the command line task <code>grails create-domain-class Person</code> or using the IDE to accomplish the same thing. After that you add some properties to the person class. You'll end up with something like this:

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


<img style="float: left; padding-right: 10px; margin-left:-80px; width:400px" src="{{site.url}}/images/grails-vs-cuba/cuba.jpg">

CUBA took a slightly different route in this regard. CUBA basically does not do much around the topic of the OR-Mapper. It doesn't have an explicit part in the framework that deals with database access. Instead CUBA uses the Java persistence API in order to fulfill its job. Entities are just POJOs with <code>@Entity</code> annotations.

An equivalent to the above Person example would look like this:


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

First of all, they differ in the way the data retrieval gets called. Grails uses a pattern called [Active record](https://www.martinfowler.com/eaaCatalog/activeRecord.html) in order to fetch data from the database. Active record means that the entity class itself is responsible for fetching the data. <code>Person.list()</code> will in this case get all the entities of the Person table.

In JPA there is the <code>EntityManager</code> that has to be used for data retrieval, while the Entity object are more data access objects (DAO) that act more in a passive manner. You can see it more or less as a [repository pattern](https://martinfowler.com/eaaCatalog/repository.html). CUBA additionally has some more options to access data. Normally instead of using the <code>EntityManager</code> CUBA has a thin layer on top of that called <code>DataManager</code>. The DataManager cares about all the stuff that JPA is not really aware of but is part of CUBA like row level security or views (see this [diff](https://doc.cuba-platform.com/manual-6.5/dm_vs_em.html) for details).


<img src="{{site.url}}/images/grails-vs-cuba/fight.jpg">

Both patterns have some advantages and disadvantages. Active model sometimes leads to a slightly messier code when you have a big application because the entities have more responsibilities, but in smaller applications it is easier to read.

<div class="information">In the upcoming Grails (GORM 6.1) version, there is also native support for <a href="http://gorm.grails.org/6.1.x/hibernate/manual/index.html#dataServices">DataServices</a> (kind of repository like) which will give you the option to chose between those to approaches.</div>

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

Grails has different options to access data. Let's look at the outstanding ones. As the example we want to fetch all pets that have a certain name. You can use dynamic finders in Grails like this: <code>Pet.findAllByName("Long John")</code>. Pretty easy, right? Actually the method findAllByName is created dynamically and interpreted by the execution engine do create the correct SQL statement. This means that it has never been defined by the application developer. Instead Grails understands all combinations of those method names. <code>Pet.findAllByNameLikeAndFirstnameLikeAndAgeGreaterThan("Long", "John", 21)</code> would also work (but the readability will go down fast for complicated queries).

As these methods are only suitable for simple queries, there is the opportunity to create a so called 'where query' that does the same thing:

{% highlight groovy %}
Pet.findAll(sort:"name") {
     name == 'Long John'
}
{% endhighlight %}

Where queries have compile time checks, which is a huge win, compared to what we have seen in CUBA with plain JPQL.

### Lazy loading vs. eager fetching

One additional thing that is very important and is something where both frameworks have a different approach is the question what data gets loaded and when this is done.

#### Grails uses lazy loading by default

In Grails your database requests will return instances of the Entity, with all attributes of the table loaded with it. It basically makes a "SELECT * FROM Pet". When you want to traverse a relationship between entities you do that afterwards. Here's an example:

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

To write and interact with lazy loaded data is so easy, because you just forget about it. It is so transparent, because from an app developer point of view it exactly looks like traversing an object graph.

<img src="{{site.url}}/images/grails-vs-cuba/fight2.jpg">


Although lazy loading is default in Grails, it is not your only choice. You can define eager fetching globally on the entity attribute level or on a case to case basis when you actually trigger the loading.

But for me, since the lazy loading is default and it is so convinient, I often felt myself in the trap of not thinking about it at all and only remember it, when the application acts really slow (which probably tells you more about myself as a programmer instead of grails as a framework ;)).

#### CUBA views make loading explicit

CUBA instead has the notion of "views". Views are basically a definition of what attributes have to be fetched and will be loaded in the entity instances. Views are defined in the corresponding views.xml file where you give a view a name and define what attributes will be loaded with this view. Here's an example of a CUBA view for the Role entity:

{% highlight xml %}
<view class="com.haulmont.cuba.security.entity.Role" name="role.copy">
    <property name="name"/>
    <property name="type"/>
    <property name="locName"/>
    <property name="permissions" view="role.edit"/>
    <property name="description"/>
</view>
{% endhighlight %}

In this example we take some direct attributes of the Role (name, type etc.) as well as a 1:N composition (permission).

I found using views is a very explicit way of dealing with the situation. It forces the developer to think about this topic. This can sometimes be cumbersome but it leads to a better place I think compared to make this whole loading story implicit.

If you are interested in more detail on this topic, I created an article around CUBA views. You can find it here: [Views - the uncharted mystery](https://www.road-to-cuba-and-beyond.com/views-the-uncharted-mystery/).


### Database migrations

Database migrations are oftentimes part of modern full stack frameworks. Ruby on Rails pionered with this idea so automatically generate deltas between two database schema versions.

Grails and CUBA both have features in place that make that happen as well. For Grails there is a commonly used plugin called [Grails database migration](https://github.com/grails-plugins/grails-database-migration) which uses a java library called [Liquibase](http://www.liquibase.org/) under the covers. CUBA instead comes out of the box with SQL scripts that gets generated via their RAP tool CUBA studio. Both approaches recognies changes in the entities and try to do their best in order to generate a SQL schema delta (like <code>ALTER TABLE...</code>) to reflect the changes in the entity model.


<img style="float: right; padding-left: 10px; margin-right:-80px; width:400px" src="{{site.url}}/images/grails-vs-cuba/fans.jpg">

Besides this build in approaches, both frameworks can use any of the mature java libraries for db migrations like [Flyway](https://flywaydb.org/) or the above mentioned [Liquibase](http://www.liquibase.org/).

With this we've basically covered all stuff related to data access. Next, we will look at the next layer which is where and how to put the business logic.

## business logic & constraints

When it comes to business logic, both frameworks are actually pretty close to each other. This has mainly to do with the fact, that Grails as well as CUBA use the [Spring framework](https://projects.spring.io/spring-framework/) under the covers. This means that in both frameworks make fairly heavy use of the [dependency injection pattern](https://martinfowler.com/articles/injection.html).

In Grails you usually create [services](https://docs.grails.org/latest/guide/services.html) via the command line. These classes are spring beans that repesent some kind of business logic and have certain characteristics. Normally the UI layer either directly interacts with the entity instances to talk to the database (in easier cases), or use services to execute some kind of business logic, that itself might call the database.

In CUBA there is as well the notion of [services](https://doc.cuba-platform.com/manual-6.5/services.html) (in the same sense). You most likely create a service via CUBA studio, which will generate a Service interface as well as a Class that implements this interface and is annotated with the Spring <code>@Component</code> annotation.

One thing to notice is that in CUBA, since it uses an four-tier application architecture, there is an optional separation between the client tier (the web server) and the middle tier (the backend application server) (see the [docs on app-tiers](https://doc.cuba-platform.com/manual-6.5/app_tiers.html) for more information). This means that you as a developer have to think a little bit more where to put your spring beans, because this has implications where you can call them.

In both frameworks you can additionally create any kind of spring bean and use it throughout the application.

### Defining constraints on entities

The last part in this blog post is about validation. Normally every application has some kind of validation logic (mostly on entities), that have to be fulfiled in order to e.g. save them in the database. This is why both frameworks have solutions for the problem built-in.

Let's look at an example for what a validation in this context means. Taking the Person - Pet example from above, let's say that the name of the person has to be there in order to create a person instance. Another one would be that at least every Person has to have one Pet.

In Grails we can define the [constraints](https://docs.grails.org/latest/ref/Constraints/Usage.html) in the entity class in a special closure called 'constrains' like this:

{% highlight groovy %}
class Person {
    String name
    static hasMany = [pets: Pet]

    static constraints = {
            name blank: false, unique: true
            pets minSize: 1, blank: false
        }
}
{% endhighlight %}

While in CUBA there is an explicit usage of the [Bean validation](http://beanvalidation.org/) annotations:


{% highlight groovy %}
@Entity(name = "personpet$Person")
public class Person extends StandardEntity {

    @NotNull
    @Column(name = "NAME")
    protected String name;

    @Size(min = 1)
    @Composition
    @OneToMany(mappedBy = "person")
    protected List<Pet> pets;

    // getters and setters...
}

{% endhighlight %}

In both framework you have options to create custom validations as classes etc. So basically the difference is just in syntax but not in the semantics.



With this, we are at the end of the first part of the comparison. In the next blog post, we will talk more about UI, security, APIs & platform features. These are actually the areas where the big differences are, so stay tuned.


<style type="text/css">
article.hentry {
  background-color:#ac43ba;
}
</style>
