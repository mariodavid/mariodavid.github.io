---
layout: post
title: My personal CRUD story – or how i came to CUBA platform
description: "In this blog post i’d like to lay out how i came to the CUBA platform and what the benefits of this tool are. I’ll dig a little into the different phases on my young “business application development” history, just to give you a little context."
modified: 2015-11-02
tags: [cuba, crud]
---

In this blog post i’d like to lay out how i came to the CUBA platform and what the benefits of this tool are. I’ll dig a little into the different phases on my young “business application development” history, just to give you a little context. So let’s start with how i came to the typical CRUD applications to help non-tech people to be a little more productive. 

<!-- more -->

## A brief histroy on my CRUD background

Over the years i stumbled upon the same kind of problems in totally different areas of companies. From small shops to fairly large enterprises, often people have some basic business process, that should be fulfilled. Often these business processes go hand in hand with entering data.

I first came across these kind of requirements in the world of renting goods. A company i worked in, started to rent little diggers to private gardeners for a weekend. Obviously they wanted to schedule the lettings of the few diggers they had.

So basically this was the domain:

<figure class="center">
	<a href="{{ site.url }}/images/my-personal-crud-story/domain-example-customer-orders-uml1.png"><img src="{{ site.url }}/images/my-personal-crud-story/domain-example-customer-orders-uml1.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/my-personal-crud-story/domain-example-customer-orders-uml1.png" title="A domain example with a customer – orders relationship">A domain example with a customer – orders relationship</a></figcaption>
</figure>


A typical example that you’ll find in every second software engineering book.

We started out with something like a Access DB to achieve our goal. As the business grew, the requirements on the software grew as well.

Going from a list of orders that showed the user what products have been rented on a given time-frame (in a very bad way i should point out), we wanted to achieve some kind of more advanced scheduling view. Because of more than one location of the users, a web based software seems to make a little more sense. So i started my journey to PHP, a quite popular choice back then.

But what really got me wondering, was the situation, that i had to do the same kind of stuff over and over again. Because besides the actual important views (like the scheduling view mentioned above), there were a ton of views and CRUD stuff, that had to be implemented as well. Normally nobody really cares about these screens, because it does not support the main business process. Nevertheless, Products, Renting Locations, Users, Roles and whathaveyou, want to be created and managed by the user as well.

While staying in the PHP world (this was pretty much before this different Frameworks came up, or at least “up onto my radar”) – so i really got to work with the “bare metal” stuff like HTML, PHP, [PEAR](https://pear.php.net/) and different crazy stuff like [script.aculo.us](https://script.aculo.us/).

## Scaffolding to the rescue

Fast Forward – a few years and CRUD projects later. Object orientation is all its facets came across my mind due to studies of computer science mostly in form of Java. It seems i was ready for the next big step. As most of us know to do web-based software in Java is more a pain than a joy. So i looked around to get rid of the old PHP stuff i made the past few years.

During that time the new kid on the blog was Rails. The concepts for web applications that were introduced like [CoC](https://en.wikipedia.org/wiki/Convention_over_configuration), [database migrations](http://edgeguides.rubyonrails.org/active_record_migrations.html) etc. were pretty amazing for me. But one thing was an eye opener: [scaffolding](http://guides.rubyonrails.org/command_line.html#rails-generate). To go from a model, that defines the business object, to generate full stack from DB definitions to HTML Forms as well as an “RESTful API” was pretty need. It shifted my mind in terms of “what i have to create programmatically and what should the software / framework / generator do for me”.


The general consensus in the software industry was and still is, that these scaffolding mechanisms have to be treated with caution. The reason for this scepsis are versatile. One thing is, that “it does not work for real applications”. Another often heard of is “the resulting UI doesn’t fit our requirements”.


<figure class="center">
	<a href="{{ site.url }}/images/my-personal-crud-story/grails-scaffolding-author-books.png"><img src="{{ site.url }}/images/my-personal-crud-story/grails-scaffolding-author-books.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/my-personal-crud-story/grails-scaffolding-author-books.png" title="A scaffolding example with authors and books generated by Grails">A scaffolding example with authors and books generated by Grails</a>.</figcaption>
</figure>

When thinking about it, this is absolutely correct. Because with Frameworks like (G)Rails you have a general purpose tool at hand. Which means, that it is not directly tied to certain kind of applications. You can create a online shop with a highly optimized ui just like you cankinds-of-application-realizeable
create a RESTful HTTP backend for a Javascript based fat client. You can do a CRUD like business applications just like a responsive website for your companies marketing campaign.


<figure class="center">
	<img src="{{ site.url }}/images/my-personal-crud-story/kinds-of-application-realizeable.png" />
	<figcaption>space of realizable applications in the web application world</figcaption>
</figure>


Due to this, scaffolding can not solve all these problems, because the range is too broad. This is why the results of these scaffolding attempts end up with a CRUD interface for domain types that have certain limitations from a UI point of view as well from its functionality. Which is total ok, because the focus of these frameworks has never been on the scaffolding side of things.


## “domain specific” depends on the perspective

Although i said, Rails alike are *general purpose*, Martin Fowler [tells us](http://martinfowler.com/bliki/InternalDslStyle.html) it is not, but rather *domain specific*. Who am i, to dissent the guy, who was part of the initial [agile manifesto](http://www.agilemanifesto.org/), author of [Refactoring](http://martinfowler.com/books/refactoring.html) and so much other good stuff, that pushed the software industry further?

The problem is, that *domain specific* is a pretty generic term. From the view of Java Servlets Rails is absolutely domain specific, where the domain is “web-app with a relational database backend”. The same is true for the more general Spring MVC vs. the full stack Framework Grails which is based on Spring (MVC). On the other hand Java Servlets are much more specific than using sockets, because you’re constraint to HTTP.



<figure class="center">
	<a href="{{ site.url }}/images/my-personal-crud-story/from-general-purpose-to-domain-specific.png"><img src="{{ site.url }}/images/my-personal-crud-story/from-general-purpose-to-domain-specific.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/my-personal-crud-story/from-general-purpose-to-domain-specific.png" title="From general purpose to domain specific in the web development world">From general purpose to domain specific in the web development world</a></figcaption>
</figure>


While i was creating mostly CRUD like business applications in these frameworks in the second layer of the diagram, i thought that what if there is a category of frameworks / platforms, that fulfills the requirements for business applications a little better. When we would narrow down the focus a little further towards this subset of applications, perhaps scaffolding can create real usable stuff.

A few months ago, i came across the [CUBA platform](https://www.cuba-platform.com/) via an [article](http://www.javacodegeeks.com/2015/06/cuba-platform-the-new-java-enterprise-applications-framework.html) from java code geeks. CUBA is primarily a commercial “framework”, that advertises itself with the slogan: “A high level Java framework for faster enterprise software development”.

At a first glance it looked like just another web framework with its common parts. OR-Mapper, Dependency Injection, Scaffolding, UI and so on. Instead of creating all the subparts by itself, it uses a kind of *Meta-framework* approach just like Grails does. The OR-Mapper is JPA (EclipseLink), Spring for DI, Vaadin as a UI Framework.

## A new category in general purpose - domain specific dimension?

After digging into it a little further, i realized, that this thing seems so be some kind of different than what i had seen before in the web development space.

The difference is, that the authors of CUBA were going the “domain specific” road a little further and did exactly the focus narrowing on “enterprise software development” as i thought of in the above image. With this focus is mind, the authors were able to be much more opinionated. Opinionated frameworks normally increase productivity if you follow their way of life (just like “[The Rails way](http://www.amazon.com/The-Rails-Way-Obie-Fernandez/dp/0321445619)” in the Rails world).

The same seems to be true for the CUBA platform. Following their opinions for stuff like filtering, security, reporting etc. creates a immense productivity boost i had not seen since i first used Rails in favor of PHP / Java.

The phrase “rapid application development” really gets another meaning here. This is due to multiple reasons.


## What makes this *thing* so special?

Often a domain model for a business application has a lot of entities as well as connections between them. This leads to a UI, where you have to create a lot of related entities in your actual workflow.


First of all, they get the basic stuff required for a business application completely right. CUBA creates a scaffolded UI that totally fulfill these needs. It is based on the different choices they made for the different styles of associations that two entities can have. A *Many to One* or *One to One* associations are represented via a PickerField, a *Many to Many* relationship creates a add / remove table for the association whereas a *One to Many* association will create a table within the detail view of the Entity that holds the *One* side.

With these simple but incredible powerful tools, you can create a pretty complex domain model with a similar complex UI within minutes, that makes creating a graph of related entities via the UI a joy.

The second part is the generic filter solution. It feels a little like the filter possibilities you’ll know from [Excel](https://support.office.com/en-us/article/Filter-data-in-an-Excel-table-7d8e9739-2898-4bfe-9d0f-c6204e6e5c8a) with the additional feature, that you can filter over associations of the entity that the table is based on. This is pretty amazing and removes the need for a lot of custom filter programming, that would have been created otherwise manually by the developer.

Next, there is a full blown security subsystem. It is based on an [ACL](https://en.wikipedia.org/wiki/Access_control_list) approach, which lets you as a user creates users, groups, roles that let you slice your application on a view, entity (+ attribute) as well as entity instance level. It is no problem to create (create in this context just means: use the software) an app, that fulfills the following requirements:


1. Managers in NY sees all customers in NY
2. Sales in NY sees customers in NY they created with all details. Customers in SF are shown with just the properties “name” and “city”, where only “name” can be edited.
3. Managers in TX (Headquarter) see all customers from all locations, but cannot edit them


With the comprehensive UI to manage these things, there is very little need to implement a homegrown security solution.

This feature list could go on for a pretty long time. I haven’t covert the reporting part as well as business process management, scheduled Tasks, HTTP-JSON API, Fat Client generation, different administrative possibilities and so on. Probably i’ll take a further look in future blog posts.

## The essence of this finding

When seeing all this different pieces, that differs CUBA from a programming framework are the different “platform” features that were added on top of these technical frameworks. These things let the programmer concentrate on the business problem a little more. This is principally a good thing, although it is a truism.

The problem is, that often this is a hard thing to imagine as a programmer. We, the techies, love *poking around with NTLM authentication*, *fiddling around until we get the perfect user interface for entering our order instances*, or *creating a hypermedia driven RESTful HTTP Interface for our API*. We want to create things, which is totally fine and the very basis of our industrie.

The other way, you can look at it from another angle. As we probably know already from Isacc Newton, [Scott Hanselman](http://www.hanselman.com/blog/WeAreAbstractingOnTheShouldersOfGiants.aspx) or Sheldon Cooper:

> We are all standing on the shoulders of giants

This is especially true in the software development space. From the bottom with stuff like all the electric engineering, the main hardware abstraction: operating system, over network protocols like HTTP, data storage mechanisms like relational databases, up to APIs & Frameworks like Servlet, Rack and Rails. All these things allow us, as business software developers, to basically create these distributed, scalable and easy to use productivity tools.

With that in mind, there is literally no reason to not go up the abstraction letter if possible. Obviously this general advice has to be treated with caution. There are various good reasons to develop software using C as well as Plain Java Servlets or CUBA.

What it pretty much boils down to is this “right tool for the job” kind of thing. When you want to create typical enterprise apps (nail) and there comes this CUBA (hammer) – it’s a very good fit. This implication, on the other hand should let nobody think, that everything is a nail.

I absolutely encourage you to check out CUBA and especially the different online demos you’ll find [here](https://www.cuba-platform.com/online-demo).

{% highlight html %}
[Edit 2015-10-30: added explanation, that CUBA is a commercial product]
{% endhighlight %}
