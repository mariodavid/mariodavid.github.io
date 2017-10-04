---
layout: post
title: JavaOne 2017 Recap
description: "In the first week of october of 2017, I had the chance to participate in the JavaOne conference in San Francisco. In this blog post I'll make a little recap of what I learned during these days."
modified: 2017-04-15
tags: [cuba, java, java-one]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

In the first week of october of 2017, I had the chance to participate in the JavaOne conference in San Francisco. In this blog post I'll make a little recap of what I learned during these days.

<!-- more -->

### Javs Keynote: All about openness
- release of JDK9
- opensource of JEE
- faster release cycles
- jigsaw
- project amber http://openjdk.java.net/projects/amber/ (type inference - var)


### A Competitive Food Retail Architecture with Microservices - REWE

- discussed way form monolith to microservices in retail space

- Asnychronous > Synchronous
  - "Having data is much better than fetching data" - duplication of data is fine in microservices
  - async event communication through Kafka
  - every microservice only takes the data it requires (like bounded contexts)
  - no event sourcing, but just the latest state in the messages

- UI gateway, which aggegates UI (HTML+CSS+JavaScript) from Microservices and pushes them to the web client


### Nashorn: What’s New in JDK 9

- JavaScript runtime on the JVM since JDK8
- use cases:
  - react prerender on server side
  - embedded scripting
  - use $EXEC to communicate with system operation systems
- able to use all java classes and SDK in javascript


### NoSQL? Have It Your Way!

- Overview over Java APIs to access NoSQL databases
- direct Driver APIs
- Object Mappers either part of the driver, or explicit Mapper projects
- Store agnostic mappers: Spring Data, Hibernate


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
  
