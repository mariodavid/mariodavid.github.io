---
layout: post
title: Grails vs CUBA
subtitle: Round 2 - Security and UI approaches
description: "In this blog post series we'll compare the two JVM based web frameworks: Grails and CUBA. This time it is all about security and UI approaches"
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: grails-vs-cuba/feature.png
---

After comparing Grails and CUBA mostly from a persistence and business logic perspective in the first blog post, this time I'll take a look at the frontend as well as at cross cutting concerns like security.

<!-- more -->

<div class="information">

This blog post series is splitted into three blog posts:
<br />
<ol>
<li><a href="https://www.road-to-cuba-and-beyond.com/grails-vs-cuba-round-1/">Round 1 - Data access and business logic</a>
</li>
<li><a href="https://www.road-to-cuba-and-beyond.com/grails-vs-cuba-round-2/">Round 2 - Security and UI approaches</a>
</li>
<li>
<a href="https://www.road-to-cuba-and-beyond.com/grails-vs-cuba-round-3/">Round 3 - Testing, plugin architecture and ecosystem</a>
</li>
</ol>

</div>


## Security
Let's start off with a topic that is relevant for every web application. How to implement security.

In Grails land there is a plugin that integrates [Spring security](http://grails-plugins.github.io/grails-spring-security-core) into the application. Actually there are a pyramid of plugins that are built on each other to provide the full range of security. There are plugins for LDAP, for using access control lists and the like. Since Spring security is such a big security solution, you'll probably find what you need. That said, you will need have the willing to create some kind of glue code in order to make it fly.

In CUBA the security aspects are not part of the ecosystem, but instead they are part of the core framework. CUBA has the basic build blocks "Roles", "Users" and "Groups". Roles and Users are pretty much the same as you get from using Spring Security in Grails. But a Role has a much more clear defined responsibility. A Role can grand or deny access to Entities, Entity attributes, UI screens, UI components within screens etc.

In Grails it is much more generic: Roles define a set of permissions that are granted to the user that has this role. What these permissions are is up to the application developer to decide.

Groups is a concept that has no equivalent in the Grails world. You could kind of try to do something similar with Spring security ACL, but there is definitively a gap there. Groups are a hierachical structure where a user is part of one and only one Group (just like in LDAP). Groups in CUBA define constraints on entities and as well define session attributes that are associated with a particular user.

To make this topic a little bit more accessible, here's an example:

In the application there is the following group hierachy:

* Company Inc.
  * Sales
    * US
    * APAC
    * EMEA
  * Marketing
    * ...

Users in the group US / EMEA / APAC only have the ability to see the customers that are based in this area. To define security permisions like this, you can just use CUBAs Groups mechanism. Basically it works this way: CUBA adds SQL WHERE conditions to the normal query in a transparent manner to the application developer. In your application code you would write <code>select c from demo$Customer c</code> and the framework will add <code>where c.area = 'US'</code> or whatever you configured.

I'll not dig deeper into the topic. If you want to find out more about security in CUBA you can take a look at the two explicit blog posts about this topic: [CUBA Security Subsystem Distilled](https://www.road-to-cuba-and-beyond.com/cuba-security-subsystem-distilled/) and [Security constraints in CUBA](https://www.road-to-cuba-and-beyond.com/security-constraints-in-cuba/)

Besides that very powerful Group feature, there is another topic to talk about. How to get a configuration UI for this topic?

CUBA comes with a UI for managing Users, Roles and Groups. The UI integration is fairly advanced. One example of this is that you can define the above mentioned constraints through a UI wizard that will take the burden from writing the WHERE queries away from you but instead lets you click through the attributes of an entity to define these constraints.

Here's an example of a management screen for the security subsystem within CUBA:

<figure class="center">
	<a href="{{ site.url }}/images/cuba-security-subsystem-distilled/role-master-data-manager.png"><img src="{{ site.url }}/images/cuba-security-subsystem-distilled/role-master-data-manager.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-security-subsystem-distilled/role-master-data-manager.png" title="Role editor with entity permissions">Role editor with entity permissions</a></figcaption>
</figure>

Grails, in fact, has a plugin called Spring Security UI that deals with this topic as well. It has some basic CRUD screens for users, roles etc. The problem with this is in my opinion, that it is not really something that you actually put in front of end users. The reason that I say this is, that since Grails has no common UI pattern, the Spring security plugin team had to decide on their own how to implement the UI for this piece of functionality.

They created a UI based on widgets from JQuery UI, which is fine (although in 2017 probably another choice would have been made), but the point is that normally you as the application developer want to create a unified user experience throughout your application. But unless you choosed to use the same UI library in order to present your app to the user, this plugin will break the overall expericence.

You could argue that you could try to style it so that it looks like yours etc. but this more or less feels like a big hack to me. Imagine you created a web app using React. In this case not only the UI widgets would be totally different, but the interaction pattern would be different too.

In fact this problem has nothing to do with Spring security UI in particular, but with the fact that there is no common UI mechanism in Grails in general. With this, we are at the next topic. It deals with the overall user interface approach.

## The wide ocean of UI

The user interface of a web application is a topic that is so large that we can't really cover all the details here. So instead I'll try to give you a brief overview of what possibilities you have for both frameworks and what the architectual differences of those are.

### Grails just recently got an opinion on UI technologies

In order to try to lay down the Grails story of UI (or at least my understanding of it), we have to go back in the Grails 2.x land. Back then there was <code>gsp</code> which was basically a cooler version of <code>jsp</code>. Grails offered (and still offers) a couple of little helper tags like <code>g:form</code> which was aware of some Grails details and translated to normal HTML. So mainly there was support for *server side rendering UI technologies based on HTML*.

You could add any type of client side UI technologies on top of it, but it was all up to you to decide. When you want to add an angular app in front of Grails, that was surely possible. The interface between those two technologies is the RESTful HTTP endpoints that you define in your Grails controllers.

Basically Grails didn't have a heavy opinion on UI. As always with opinions, this can be a good and a bad thing depending on your point of view.

The good side of this is, that you are free do decide about your UI architecture. As for the architecture, I really like it, and a very good fit for this idea of Grails would be a [recource oriented client architecture](http://roca-style.org). It is mainly relying on server side rendering and doing unobtrusive JavaScript instead of creating a single page application with something like EmberJS or Angular. But you are also totally free to do an Angular or React based SPA - both approaches are possible.

On the bad side of this, you can say that not having an opinion means that your are in charge of deciding what way to go and what technologies to use. This can sometimes be overwhelming, and when we look at the JavaScript landscape it can be very complicated too. Additionally, when we look at the overall value proposition of Grails, it is that as a full-stack framework it covers the full stack and with this it gives guidance for every layer of the stack.

#### Application profiles to the rescue

Although there were attempts to solve this problem in 2.x world, in Grails 3.x the situation has become so much better. Grails 3 introduced application profiles, which lets the user slice the application in a particular way, especially regarding the UI technologies. In the recent past Grails 3 got app profiles for Angular 1 / 2, React and Vaadin as well as the server side rendered HTML approach from above.

Basically, what those profiles are doing is scaffolding as well as giving a easier integration into the Grails ecosystem (just like <code>g:form</code> did).

To summarize it, Grails nowadays has support for the major players in the UI technologies market and with the move of app profiles is somewhat UI-technology agnostic. This has some positive aspects, because going all in on a particular UI framework (even in 2017) is not really sustainable. It also leaves room for app developer decisions on this topic. But it has also the negative aspect, that the application developer can't just don't care about the UI technology and get productive from the first minute.

### CUBA is Vaadin and Polymer

Compared to Grails, it is fair to say, that CUBA has an opinion on the UI technology. It uses Vaadin and with its particular UI architecture. In Vaadin you don't develop in JavaScript, but instead create Java code where some parts get transpiled into JavaScript.

What CUBA offers on top of this, is the declarative UI definition approach. This is based on a XML file that describes the layout and the components of the user interface and a Java controller class, that defines the logic for this screen. This declarative approach look to some degree like developing UIs for Android.

In CUBA the Vaadin based UI is called "generic UI". It is the user interface technology where the platform screens are developed in. Besides the fact that developing web UIs in Java feels more like 2007 instead of 2017, the reality is that creating these UI screens is really fast. That is based on the combination of using a declarative method to describe the UI as well as the fact that there are a lot of high-level components to choose from.

But there are downsides on this UI approach. This UI architecture has a comparably high memory footprint on a per user basis. So, the more concurrent users are using the application, the higher the memory consumption on the front-end web-servers is. This is totally ok for an business / enterprise applications where you can guess the number of concurrent users and can predict the grows of it.


#### End user facing parts with polymer

But what about internet scale applications? This is where it would get tricky, because the total amount of memory is very high (let's forget for a second that there are probably other more problematic parts of such an system like blocking IO, relational SQL databases etc.). Nevertheless, this is probably one of the reaons, why CUBA introduced support for a JavaScript based UI in form of polymer.

CUBA offers some polymer components and does some integration for the REST API. Additionally it adds scaffolding support through their RAD tool CUBA studio. The degree of integration into the backend is comparable to the support Grails offers with their app profiles.

But compared to the Grails world, it seems that the polymer UI approach is an addition for the Vaadin based generic UI for specific use cases. The main UI technology is Vaadin.


#### How CUBA solves the spring-security-ui UI problem
I talked about the fact that having no common UI technology leads to the problematic situation from the spring security UI plugin from above. In CUBA the fact that there is the Vaadin UI as a common interface experience, this situation probably does not happen in this case. When you look at a plugin I created called [runtime-diagnose](https://github.com/mariodavid/cuba-component-runtime-diagnose):

<figure class="center">
	<a href="https://github.com/mariodavid/cuba-component-runtime-diagnose"><img src="https://github.com/mariodavid/cuba-component-runtime-diagnose/raw/master/img/groovy-console-screenshot.png
" alt=""></a>
	<figcaption><a href="https://github.com/mariodavid/cuba-component-runtime-diagnose" title="UI of the runtime diagnose application component">UI of the runtime diagnose application component</a></figcaption>
</figure>

Now imagine what would happen to this UI when you decide to change the apperance of the UI through CSS:

Since I used only used standard CUBA / Vaadin UI components, and you probably will define changes of the UI components in a global manner, my runtime-diagnose screens will change accordingly. This is a fairly big difference and it is the same reason why CUBA is able to deliver already predefined platform screens with the framework.

<div class="information">To be fair, you might be able to accomplish the same result with the spring-security-ui plugin from Grails, but since there is no common ground the normal plugin can use, there is a much higher likelihood that multiple plugins will chose different UI technologies. Then the problem of homogenising the apperance through CSS will get much harder.</div>


With this, we covered everything for the second part. The remaning topics like Testing and the ecosystem as well as a summary of the comparison will be covered in the last blog post.


<style type="text/css">
article.hentry {
  background-color:#ac43ba;
}
</style>
