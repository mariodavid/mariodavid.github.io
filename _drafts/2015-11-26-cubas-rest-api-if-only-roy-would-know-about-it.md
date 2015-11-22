---
layout: post
title: CUBA REST API - If only Roy would knew about it
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

In the last few years the need for an API increased dramatically. I think this is due to different attempts to create modularised application architectures. We had different possibilities to achieve this. In the monolithic world we used what the programmic language gave us, like Java Interfaces, Java Package Structures, Approaches like OSGI and so on. 

With the emerge of the idea of services and communication on a service layer that the whole SOA movement brought us, there were also different possibilities to achieve this on an IPC basis. SOAP as the communication and application protocol came up after a few other attemps. A few years later, it turns out that this was not the best idea.

In parallel the Web came up (or was already there). With it there came an architectual style called *Representational State Transfer* - REST (created by [Roy Fielding](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm), just in case you wonder about the headline), that describes a certain kind of software architectures, that share and fulfill a set of [constraints](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm), which i'll not go through right now. After battleing a little (or should i say, a lot?) (which is nonsense by the way, because how could anyone compare a communication protocol with an software architecture style?). Nevertheless, REST climbed up the letter of success that nowadays (2015), it's in almost every project the default choiche when it comes to the question about *how to interact with the software programatically*.

## Evaluation of CUBAs implementation from the architectual style angle

So, that was a brief history. Just to give you a little context. With this is mind, we can go back to the concrete build-in REST API from CUBA and have a look at it from architectual style angle.

To get a feeling, if a software architecture fulfills the constraints that the architecture style *REST* defines, we could go through the different constraints, or use the REST majority model, which Leonard Richardson [introduced](http://www.crummy.com/writing/speaking/2008-QCon/act3.html) a few years ago. This model categorized software architecture implementations into different levels. These levels differentiate in terms of how much they fulfill the ideas behind REST and especially RESTful HTTP. You'll find a good [description](http://martinfowler.com/articles/richardsonMaturityModel.html) about the model at Martin Fowlers [blog](http://martinfowler.com/articles/richardsonMaturityModel.html).

Level 0 literally means: You're just using HTTP as a communication protocol to fulfil the [port 80 constraint](http://tools.ietf.org/id/draft-blanchet-iab-internetoverport443-00.html) of the internet. Nothing more.

Level 3 and more means, you're probably in REST heaven, because you use stuff like Hypermedia, Resources and HTTP Verbs and so on.

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

Level 2 says: Use HTTP Verbs. Well, ok you have to use two different HTTP Verbs (GET and POST), but they are not really used with the characteristics they have in mind like *[Idempotence](http://restcookbook.com/HTTP%20Methods/idempotency/)* and [Safe](http://restcookbook.com/HTTP%20Methods/idempotency/)*. They are more a tunneling mechanism for HTTP. Additionally other Verbs are not supported.

Level 3 says: Use Hypermedia, which obviously is requires Level 1 to be covered - which is not the case, so its not possible.


### Fielding would probably say: NO

One REST constraint is *Layered System*. If the API Layer knows about the Persistence Layer terminology like in this case (where i can use JPQL Queries in the HTTP Request), we have no layered system at this point.

Another valid objection is, that RESTful HTTP is **not** CRUD over HTTP via the Verbs. And the resources normally should not match 1:1 from entites to HTTP resources, which is kind of the case here.

So lets face it: from the angle of *REST* as an *architectural style*, this API Implementation does not at all fulfil the different requirements to get the *Glory of REST* as Leonard Richardson calls it.


### The story about the two hats

At this point, we could leave this topic. But the above description is made just my theoretical dogmatic Hat on my head. It's ok, i like this Hat. I wear it quite often. It fits quite well and looks gorgeous. But the problems with this Hat are versatile. First of all it is expensive. If you want to wear it, you have to pay a pretty high price. Additionally, it is a Hat, that it is not a every-day Hat. If you want to wear it on a normal business day in the office, probably collegues look at you a little mazed.

Because of this, i'll put this hat beside for now and put on the other one i own: the practical pragmatic one.