---
layout: post
title: JavaOne 2017 Recap
description: "In the first week of October of 2017, I had the chance to participate in the JavaOne conference in San Francisco. In this blog post I'll make a little recap of what I learned during these days."
modified: 2017-04-15
tags: [cuba, java, java-one]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

In the first week of October of 2017, I had the chance to participate in the JavaOne conference in San Francisco. In this blog post I'll make a little recap of what I learned during these days.

<!-- more -->

#### Java Keynote: All about openness

Starting with the first day, there was the [Java Keynote](https://www.oracle.com/javaone/on-demand.html?bcid=5596229112001) from Mark Cavage ([@mcavage](https://twitter.com/mcavage)) and Mark Reinhold ([@mreinhold](https://twitter.com/mreinhold)).

The Keynote was all about openness of the java platform. They talked about their recent annoucement of moving JavaEE to the Eclipse foundation and what that means in terms of openness. Then they annouced that they want to change their release cycles dramatically from feature-based releases with timeframes of 3-4 years to times based releases every 6 month.

Next they talked about the jigsaw project as part of the JDK9. Since this the major feature of Java 9, there has been written a lot before about it. Being able to define modules In Java seems to be reasonable approach and is also the missing piece that other languages had for ages. Since the major libraries also need to support the java 9 module system, time will tell how this works out.

The last notable thing is called project amber. This is an effort that has been around for quite some time. It deals with increasing the features of the Java language itself in order to make it a little better step by step.

Within project amber there are subproject that will deal with certain parts. One is to introduce pattern matching into Java, which is very cool. Next thing is automatic type inference within a function. This would allow to use the keyword <code>var</code> instead of defining the type. Not to confuse with dynamic typing. It is just, that the java compiler will infer the type of the variable from the right side of the assignment.

So great feature enhancements. But they are not part of Java 9. Instead they will be part of ext releases of the jdk. However, combined with the new release schedule, it should out to users much faster. The <code>var</code> thing e.g. will be part of the next (18.03) release in march of 2018.


#### A Competitive Food Retail Architecture with Microservices - REWE

Next up was an interesting and practical talk from two german software developers working for a major retail company called REWE: Sebastian Gauder ([@rattakresch](https://twitter.com/rattakresch)) and Ansgar Brauner ([@a_brauner](https://twitter.com/a_brauner)).

It was a monolith to microservices story as it has been told a few times, but with a practical showcase it is always interesting.

One of the main things was "Asnychronous > Synchronous". So whereever it may possible, async communication should be preferred, because it will highly decouple the elements in the architecture.

With this there comes the second notable takeaway: "Having data is much better than fetching data"
This basically means, that it is totally ok to duplicate data in a MS based architecture. It is not perfect either, but doing distributed sychronous network calls is even worse.

They used Apache Kafka for message based communication. Additionally, the receivers of the message only take the data of the message it requires. This is somewhat related to the idea of a bounded context in domain-driven-design.

Interstingly they said that they decided to not go the event sourcing route, but instead hold the latest state in the messages.

The last quite interesting thing in a MS architecture is the question on how the UI integration works. There are different approaches to this. REWE decided to create a thing, they called "UI-gateway", which aggegates  the UI (HTML+CSS+JavaScript) from Microservices and pushes them to the web client. This means that the microservices have to agree on a particular UI technology (and a version of it too) like React v.15.0. That has some obvious drawbacks, but it was interesting to see that that took that as a trade-off.

The slides for this interesting talk can be found at Speakerdeck: https://speakerdeck.com/abrauner/javaone-2017-a-competitive-food-retail-architecture-with-microservices.


### Ten Simple Rules for Writing Great Test cases

- two presenters: 20+ years of experience in testing

- don't think: "Devs create unit tests, QA creates system tests" but:
  - QA: long running & complex domain knowledge tests
  - Dev: fast running & easy to understand

- test code shoud be considered just like production code
- test one thing


### You Deserve Great Tools: Commit-to-Production Automation at LinkedIn

- CD applied to Open-Source library: Mockito
  - experience from Mockito CD approach: every pull request creates a release
  - code, tests & documentation has to be there, because of automatic release

- LinkedIn CD:
  - 3x3: 3 releases per day & max 3 hours to release
- before: 1x release / month
  - removes "feature-rush" (a few days before the release --> #commits go up, to get it in)
- no pre-release manual verification, per-featuer verification
- if something is hard: do it more often
- resolve flaky tests: running test-suite 1000x at night, count the flaky ones & remove the flaky ones & fix them
- master branch always green in big teams


### Developer Keynote

- Oracle Cloud product annoucement

- Patick Debois - DevOps founder:
  - "ServiceFull" - overuser of services (Github, Google calender, Circle CI...)
  - Promise theory (http://a.co/imya0c4) - take a promise as a fundamental building block which should be the standard idea
  - create promises for your service only on what you can control
  - eliminate single point of failures even when using services (using OpenFaaS instead of Lambda) - it's
  about the choice, in order to keep your promise
  - problem with services: the idea of DevOps does not holds when using external services very much
  - external services: how do they communicate with problems etc. which allows you to fulfil your promises a little better
    - post mordems
    - changelog
    - open sourcing
    - direct access to engineers
    - ...
  - DevOps: not only within the company, but in ServiceFull space: communicate between companies / services


## Wendsday


### How Languages Influence Each Other: Reflections on 14 Years of Apache Groovy

- Groove lives in the C-Family of programming languages
- strong collelation between ruby, groovy and swift
- named parameters also work for regular methods:

{% highlight groovy %}

def rectangle(Map m, Color r) {
  println "$m.width:$m.height $r"
}


rectangle red, width: 100, height: 200

{% endhighlight %}

- @Immutable Annotation to create an immutable class
- Type aliases through import ...ClassName as CN
- Null-Safe operator was invented by groovy



### Testing Containers with TestContainers: There and Back Again

- JUnit testing extension in order to run dependencies in Containers
- solves the problem of starting databases in integrations tests e.g.
- removes the burden of handling with ports and docker env variables
- testcontainers.org
- create specific classes extending GenericContainer, like HazelcastContainer
- docker network support in unit tests
- support for spock
- alternative to arquillian cube



### Docker Tips and Tricks for Java Developers

- Dockerfiles:
  - "apt-get install wget --no-install-recommends" will only install the really necessary packages
  - every RUN cmd will create a layer: combine RUN cmds
  - always use tags in "FROM image"
  - RUN will get executed every time. "apt-get install openjdk-8-jdk" will not guarantee the same version --> specify the versions even in apt-get
  - always use explicit user in Dockerfile
  - "RUN chown +x ..." creates another layer with the executable version of the file / directory
  - using official images or your own with proper tipps (the from above)
  - https://stacksmith.bitnami.com
  - every "docker run" will create another layer of this "contnainer". So 100x "docker run" will add 100x layers
  - docker run --> docker kill && docker rm || docker run -it --rm debian
  - be careful with logs into the filesystem of the container, because of the above
  - docker system prune
  - for java apps: the heap size is normally not defined by docker memory constains: -m
  - newer versions of java can "-XX:+UseCGroupsMemoryLimitForHeap" in order to use the values from -m
  - https://hub.docker.com/r/fabric8/fabric8-java/ <-- good java image
  - multi stage builds: 1st container --> container with mvn e.g.; 2nd container builds from first container
  --> therefore
  - docker run excepts STD_IN & STD_OUT for pipeing etc.
