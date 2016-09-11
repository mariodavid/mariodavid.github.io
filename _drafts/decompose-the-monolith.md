---
layout: post-black
title: Decompose the monolith with CUBA application components
description:
modified: 2016-08-09
tags: [cuba, application components]
image:
  feature: decompose-the-monolith/feature.gif
---

In the latest release 6.3 of the CUBA platform there are a few interesting new features coming up. One of the most interesting ones is the feature of application componentes. In this article we will try to understand the use cases for this and make an example of how to use them.

<!-- more -->

## From big balls of mud and Majestic Monoliths

In recent years there has been a lot of debate about how to decompose monoliths mainly in terms of deployment. Microservices are an extreme variant of this approach. Other approaches appear that are more moderate compared to a real monolithic architecture like [Self containted systems](http://scs-architecture.org/). And then there is the end of the spectrum that either unconscious or conscious choosed to make a monolithic architecture (see [@DHH's](https://twitter.com/dhh) blog post [The Majestetic Monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228#.p2bkbz76i) for further information).

All of this approaches care about the outermost layer of a monolith - the deployment artefacts. Although it is the most important layer to think about to avoid / not avoid monolithic structures, there are other layers that are notable as well.

The one that we want to take a look at right now talks about the components within one deployent artefact. So they are at the code reuse level. Something that has been there for a very long time with something like Windows DLLs, Ruby Gems or Jar files.

### Reuse of code - a solved problem?

So you may ask - what are application components then and why could it not be solved with something like plain Jar files. Well, with something like plain Jar files you can share a lot of code. But there are situations where you don't just want a paket for class definitions but a slightly better integration with the actual eco system you are in with your application.

A very good example of this is the [Grails plugin system](http://docs.grails.org/latest/guide/plugins.html). With a Grails plugin there is an additional integration to the Grails framework compared to a plain Jar paket. In a Grails plugin you can register required Spring beans for this plugins, create Domain classes and pre define UI's for your application part that is encapsulated in the plugin. Mainly the difference is that since Grails is a full-stack framework it is a good idea to have a composition model that is full-stack as well.

[CUBA application components](https://doc.cuba-platform.com/manual-6.3/app_components.html) are from a 10'000 feet view pretty much the same but just for CUBA applications. So let's dive a little bit deeper into what we can do with this and what the use cases are.

## Application components as a way to decompose your monolith

As i understand it, the idea of decomposition for CUBA application had already been in the framework under the name "base project", but the docs were not really talking about it. In the 6.3 release though, the idea has been rethought and generalized. This is at least what the term "application components" suggests.

There are three common use cases that come to my mind for this feature. They allow the end result either to share code in various degrees or customize it in a more structured manner. We'll go through this uses cases right now.

<figure class="center">
<img style="width: 400px;" src="{{ site.url }}/images/decompose-the-monolith/comopnent-options-legend.png" alt="">
</figure>

<hr>

#### 1. product line platform

<a href="{{ site.url }}/images/decompose-the-monolith/component-options-1-product-line-platform-big.png"><img style="width: 256px; float:right;" src="{{ site.url }}/images/decompose-the-monolith/component-options-1-product-line-platform.png" alt=""></a>

An example of the first one would be something like creating a MappedSuperclass for reference data
or for entities with temporal validities. Or if you want to share certain UI scenarios like a wizard in a superclass or a helper class. But in this first scenario the main business logic remains in the applications that use this application component.

<hr>

#### 2. Component composition

<a href="{{ site.url }}/images/decompose-the-monolith/component-options-2-component-composition-big.png"><img style="width: 256px; float:right;" src="{{ site.url }}/images/decompose-the-monolith/component-options-2-component-composition.png" alt=""></a>

The next possible option would be useful if you want to create a deployment monolith, but within this monoliths there are several independent parts like HR, controlling and ordermanagement within your ERP application. These parts could potentially be independently sold but should end within one war file where i just have to login once and stay in this application for "creating a new employee" as well as "rejecting an order". In this scenario every part can be a CUBA application component and then there is one application that literally has no business code in it but just includes this components.

<hr>

#### 3. Product customizations

<a href="{{ site.url }}/images/decompose-the-monolith/component-options-3-customizations-big.png"><img style="width: 256px; float:right;" src="{{ site.url }}/images/decompose-the-monolith/component-options-3-customizations.png" alt=""></a>

The third option is something like the people at Haulmont describe in their article: [How to Develop a Highly Customizable Product](https://www.cuba-platform.com/blog/how-to-develop-a-highly-customizable-product). In this case they have a base product like this taxi management solution which is the application component. Then there are customizations above the component that just adjust the main application very slightly. This might have different layers of customization as they did with [Sherlock](http://www.sherlocktaxi.com/).

<hr>

The three use cases differ mainly in the amount of code that is placed in the components and obviously the purpose of use. Of course these patterns of component usage can be combined if required. E.g. a component composition can also have a product line platform for each (or a subset) of the components. Product customizations can additionally be part of this scenario where the either customize the application as a whole or just certain business components.

Actually there is a forth use case that i will not really go through since this is currently not possible in CUBA, but is a very interesting one: Share components in marketplaces. Grails has a [central plugin mechanism](https://grails.org/plugins.html) that allows users to share plugins with the community. You can think of it as open source add-ons to the framework. But they can also be paided extensions for common business problems, like it's done with the CUBA application components: *reports, fts, charts and bpm*. Currently CUBA has only a distribution channel for it's own commercial components, but probably it's not a huge deal to get something like this going.


## Application components in action

As an example of this i created an Github repository: [CUBA example: application components](https://github.com/mariodavid/cuba-example-application-components/tree/master) that has different of the described use cases implemented.


<figure class="center">
	<a href="{{ site.url }}/images/decompose-the-monolith/project-management-app.png"><img src="{{ site.url }}/images/decompose-the-monolith/project-management-app.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/decompose-the-monolith/project-management-app.png" title="The project management app uses different application components like the appointment featureset">The project management app uses different application components like the appointment featureset</a></figcaption>
</figure>

It consists of three following three parts:

* appointments (*Component composition*)- an application component that is a subsystem for managing appointments
* project-management-platform (*product line platform*) - an application component which provides certain base classes or common shared code
* project-management-app - the application that has code for managing projects combined with the possibility to add appointments to projects and use shared code from the platform component. First we can take a look at the appointments component.

### Component composition: appointments

When you clone the repository and open up the appointments sub directory in studio it appears as a normal CUBA application. You can run it and what you will see is everything that this component contains. A CRUD user interface for creating appointments together with reminders for this appointments. So nothing special about this one.

The only thing that is different to a normal CUBA application are two things:

1. the module prefix for this component has to be specified and changed from "app" to something like "company" in this case.
2. The "App component descriptor" has to be created (see corresponding Link in Project Properties).

I did this already for the appointments component - so you don't have to do any of these steps for this example. But what you have to do is to get a compiled version of this component into your local maven repository. To do this run the gradle install task. In studio this is possible via the global search icon and then typing "install". This is necessary because you downloaded the raw sources for this dependency. But when you want to use this in the upstream components or applications (like project-management-app), you need a compiled version that can just pulled via maven dependecy management.

Alternativly i created a bash script [install-components.sh](https://github.com/mariodavid/cuba-example-application-components/blob/master/install-components.sh) that you can run in the root of the git repository via:

{% highlight bash %}
$ git clone https://github.com/mariodavid/cuba-example-application-components.git
$ cd cuba-example-application-components
$ ./install-components.sh
{% endhighlight %}

### product line platform: project-management-platform

The next component is an example of a application component that falls in the first category: product line platform. Is would probably contain code that is reused across different CUBA applications or application components. Examples of code that could be shared in these types of componens would be:

* static layout files
* Abstractions for UI screen patterns
* common Services (like the [SayHelloService](https://github.com/mariodavid/cuba-example-application-components/blob/master/project-management-platform/modules/global/src/com/company/platform/SayHelloService.java) in the project-management-platform component)
* MappedSuperclasses for entities


For the third approach i don't have a running example. It would probably be something like [extending existing functionality](https://doc.cuba-platform.com/manual-6.2/extension.html). In the case of the appointment component it could be that the project-management-app would define the Appointment entity to add another property "priority".

But i think you should be able to imagine what you can do with this decomposition mechanism. For me at least, coming mainly from the Grails world, it was one important missing piece of the puzzle and i'm very happy it is now included in the platform and easily accessible through studio.

If you have any questions around that topic, you can leave a comment or write me an email.
