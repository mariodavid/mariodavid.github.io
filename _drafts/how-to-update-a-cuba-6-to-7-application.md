---
layout: cuba-7-release-party
title: How to update a CUBA 6 app to CUBA 7
description: "In this blog post let's celebrate the long awaited big major release 7.0 of CUBA which brings great polished APIs and updates of the underlying libraries."
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: cuba-7-release-party/feature.png
---
<img src="/images/cuba-7-release-party/star.png" />

In this blog post let's celebrate the long awaited big major release 7.0 of CUBA which brings great polished APIs and updates of the underlying libraries.

<!-- more -->


### A path to adopt: CUBA bakery distribution

Let's look at an imaginary application that leverages the CUBA ecosystem. The application is an internal bakery company application dealing with managing the distribution of bakery products into their branches. This application is a CUBA 6.9 app. It has round about 50 screens and a domain model containing of 80 entities. Furthermore it uses the following application components:

* multi tenancy
* runtime diagnose
* attachable
* user session attribute
* dashboard
* zookeeper

Additionally it has two internal application components, that act as the base for similar CUBA applications of the company as well as some special integration app component with another system, that was desinged to be somewhat independent of the main application.




This is true for public available (as well as internal) application components, that have been build or a particular version of CUBA. But it is also true for the applications itself, for example the adjustments that are required by updating to CUBA 7 through breaking API changes / concept changes.
