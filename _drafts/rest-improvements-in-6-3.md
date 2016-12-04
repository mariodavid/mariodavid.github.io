---
layout: post
title: REST improvements in CUBA 6.3
description:
modified: 2016-09-25
tags: [cuba, REST]
image:
  feature: rest-improvements-6-3/feature.png
---

Next to the application components, there is another interesting feature that is part of the 6.3 release of the CUBA platform. It deals with the generic REST API feature of the platform. In this blog post we will have a deeper look into it and see where are the differens and potential benefits of it.

<!-- more -->

## Once upon a time, there was a blog post

Last year i did a [blog post about the REST-API implementation of the CUBA platform](https://www.road-to-cuba-and-beyond.com/cubas-rest-api-if-only-roy-would-knew-about-it/). In the current release 6.3 the generic REST API has been more or less completely redesigned. Due to the massive changes, it worth taking another look into it.

Just to give you a quick recap on what the old (v1) REST API was all about here's a little list of features that have been included:

* (API-)user authentication
* CRUD on single entity instances
* get a list of entities through a JPQL query definition
* execute business logic through service calls via HTTP
* up- / download files from file storage
* create views to customize entity representation retrival

In the new implementation the feature list pretty much stayed the same, but the way to execute the different features changed quite a bit.

So let's look at the first thing that we have to do in order to communicate with the new REST API.

### Authenticate via OAuth2

The first major change concerns the login mechanism. The v1 based login mechanism was based on a homegrown login mechasim, which basically goes like this:

API client sends a GET / POST request to the server


{% highlight bash %}
$ http --form --auth client:secret  POST localhost:8080/app/rest/v2/oauth/token grant_type=password username=admin password=admin

$ http localhost:8080/app/rest/v2/entities/cerv\$Customer 'Authorization:Bearer e0e0eb48-5617-4d60-871d-066883908eec'

{% endhighlight %}
