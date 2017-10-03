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


### REWE

- discussed way form monolith to microservices in retail space

- Asnychronous > Synchronous
 - "Having data is much better than fetching data" - duplication of data is fine in microservices
 - async event communication through Kafka
 - every microservice only takes the data it requires (like bounded contexts)
 - no event sourcing, but just the latest state in the messages

- UI gateway, which aggegates UI (HTML+CSS+JavaScript) from Microservices and pushes them to the web client
