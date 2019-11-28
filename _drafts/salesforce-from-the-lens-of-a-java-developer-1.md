---
layout: cuba-7-release-party
title: Salesforce from the Lens of a Java Developer #1
description: "This blog post series is about the my learnings tht I had during working in the domain of Salesforce Development. I will try to put SF into a broader context and compare it to previous experice - in particular CUBA"
modified: 2019-11-18
tags: [cuba, Salesforce]
image:
  feature: cuba-7-release-party/feature.png
---
This blog post series is about the my learnings tht I had during working in the domain of Salesforce Development. I will try to put SF into a broader context and compare it to previous experice - in particular CUBA.

<!-- more -->

Lately I had the chance of learning about the Salesforce technology. The same interest that brought me to CUBA platform in the first place - which is the interest in higher level abstractions, was not the reason I wanted to digg deep into Salesforce.

In this blog post I want to explain what I learned about a Platform that is even high level then CUBA platform. I will also put this into a general context of the different approaches to do software development.

In upcoming blog posts about this topic, I will digg deeper into technical details about specifics in the development / administration of a Salesforce application.

Let's start out with a necessary introduction about what the "Concept Salesforce" and how it sets the scene for technical details.

### What is this Salesforce

Salesforce (SF) is a lot, this is why it is quite hard to crisalize. Instead I will give you a couple of examples:

* SF is a Customer Relationship Managment System
* SF is a Software as a Service Offering
* SF is a Platform to _build_ programs in
* SF is the MS Access of the Web
* SF is a lot more

Let's go through them one by one. SF started out as a CRM system, that is offered since round about 20 years in the cloud. In fact it was one of the very first SaaS offerings in the world and it pretty much coined that term.

### A Brief History on Salesfroce

Salesforce was an invention from a couple of Ex-Oracle employees that wanted to build a competitor to the omnipresent CRM out there back then: Sibel.

SF was back then very revolutionary with this idea of having software not installed and maintained in the enterprise but instead just "consumed" via the web. The payment model has always been (what now is pretty common in SaaS offerings): x $ / User / Month.

There was for a long time a very famous Logo and Marketing Campaign from SF that claimed "no-software", refering to that very fact. Of course it was very much software - just not what regular people would associated with the term "software" back then (for the most part).

SF very successful operated in the domain with this multi tenancy application that has been offering CRM in the cloud.

A couple of years of experience in Saas, a couple of thousands employees, and a couple of billion dollars revenue later, SF broadened their offering to a wider range of use-cases. SF transitioned towards a platform that was able to offer software for more (related) domains.

Eventually, after this platform (technically) matured enough in terms of runtime quality and abstractions - the platform was opened up for more and more external developers / builders to extend the capabilities of the platform.


### Salesforce Today

Fast forward to 2019 - this platform is still a CRM (as well as other Software), but the ratio of CRM to Platform has dramatically changed. It has become a complete Platform for "building" more or less _general purpose applications_.

Besides the enterprises own capabilities to build application in this runtime, they can also leverage a huge marketplace of off-the-shelf software that are SF applications, but from third-party vendors. This Marketplace goes from simple CRM improvement tools to complete line of business offerings for domains totally unrelated to CRM.

The company has grown to a massive size of 30k Employees with a revenue of 10 billion dollar per year.


### SF - Conceptually from 10'000 feet

Let's shift gears here a little bit from this history description folklore style to the software world. In order to understand the SF platform from a software developer perspective, let's step back a little from our understanding of normally Software is created in a "classical" agile software development process.

There a a couple of concepts that are relevant when we want to understand the way how software is build on the SF platform.

#### Concept 1: Software Development without Code

Actually, the headline should be "Software Development without Code (and with Code)" - but the other one is a little more controversial and catchy.

What does that even mean: Development without Code. Isn't Development == Code? Is there any difference in those two words?

In fact there is. Developing or creating Software or something that a user can use in order to achieve a task is a much broader concept. When thinking about examples I can see various examples, that can be categorized into that bucket, but still do not require a person to write code (or write even anything at all).
