---
layout: post
title: Decompose the monolith with CUBA application components
description:
modified: 2016-08-09
tags: [cuba, AWS, cloud, ECS, VPC]
image:
  feature: decompose-the-monolith/feature.gif
---

In the latest release 6.3 of the CUBA platform there are a few interesting new features coming up. One of the most interesting ones is the feature of application componentes. In this article we will try to understand the use cases for this and make an example of how to use them.

<!-- more -->

## From big balls of mud and Majestic Monoliths

In recent years there has been a lot of debate about how to decompose monoliths mainly in terms of deployment. Microservices are an extreme variant of this approach. Other approaches appear that are more moderate compared to a real monolithic architecture like [Self containted systems](http://scs-architecture.org/). And then there is the end of the spectrum that either unconscious or conscious choosed to make a monolithic architecture (see [@DHH's](https://twitter.com/dhh) blog post [The Majestetic Monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228#.p2bkbz76i) for further information).

All of this approaches care about the outermost layer of a monolith - the deployment artefacts. Although it is the most important layer to think about to avoid / not avoid monolithic structures, there are other layers that are notable as well.

The one that we want to take a look at right now talks about the components within one deployent artefact. So they are at the code reuse level. Something that has been there for a very long time with something like Windows DLLs, Ruby Gems or Jar files.

### Reuse of Code - a common problem

So you may ask - what are application components then and why could it not be solved with something like plain Jar files. Well, with something like plain Jar files you can share a lot of code. But there are situations where you don't just want a paket for class definitions but a slightly better integration with the actual eco system you are in with your application.

A very good example of this is the [Grails plugin system](http://docs.grails.org/latest/guide/plugins.html). With a Grails plugin there is an additional integration to the Grails framework compared to a plain Jar paket. In a Grails plugin you can register required Spring beans for this plugins, create Domain classes and pre define UI's for your application part that is encapsulated in the plugin. Mainly the difference is that since Grails is a full-stack framework it is a good idea to have a composition model that is full-stack as well.

CUBA application components are from a 10'000 feet view pretty much the same but just for CUBA applications. So let's dive a little bit deeper into what we can do with this and what the use cases are.

## Application components as a way to decompose your monolith

As i understand it, the idea of decomposition for CUBA application had already been in the framework under the name "base project", but the docs were not really talking about it. In the 6.3 release though, the idea has been rethought and generalized. This is at least what the termn "application components" suggests.

There are three common use cases for this feature:

1. share common "platform" code for different applications
2. compose your application through different independent feature sets
3. customize your base application per environment / customer / requirement

An example of the first one would be something like creating a MappedSuperclass for reference data
or for entities with temporal validities. Or if you want to share certain UI scenarios like a wizard in a superclass or a helper class. But in this first scenario the main business logic remains in the applications that use this application component.

The next possible option would be useful if you want to create a deployment monolith, but within this monoliths there are several independent parts like HR, controlling and ordermanagement within your ERP application. These parts could potentially be independently sold but should end within one war file where i just have to login once and stay in this application for "creating a new employee" as well as "rejecting an order". In this scenario every part can be a CUBA application component and then there is one application that literally has no business code in it but just includes this components.

The third option is something like the people at Haulmont describe in their article: [How to Develop a Highly Customizable Product](https://www.cuba-platform.com/blog/how-to-develop-a-highly-customizable-product). In this case they have a base product like this taxi management solution which is the application component. Then there are customizations above the component that just adjust the main application very slightly. This might have different layers of customization as they did with [Sherlock](http://www.sherlocktaxi.com/).
