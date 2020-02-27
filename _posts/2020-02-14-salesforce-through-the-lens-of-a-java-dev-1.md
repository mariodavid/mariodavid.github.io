---
layout: sf-lens
title: Salesforce through the Lens of a Java Dev
subtitle: Part 1 - Salesforce Landscape overview
description: "This blog post series is about the learnings that I had during working in the domain of Salesforce Development. I will try to put SF into a broader context and compare it to the previous experience - in particular, CUBA"
modified: 2020-02-14
tags: [cuba, Salesforce]
image:
  feature: salesforce-through-the-lens-of-a-java-dev-part-1/feature.png
---
This blog post series is about the learnings that I had during working in the domain of Salesforce Development. I will try to put Salesforce into a broader context and compare it to the previous experience - in particular CUBA.

<!-- more -->

Lately, I had the chance of learning about the Salesforce technology. The same interest that brought me to CUBA Platform in the first place - the interest in higher-level abstractions - was also the reason I wanted to dig deep into Salesforce.

In this blog post, I want to explain what I learned about a platform that is in some sense on a much higher level, in some sense a  high level similar to CUBA platform. I will also put this into a general context of some different approaches to software development.

In the upcoming blog posts about this topic, I will dig deeper into technical details about specifics in the development/administration of a Salesforce application.

Let's start with a necessary introduction about what the "Concept Salesforce" is and how it sets the scene for technical details.

One remark here is that although this blog post is about this particular software vendor, the majority of statements are also applicable to other similar vendors. Additional disclaimer: I am exposed to the Salesforce platform for about half a year. Therefore, those opinions are probably not 100% accurate and some conclusions might also be heavily influenced by that (little) amount of time & understanding I have about this platform.

That being out of the way - let's get started...

### What is this Salesforce

Salesforce (SF) is a lot, this is why it is quite hard to crystallize. So instead of finding the one describing sentence, I will give you a couple of examples:

* SF is a Customer Relationship Management System
* SF is a Software as a Service Offering
* SF is a Platform to _build_ programs on top of
* SF is the MS Access of the Web
* SF is a lot more

To unfold these statements, let's first go into a little exploration of the history of Salesforce as a company and a technology.

### A Brief History on Salesforce

SF started as a CRM system, that is offered for about 20 years in the cloud. It was one of the very first SaaS offerings in the world and it pretty much coined that term.

Salesforce was an invention from a couple of Ex-Oracle employees that wanted to build a competitor to the omnipresent CRM out there back then: Sibel.

SF was revolutionary back then, with this idea of having software not installed and maintained in the enterprise but instead just "consumed" via the web. The payment model has always been (what now is pretty common in SaaS offerings): <code>x $ / User / Month</code>.


<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-1/no-software-logo.png" style="float:right;width:100px" />

For a long time there was a very famous Logo and Marketing Campaign from SF that claimed "no-software", referring to the above-mentioned situation of not having to install & maintain software anymore. Of cfourse, it was still very much software - just not what regular people would associate with the term "software" back then (for the most part).

SF successfully operated in the domain with this multi-tenancy application that was offering CRM in the cloud.

A couple of years of experience in SaaS, a couple of thousand employees and a couple of billion dollars revenue later, SF broadened their offering to a wider range of use cases. SF transitioned towards a platform that was able to offer software for more (related) domains.

Eventually, after this platform (technically) matured enough in terms of runtime quality and abstractions - the platform was opened up for more and more external developers/builders to extend the capabilities of their usage of SF.


### Salesforce Today

Fast forward to 2020 - this platform is still a CRM (as well as other software), but the ratio of CRM to Platform has dramatically changed. It has become a complete Platform for "building" more or less _general purpose applications_.

Besides the enterprises' capabilities to build applications in this runtime, they can also leverage a huge marketplace of off-the-shelf software that are SF applications, but from third-party vendors. This Marketplace goes from simple CRM improvement tools to a complete line of business offerings for domains unrelated to CRM.

The company has grown to a massive size of 30k Employees with a revenue of 10 billion dollars per year.

If you want to read more about the history of SF, there is quite a good book looking into that in much more detail: [Behind Cloud](https://www.amazon.com/Behind-Cloud-Salesforce-com-Billion-Dollar-Company/dp/0470521163).

### SF - Conceptually from 10'000 Feet

Let's shift gears here a little bit from this folklore-style history description to the software platform itself. To understand the SF platform from a software developer's perspective, let's step back a little from our understanding of how software is normally created in a "classical" agile software development process.

There are a couple of concepts that are relevant when we want to understand the way how software is built on the SF platform.

#### Concept 1: Software Development without Code

The headline should actually be "Software Development without Code (and with Code)" - but the other one is a little more controversial and catchy.

What does that even mean: Development without Code? Isn't Development == Code? Is there any difference between those two words?

In fact there is (or at least in the SF world they try to emphasize that to get through their point). Developing or creating software or "creating something that a user can use to achieve a task" is a much broader concept. When thinking about it, I can see various examples that can be categorized into that bucket, but still do not require a person to write source-code (or write even anything at all).

Let's go through a couple of those examples:

* Define a calculation with inputs in MS Excel
* Create automation of a workflow via Mac OS X Automator application
* Configure a webshop via Shopify with PayPal integration
* Adjust the validation behavior of your SAP ERP system
* Creating a JIRA + Slack integration through Zapier

All of those things are in the area of software development, while they don't require the ability to express intends of how a computer should execute a task at hand via "writing special instructions in a source code file".

Salesforce has embraced this area very heavily. There are various offerings in the SF platform itself that range from "defining the field layout and arrangement of a UI screen" to "process automation that executes by certain environment triggers".

In the SF ecosystem, they market it more as "source code is optional". And in fact this is true to a pretty high degree. They stretched this concept to its logical conclusion so that it is possible to build non-trivial applications & workflows without ever considering yourself as a programmer/developer.

But "optional" also means that source code is possible.

If we put the general distinction between code and non-code in an application created by custom software development into an advanced Venn diagram, the ratio between code and non-code parts would look like this:

#### Code vs. No-code Ratio in a General Solution
<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-1/config-vs-code-ratio-generally-2.png" style="width:400px" class="drawing"/>

The numbers represent the parts of a solution that are either achieved by configuration or by creating source code. 1-4 can then be described as:

1. creating a solution only by configuration
2. creating a solution normally by configuration, code variant is possible
3. creating a solution normally by code, sometimes configuration is possible
4. creating a solution only by code

Let's compare that to the domain of applications created in the SF ecosystem.

#### Code vs. No-code Ratio in a Salesforce Solution

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-1/config-vs-code-ratio-sf.png" class="drawing" />


We see a couple of differences there. In general, the size of distinct areas is smaller - in particular the section (4) is much smaller compared to the former picture. THis is not so much because it is not possible to express the same amount in source code compared to a general slution.

But it is because the area (2) + (3) in relativity are much bigger. This means the amount of stuff that can be done by non-code abilities is overall bigger.

There are still parts that are better suited (3) or exclusive (4) to be done by code. But the ratio beteen (3) + (4) compared to (1) + (2) is much more balanced compare to general solution. 

With that observation let's go to the second concept that is worth pointing out:

#### Concept 2: Democratization of Software Development

Writing source code is a skill that not everyone has. Although it can (and should) probably be brought to a much bigger audience of the overall world population, it will stay comparably low.

That being said, as <a href="https://a16z.com/2011/08/20/why-software-is-eating-the-world/">Software is eating the world</a>, there will, for some foreseeable time, be much more demand for people who are able to create software. But as <a href="https://www.forbes.com/sites/peterbendorsamuel/2019/10/14/software-is-eating-the-world-but-services-is-eating-software/">Software is eating the world, but services is eating software</a> there will be a broader need in skills outside of classical software development.

With offering all those higher-level abstractions and tools for creating software by SF, they basically include all those people into software development, that would otherwise be excluded because they are not willing or able to write source code.

If you e.g. think about the number of business people that are able to work with Excel, define calculated cells via formulas, shape some form of UI through Excel, etc., people of this kind are empowered by Salesforce to participate in the creation of software solutions.

This is a very fundamental underpinning of a lot of efforts done by SF. And although this term "Democratization of Software Development" sounds almost like SF is trying to perform a service to the world, most likely there is also a very strong business reason behind it: one of the main audiences of this SaaS offering from Salesforce is exactly these tech-savvy business people, that previously in their job worked heavily with MS Excel et al.


### Speed vs. Flexibility

When it comes to leveraging higher-level abstractions, it boils down to the trade-off between Speed and Flexibility. Back in my [very first blog post](https://www.road-to-cuba-and-beyond.com/my-personal-crud-story-or-how-i-came-to-cuba/) I drew a diagram that points out this trade-off. Let's take a look at a new variant of that and put Salesforce into the picture:


<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-1/speed-flexibility-pyramid.png" class="drawing" />


The diagram shows a pyramid where the vertical area is referring to the speed of development and the horizontal part describes how much flexibility the technology allows a software developer to have.

What can be seen in the diagram is that the lower down the stack the more flexible (potentially) a solution can be.

Something like the Servlet API (referring to a fairly "low level" abstraction in the Java Ecosystem) allows creating a very broad range of applications. A business application, a twitter clone, a brand website, an API backend for a newspaper app, so basically everything these days. But even this low-level abstraction has limitations: it at least requires HTTP as the communication channel, and it also enforces a client-server architecture.

Then there are certain higher-level abstractions, and CUBA is one of them. It already narrows down the main use-cases very much to:

> _business applications with a client-server architecture_

It cuts scope (heavily) and in return gives speed because of default choices and out-of-the-box use-case specific solutions. In CUBA's case this would be something like Access Management, Audit functionality, Reporting, BPM, etc. Do you want to write a business application? - go for it. Do you want to build a Twitter clone? - don't even try with CUBA.

Then you have on the very top of the pyramid something like JIRA. It is a pre-built software that has a very specific business use-case: Ticket Management & Collaboration. This piece of software is probably for the most part not even considered as a software development environment anymore.

But if you think about the configurability and the way you can either _configure_ your desired workflows or even add plugins via code to it, there is still _some_ flexibility in it.

But here (once again), you should neither try to create a twitter clone nor a general-purpose business application. But if you need a solution in the collaboration space with some flexibility, just roll with it.

Where does Salesforce fit now into this picture? Most likely it is somewhere in this wide range between the JIRA example and a CUBA application.

It really depends on how the SF platform is used. The plain _usage_ of the Sales / Services CRM functionality with little adjustments is more on the higher end, where creating a LoB application with custom objects, workflows and a lot of APEX and JavaScript code is more on the lower end of the range.

Digging into this particular topic will be the content of the next blog post, where we will take a closer look at the technical details of SF. Furthermore, I will try to explain the technical differences between a Salesforce application, the CUBA equivalent and a more general Java stack.


### Summary

To summarize this conceptual overview blog post: Salesforce started as a CRM SaaS application for businesses and turned into an application development platform where the CRM module is just one example of its usage.

Salesforce it treating configuration as a first-class citizen and makes coding optional for a lot of cases. In the (short term) speed vs. flexibility, it lives in another category then a CUBA application and even more compared to a more general Java application.

It is different from traditional software development in various ways, but there are also a lot of similarities. What the differences and similarities are, we will unfold in the next blog post.