---
layout: post
title: CUBA's REST API - If only Roy would knew about it
description: "CUBA Platform has a build in REST API - I'll give a quick overview of what this generic API looks like, how to interact with it and what the limitations of it are. After that i'll show another way of creating a RESTful HTTP API with CUBA."
modified: 2015-11-18
tags: [cuba, rest]
image:
  feature: 2015-11-26-cuba-rest-api/feature.jpg
  feature_source: ...
---

Going further on the feature overview of CUBA Platforms capabilities, i'll will go through the next topic, which is quite important these days: APIs. CUBA has a build-in feature, called REST API. We'll have a look at what this means and how to interact with it. After this overview, we'll have a look about what alternatives we have, if this generic API is not suitable for our needs.

<!-- more -->


## Why do we need an API at all?

In the last few years the need for an API increased dramatically. I think this is due to different attempts to create modularised application architectures. We had different possibilities to achieve this. In the monolithic world we used what the programmic language gave us, like Java Interfaces, Ruby Gems, Approaches like OSGI and so on. 

With the emerge of the idea of services and communication on a service layer that the whole SOA movement brought us, there were also different possibilities to achieve this on an IPC basis. SOAP as the communication and application protocol came up after a few other attemps. A few years later, it turns out that this was not the best idea.

In parallel the Web came up (or was already there). With it there came an architectual style called *Representational State Transfer* - REST (created by [Roy Fielding](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm), just in case you wonder about the headline), that describes a certain kind of software architectures, that share and fulfill a set of [constraints](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm), which i'll not go through right now. After battleing a little with SOAP (or should i say, a lot?) (which is nonsense by the way, because how could anyone compare a communication protocol with an software architecture style?). Nevertheless, REST climbed up the letter of success that nowadays (2015), it's in almost every project the default choiche when it comes to the question about *how to interact with the software programatically*.

## Evaluation of CUBAs REST API implementation from the architectual style angle

So, that was a brief history. Just to give you a little context. With this is mind, we can go back to the concrete build-in REST API from CUBA and have a look at it from architectual style angle.

To get a feeling, if a software architecture fulfills the constraints that the architecture style *REST* defines, we could go through the different constraints, or use the REST majority model, which Leonard Richardson [introduced](http://www.crummy.com/writing/speaking/2008-QCon/act3.html) a few years ago. This model categorized software architecture implementations into different levels. These levels differentiate in terms of how much they fulfill the ideas behind REST and especially RESTful HTTP. You'll find a good [description](http://martinfowler.com/articles/richardsonMaturityModel.html) about the model at Martin Fowlers [blog](http://martinfowler.com/articles/richardsonMaturityModel.html).

The most immature level is something like: You're just using HTTP as a communication protocol to fulfil the [port 80 constraint](http://tools.ietf.org/id/draft-blanchet-iab-internetoverport443-00.html) of the internet. Nothing more. The most mature level means, you're probably in REST heaven, because you use stuff like Hypermedia, Resources and HTTP Verbs and so on.

So, lets start with looking at CUBAs [API implementation](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#rest_api). Assuming we have our previously defined domain model with customers, oders and products, the first thing that we would want to acomplish through our API is to get the list of orders. In CUBA we could do that with something like:

{% highlight bash %}

curl -X GET -H 'Accept: application/json' \
     'http://localhost:8080/app-portal/api/query.json?e=om$Order&q=select+o+from+om$Order+o&s=...'

{% endhighlight %}

To create an order via the API do a call like this:

{% highlight bash %}

 curl -H "Content-Type: application/json" -X POST \
      -d '{"commitInstances": [ { "id":"NEW-om$Customer", "name": "David", "firstname": "Mario", "street": "Example Street 1", "postcode": "12345", "city": "Example City" } ]}' \
      'http://localhost:8080/app-portal/api/commit?s=...'

{% endhighlight %}

Deletes and Updates are similar to the above with different content in the json document.

Level 1 says: Use *Resources*. When you look at the URLs: */api/query/*, */api/commit* you'll see, that these are not the resources of the business domain, but rather RPC calls if you will (like in Level 0).

Level 2 says: Use HTTP Verbs. Well, ok you have to use two different HTTP Verbs (GET and POST), but they are not really used with the characteristics they have in mind like *[Idempotence](http://restcookbook.com/HTTP%20Methods/idempotency/)* and *[Safe](http://restcookbook.com/HTTP%20Methods/idempotency/)*. They are more a tunneling mechanism for HTTP. Additionally other Verbs are not supported.

Level 3 says: Use Hypermedia, which obviously is requires Level 1 to be covered - which is not the case, so its not possible.


### Fielding would probably say: NO

A valid objection is, that RESTful HTTP is **not** CRUD over HTTP via the Verbs. And the resources normally should not match 1:1 from entites to HTTP resources, which is kind of the case here.

When we look at the compliance of Level 3 constraint (Hypermedia), Roy has a pretty decided opinion on that: [REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven).

So lets face it: from the angle of *REST* as an *architectural style*, this API Implementation does not at all fulfil the different requirements to get the *Glory of REST* as Leonard Richardson calls it.


<img style="float:right" src="{{site.url}}/images/2015-11-30-cubas-rest-api/hat-theoretical.png">


## The story about the two hats


At this point, we could leave this topic. But the above description is made just my theoretical dogmatic hat on my head. It's ok, i like this hat. I wear it quite often. It fits quite well and looks gorgeous. But the problems with this hat are versatile. First of all it is expensive. If you want to wear it, you have to pay a pretty high price. Additionally it is not a headgear for every day. If you wear it on a every other normal business day in the office, probably collegues look at you a little mazed.


<img style="float:left; width:64px;" src="{{site.url}}/images/2015-11-30-cubas-rest-api/hat-practical-gekippt.png">
<p style="padding-top:25px;">
Because of this, i'll put this hat beside for now and put on the other one i own: <br/>the practical pragmatic one.
</p>

When i step back a little and rethink about the above written evaluation, a few things come to my mind. 

First of all, i ask myself, even if this assesment is true (which i think it is - otherwise i wouldn't have written it), the practical question is, how relevant this really is. When you follow the constraints or rules of REST just for the sake of it, nothing will happen. Probably you'll not get a congrats mail from Roy the next day. As well, at your next software conference visit, there is no special closed area with a security guy that only let's you in to stay with the cool RESTafarian kids. 

## Do you really need the benefits of a REST compliant architecture?

So there must be some kind of practical value to follow the rules. When we look at the key benefits that derive from fulilling the REST constraints a few that come up immediately to my mind would be:

* Be able to scale (Internet-Scale) (due to Statelessnes and Caching)
* Drive the app state through API (when doing [HATEOAS completly right](https://www.jeffknupp.com/blog/2014/06/03/why-i-hate-hateoas/))
* Be able to identify things through Resources
* Fairly understandable Interface mechanism (through Uniform Interface)
* ...

These are all valid points. Don't get me wrong here, i really like the REST architectural sytle. But i think it makes more sence to follow this constraints to get these benefits in some environments that it might make in others.

When we look at the first benefit: scalability, i would argue if you want to build the next facebook, this is a point that your business should be really concerned about. But when we are in the environment of business software, just a certain amout of scalability might be enough.

Driving the app state though the API responses, means that the server sends back the data as well as state machine transitions, that are valid at this point. For example, if you ask the server for the information about the *Customer 123*, you'll get back the data of this customer:

{% highlight javascript %}

{
  id: 123,
  name: "David",
  firstName: "Mario",
  enable_state: "disabled",
  _links: {
    _self: "/customers/123",
    orders: "/customers/123/orders"
    change_enable_state: "/customers/123/enable_state"
  }
}

{% endhighlight %}

As well, you'll some links back, that guide the client app to the next possible steps. In this example, the customer has the status disabled, that can be changed, because there is link: *_links.change_enable_state*. Due to this the client app can show a button to enable the customer. If the link would not be avaliable, the button would disappear.

This enables the client app, to be quite flexible, because the server can control the flow of the end user experience. I totally understand and see the beauty of it from a computer science perspective. But from a economics perspective, how much maintanance costs do you really need to have at the client app, to do this kind of things? I really don't see the business value of it in most cases.

If you want to know more about the different arguments, here is a [SO question](http://stackoverflow.com/questions/2191049/what-is-the-advantage-of-using-rest-instead-of-non-rest-http) which extends this discussion about RESTful HTTP vs. Non-RESTful HTTP a little more.

I'll leave this sub topic for now with the summary, that the key benefits from following the REST constraints are not important for a large number of enterprise apps. Due to this, it's totally ok, that CUBA's REST API does not support stuff like Hypermedia, customized resource representations (not the general XML / JSON), HTTP Verbs and so on.

## What CUBA's REST API Feature brings - and what it costs

Lets recap, what we get, when we use CUBA's REST API: **CRUD via a HTTP API with exactly zero effort**. This is pure efficiency. This is true for 10 entities as well as for 250 entities. Creating a RESTful HTTP API for 250 entities with Spring MVC for example on the other hand requires implementing a **lot** of boilerplate code.

If you want to achieve the *Glory of REST* in any kind, you basically don't need to use the REST Feature. Instead you should use the [Portal Component](http://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#portal), which is basically the place in a CUBA app, where you put your custom code, that interacts with HTTP directly. Just create a Spring MVC Controller and do what ever needs to be done to create a hypermedia HTTP Verbs based API. 

This is really a point that i want to emphasized. *A good framework has seams*. When using a framework or platform it comes with a litte or a lot of abstraction layers. What happens if the abstraction layer that is provided is not suficient in any case? Are you able to go down the abstraction layer and work on a lower layer (through a *seam*)? This is really critical, because otherwise you are stuck and the productivity improvements that you got through the higher abstraction before does not help you at all in this case.

CUBA enables the developer to either go with the generic REST API, or dig down through the *Portal Component Seam* into the technical details of writing a custom Spring MVC Controller that interacts with request and response objects directly.

A pretty good combination seems be to create a public facing API for a iOS app for example with a custom defined API trough the Portal. In this API you can do exactly what is needed for this API. Probably there is only a subset of entities that need to be exposed through the API. Additionally there are not all CRUD operations required for a public API. For other use cases, like internal requests, integration scenarios with other software within the organisation and so on just use the generic REST API Feature.

Another approach would be to create a little microservice that only serves the API. It is just a proxy service, that will delegate to the actual CUBA REST API. Within this service, you can easily define your own API and translate between the two APIS. That enables a little more freedom in terms of scaling, distribution and so on.

With this approach, you will get the benefits of both sides: no effort for the of-the-shelf use cases and full control over the outcome where its appropriate.


