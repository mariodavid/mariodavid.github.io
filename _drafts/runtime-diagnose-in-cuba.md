---
layout: post
title: Runtime diagnose in CUBA
description: ""
modified: 2017-05-19
tags: [cuba, debugging, runtime, diagnose]
image:
  feature: runtime-diagnose-in-cuba/feature.jpg
---

Doing diagnosis on running applications is oftentimes a very tedious thing. <code>grep</code> through log files is mostly the preferred way of doing it. Since this is not the most efficient way, there are other options. One of those is to execute diagnosis code at runtime. Let's look how this can be done in CUBA.


<!-- more -->

## Investigating on a running system is hard

When you try to resolve an issue in a running system it can be hard sometimes to effectivly getting the information you need to understand the problem. First of all you need to understand the not always so clear bug descriptions you might have gotten from your customers. Sometimes it is reproducible in your test system, other times it is not.

In these cases you have to somehow get further information from the running system. In case you run you software on your own you have to get access to the corresponding log files or application to see the error in real life or at least see the impact it has on the software. When doing boxed software the situation is even worse, because the barrier to get to the required information just is higher and the process is more tedious.

If you already live in this fancy microservices world where you have full control over your running software. In this case you probably have automated environments in place that will raise the visibility of insights into the system.

If not you are in this weired place where trying to identify the actual problem feels like trying hit a target with a bullet in a totally dark room. Probably the majority of software developers have more or less been in this situation from time to time.

If you can identify with this situation or with the one below in the gif, you might be interested in seeing another tool in the runtime-debuggers' toolset:

<figure class="center">
<img src="{{ site.url }}/images/runtime-diagnose-in-cuba/fixing-bug-in-production.gif" alt="">
	<figcaption>fixing a bug in production is like...</figcaption>
</figure>


## Runtime diagnoses for CUBA to the recuse

When i was working in [grails](https://grails.org/) land, there was (and still is) a really cool plugin called the [Grails console](http://plugins.grails.org/plugin/sheehan/console). What that is basically, is just like an interactive [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) for a running application. It has a web UI, so it is possible to write code in this UI and execute it on the fly. This can be used for totally different purposes:

* manually adjust the Log4j settings
* insert data into the running system
* executing business logic you wrote in services e.g.
* shutdown the tomcat server

While some of these features are pretty cool, others are really scary. So this plugin has to be used with caution and properly configured from a security point of view. Or like Spider-Man likes to phrase it:

> With Great Power Comes Great Responsibility

### The grails console for CUBA

I recently created a application component that took the ideas from the grails console and transfered it to CUBA island. Actually this was just the starting point, because i took the chance to look a little bit deeper in the topic of runtime dianose and came up with three different areas of features that are useful for doing analysis on problems in a running system.

### 1. Groovy console for ad-hoc debugging & diagnosis

<figure class="center">
	<img src="https://github.com/mariodavid/cuba-component-runtime-diagnose/raw/master/img/groovy-console-screenshot.png" alt="">
	<figcaption>Groovy console in CUBA</figcaption>
</figure>


## Summary
