---
layout: default
title: Thoughts on the CUBA-to-Jmix transition
description: "Recently the creators of CUBA Platform revealed their plans for the next major version of the framework. In this blog post, I want to share some thoughts on those plans and my interpretations on what that means right now and in the long run."
modified: 2020-12-28
tags: [cuba, Jmix]
image:
  feature: salesforce-through-the-lens-of-a-java-dev-part-2/feature.png
---

Recently the creators of CUBA Platform revealed their plans for the next major version of the framework. In this blog post, I want to share some thoughts on those plans and my interpretations of what that means right now and in the long run.

<!-- more -->


## Jmix formerly known as CUBA Platform 8.0

In the last weeks of December 2020 Haulmont, the creators of the CUBA platform announced the plans for the next major version of the platform. In particular, they showed in what shape and form CUBA Platform 8.0 will see the light of day.

The last major version of CUBA has been two years ago. The main additions back then were a re-designed event-based API for the UI parts of the framework, the way to define data loading from the UI as well as a lot of keeping-up-to-date work like support for newer Java versions.

On the broader ecosystem, a lot of stuff changed as well, like the switch of the tooling to be IntelliJ based or the marketplace, which provided an easier way to deliver application components to users.


But one thing, that did not change was the groundwork in terms of Spring. Looking into the broader Spring ecosystem in 2018/2019 it was becoming clear that a regular Spring application was better labeled "classical" as the _normal_ way of doing a new Spring-based application was _Spring Boot_.


### Spring Boot in a Nutshell

If you are not familiar with Spring Boot, let me just try to give you a brief overview.

Spring Boot is just a regular Spring application. But it is a Spring application, that aims to take away the problematic areas that people were suffering from when creating Spring applications in the early years.
In particular, Spring Boot provided an "opinionated way" of configuring Spring applications that fit 80% of the users. Those opinions were put into source code so that it was not up to the developer to configure everything, but only if explicitly needed to differ from the standard way.

Furthermore, with _Spring Boot Starters_ it provided a way for other libraries to do the same favor to their users when they want to integrate with Spring. E.g. the Camunda BPM engine provided a Spring Boot starter that could just be activated in the application via adding a dependency <code>compile 'org.camunda.bpm.springboot:camunda-bpm-spring-boot-starter:7.14.0'</code>. Everything else was done by the so-called "auto configuration" that is delivered as part of this starter.

Additionally, Spring Boot tries to solve parts of the dependency management problem. Before Boot, it was necessary to come up with a possible combination of non-conflicting versions for all of the dependencies. You, as the application developer needed to figure out the right version of Spring Core, Spring Security, Spring LDAP, Jackson, Cucumber, Guava, etc. Depending on how many dependencies you had, this became a non-trivial problem to solve. In particular, if there were inter-dependencies like Spring MVC 4.1 only works with Jackson 2.2 and so on. The reality was (and still is) that this is not something that a regular enterprise app developer should need to take care of daily.

Spring Boot solved a lot of those dependency problems by basically pre-checking working combinations of libraries and putting that under a Spring Boot version umbrella (just like CUBA or Grails do to a certain degree as well).


All of those upsides lead to the situation that within a couple of years, the big majority of newly created apps in the Spring ecosystem took Boot as their base. Other meta-frameworks switched to Spring Boot as well to leverage some of the benefits it provided (like Grails in [2015](https://www.infoq.com/news/2015/04/grails-3/)).

You can see here the google trend comparing the Spring Boot to Spring overall:

<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/2431_RC04/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Spring Boot","geo":"","time":"all"},{"keyword":"Spring Framework","geo":"","time":"all"},{"keyword":"/m/0dhx5b","geo":"","time":"all"}],"category":5,"property":""}, {"exploreQuery":"cat=5&date=all&q=Spring%20Boot,Spring%20Framework,%2Fm%2F0dhx5b","guestPath":"https://trends.google.de:443/trends/embed/"}); </script>
``


## The (long) history of Jmix

While writing this blog post I wrote down that I initially saw the origins within Github at the beginning of 2020. But in fact, when double checking on certain commits that I wanted to link to, I realized that the actual development of the framework started already at the beginning of 2019, which means: almost two years ago. So in fact it has actually been quite a ride, that I want to go through together with you.

For me, this announcement was not really a big surprise, because I was following the [Github activity](https://github.com/Haulmont?q=jmix&type=&language=) since the beginning of _2019_, where the initial development started of Jmix started. I had some discussions with Haulmont team members during those months, but mainly just observed the development during the years as it happened in the open.


During those months, the development grew from a PoC character, which was mainly driven by the question of [how to set up a framework on top of Spring Boot](https://github.com/Haulmont/jmix-old/commit/eba8f6edc51d2c0aeebb42e821b98c1b009ce4ee) to later stages, where a lot of already existing parts from CUBA needed to be "copy & pasted" over to the new structure (like the some random [UI parts](https://github.com/Haulmont/jmix-old/commit/333c9babc14e18c484afb65d3ff97e25853d5924)). Also, new concepts were introduced while others were replaced by their Spring Boot counterparts that provided the same or at least a similar functionality.


## Revisit Fundamental Design Choices

When such an update is performed, for framework developers this is most likely the only time of development where new concepts can be established or existing ones can be adapted. There are actually a couple of them, that I quickly want to discuss here. For a lot of those changes, it seemed to be a revisit of the decisions that have taken place a long time ago. Back then the Java ecosystem was a different one, the deployment patterns were quite different, etc. So it seems to make sense to question some fundamentals that have their roots more than ten years ago.

### DB Migration

In Jmix the topic of DB migrations will be offloaded to Liquibase, a well-known library in the Java ecosystem for solving exactly that task. In 2020 most likely almost no-one would conclude implementing a multi-database migration framework that spits out SQL. Mainly because the open-source ecosystem has solved this problem with Flyway or Liquibase (in the JVM ecosystem) with a level of majority that is hard to reach.

As those projects only take care of this question, the likelihood that you as a framework developer get better at this topic is just quite low. Besides the fact that you spend time doing something that someone else did quite well. "Don't reinvent the wheel" not only applies to applications but framework development as well.

But back then twelve years ago, this decision was probably not so obvious. The tools simply were not _that_ established or mature.

### Scalability Approach

Another example of that is the way to scale applications. Back in the day of Java application servers, deploying in a monolithic fashion was the standard way of doing it. This means that when scaling out, the application as a whole was replicated. Furthermore, the original attempt of the application servers was to run multiple Java applications within one server to share common configuration, security settings, etc.

CUBA embraced that N-tier application architecture style and allowed to scale the different layers independently from each other. Implementation wise this is a non-trivial thing to do. The web and core module interacting either via HTTP or through direct service interaction within the same JVM with each other. By providing that scalability option, it comes with some accidental complexity like object serialization to communicate between the layers, which causes some downstream limitations or problems.

Nowadays scalability is oftentimes solved through vertical splitting rather than scaling up of the different layers. This means, splitting an application by its different functional areas and deploying those independently from each other - and then scale those parts according to their needs.

With the knowledge the industry has moved within the last decade this was also one thing that was presumably taken into consideration for Jmix. Spring Boot is very well known for using it for this kind of scalability and be "microservices-ready". So if this is the case, one honest question one should ask is: "Is it really necessary to keep this multi-layer heavy-lifting approach within the framework?". I assume in some form or the other such discussions took place within Haulmont, where the outcome was: no, actually it is not needed anymore.

The result of that will be great simplifications within the framework as well as in the applications themselves. For the framework, there is no need to codify all this behavior for multi-JVM interaction, different caching solutions for frontend/backend, serialization problems anymore. For the applications that are built on top of Jmix, there is no necessity to run a multi module Gradle project (if not needed for other reasons).


## But what about Backward Compatibility?

What I found in particular astonishing was to observe the task of keeping backward compatibility in place.

Imagine the situation, that you are re-writing a very big part of some piece of a library (like this framework) but as you already have a huge user base, you are mainly locked-in to almost all your APIs/interfaces/classes that you have provided to other developers.

The solution they came up with is to have a particular compatibility module called [https://github.com/Haulmont/jmix-cuba](jmix-cuba) which keeps the implementation of almost all old APIs in place. This allows developers to have a seamless transition to the new version, without noticing any major disruption within the source code.

Using that compatibility module means that you are not automatically using all of the newest APIs with the new Jmix namespaces and so on. But it allows the app developer to continue to update without forcing them to spend a significant amount of time when performing the update all at once. It means that it decouples the task of updating the dependency to the framework from updating all of its usages within the application code.

This pattern of providing a generous update path is paramount, because it puts the app developer in front of the steering wheel of the update process instead of letting the framework dictate when and in what form to perform the overall update.

For a software business that actually is supposed to deliver business value, such an update path is way easier. Because you can balance out the "infrastructure-work" like updating the framework over several weeks/months instead of telling the business to stop every feature development for weeks to come because a dependency update requires weeks of plumbing right now.

This compatibility module will probably also speed up the adoption of the new major version, as it does not require substantial upfront investments.

This level of caring about backward compatibility is quite seldom in this ever-changing software-framework business.
I would assume that with the jmix-cuba module they have already come to 70%-80% of all of the update problems that would normally occur. The remaining part will most likely be covered by the developer tooling aka "Studio".


But I would assume that if you have created specific extensions that are overriding particular internal parts of the framework, this part might be the target of change. So e.g. if you needed to extend the implementation of the <code>DataManager</code> interface I would assume that those parts have to be revisited and adjusted according to their new implementation.


## UI Technology Evolution

One part of the framework of particular importance for business app developers is the user interface. CUBA currently supports Vaadin 8 as the primary way of writing UIs. For end-customer/special UIs it is also possible to use any other UI technology and interact through a REST API with the backend. For React, in particular, they started to provide better support over just a REST API. In particular, the tooling helped to generate frontend code to reduce the amount of functionality that has to be developed on the side of the app developer.

Now, Vaadin has evolved quite a bit over the years with the shift to Vaadin 10+ which already happened quite a while ago. CUBA did not implement support for that version of the framework up until now and instead started to shift more to the React side of things. Haulmont very explicitly talked about the status and the overall situation one year ago in a blog post [Vaadin 10+ as the Future of CUBA UI](https://www.cuba-platform.com/blog/vaadin-10-evaluation/).

For me, it seems, that the React support is something that is going to stay. React as a frontend framework has become so omnipresent, that it is quite a safe bet (compared to three/four years ago). With the omnipresence also comes the huge ecosystem that brings a lot of choices.

But React itself is not a full-fletched UI framework. Instead, it is a library, that mostly takes care of rendering components, but does not provide those itself. This means that actually to have a similar experience to what Vaadin offers, a component library needs to be chosen additionally. There are several ones out there with all different sets of components and different functionalities. [Material UI](https://material-ui.com/), [Ant Design](https://ant.design/) and [Semantic UI](https://react.semantic-ui.com/) are just a few examples.

This is very good for developers when there is a need to have a choice. But with that diversity, the upsides mentioned above with the omnipresence does not really apply to the same degree anymore. Because when Jmix wants to provide tight integration in UI component generation, they actually have to pick one component generation for the application developers to build their tooling around. Also, it means that the community is actually the community of the UI component framework, not of React in general.

That being said, it is relatively easy to take one other specialized UI component that is not available in the corresponding component library and use it in the application vs. doing that same thing from Vaadin.


Besides the point of how good/bad it is to shift to React, one thing that I assume is quite certain is, that the Java-based Vaadin 8 support within Jmix will stay for quite some time (at least multiple years). First and foremost because there are so many big applications out there that are heavily relying on this way of building UIs.

But also because of backward compatibility. If you look into another example of CUBA UI evolution: The CUBA 6 APIs for UIs like <code>AbstractEditor</code> are still being ported to Jmix, although the CUBA 7 APIs have been released already two years ago. This shows to me the dedication to keep old APIs intact and not force developers to rewrite big chunks of their application with each major version update.

I would assume there will also be support for future versions of Vaadin to keep the Java-Development model. Although it is not very common, over the years the idea of writing Java for the UI part of the application has been hugely beneficial for me. Not necessarily because of the language itself, but rather because of the overall experience. Hot-reloading, the ability to debug the UI parts in the same IDEA instance as the backend code, no need to build a REST API with all its consequences, the ability to do UI integration testing without the hassle of the browser involved are just a few examples of that.

Overall, the frictionless development experience is more mature even (or in particular) after working with other frontend technologies (like React, Angular, and others) over the years. But instead of it being the primary way, there will be two ways of building UIs.

How this change relates to administrative screens, which are currently provided by the framework (and the add-ons) is still an open question for me.


## Evolution of the Ecosystem

The last part I wanted to talk about is the topic of what this change might mean for the CUBA ecosystem.

CUBA platform as an open-source project is relatively young with its ~4 years in existence. This is just the tip of the iceberg as the framework has been developed in some form or the other internally since 2008. Also, being a meta-framework the underlying technologies have been established sometimes for almost 20 years. So it means the tech is super mature. But I'm not speaking about is the tech itself, but the "open source ecosystem".

Although the CUBA community has grown over the years, the size in absolute terms is still very low. This is true for the contributor ecosystem as well as the overall user ecosystem. You can see that in various shapes and forms:

* the CUBA platform project is almost exclusively developed by Haulmont
* the marketplace consists of roughly 70 addons, where 30 of them are from Haulmont itself. Removing the 15 or so that I myself did, the remaining number is quite low
* there is no Book, no Pluralsight Course, or some other form of material out there to leverage for newcomers
* no dedicated conferences are held, almost no conference talks are given about this technology

This seems somewhat problematic, at least from a long-term sustainability point of view. Of course, it is true, that there is growth involved but once again: in absolute terms compared to other open-source ecosystems, those numbers are quite low.

The funny thing is that compared to the value it actually provides, for me it feels like a heavy mismatch. When you look into a lot of the technology that is hyped over the years, the value it oftentimes actually provides is minor compared to the buzz it creates. It seems, CUBA platform has never aimed to go down this route, but instead inversed that pattern and provide high value and "don't talk about it that much".

But CUBA platform is positioned in a niche being "fast business apps for non-hardcore devs with one of the most mature technical underpinnings out there". So probably it is not really comparable to general open source ecosystems that have a much broader scope that naturally attract hundreds of thousands of developers.

That being said, the value of embracing more people is still attractive and relevant for various reasons. To address those topics from above, going down the route that Jmix tries to head towards is indirectly addressing those concerns.

Adopting Spring Boot and React will bring the core values and the ideas behind the CUBA platform to more developers. The modularization of the framework will broaden the usage of pieces of it as well. With more developers come more collective knowledge and more add-ons, stack overflow QAs, blog posts, etc.

Of course, those are only my personal assumption, but I think the direction of Jmix with the underlying principles and goals is highly beneficial, not only for the technology itself but also for the broader ecosystem.

Spring Boot will bring an even more mature basis to the platform. Focussing on the core value-chain and "outsourcing" non-core parts of CUBA makes it more mature in those areas and reduces the opportunity cost associated with such endeavors.

Embracing React will potentially broaden UI choices further. If it will become a reality that the speed of development can be increased and the complexity can be reduced to the same level that the Vaadin UI is currently at, then it will really serve as a strong second option to choose from.

So - that being said, I'm really looking forward to what is going to happen in the next year with those new developments.
