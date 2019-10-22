---
layout: post
title: Undifferentiated heavy lifting of business applications
description: "In this blog post, we will discover the insights of CUBA's Security subsystem and how different business security requirements can be achieved."
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

Undifferentiated heavy lifting, a term coined in the origins of AWS is also applicable for other parts of the IT industry. Creating business applications has a lot of things in common with IT infrastructure.

<!-- more -->

A few years ago i stumbled upon the term "undifferentiated heavy lifting". As it turns out it was most prominently used by Jeff Bezos and Werner Vogels, C level executives at Amazon. When you look at the origins of the Amazon Web Services offerings, this is basically the core of why they exists.


## What is undifferentiated heavy lifting

Undifferentiated heavy lifting (UHL) in the context of IT operations describes the situation that there has to be done a lot of stuff in the IT departments, while only a very small part of this is directly related to the business outcome.

To make this a little clearer, let's take a look at a traditional IT department of a ordinary furniture production company - let's call them KEIA Inc. Here's a little list of the most common tasks that the IT department has to manage on a daily basis:

* installing anti virus updates on windows client fleet
* manage cisco switch firmware upgrades
* install physical server for newly to be used ERP system
* update PostgreSQL database for a commonly used business related standard software
* re-configure windows active directory after update to Windows Server 2016 R2

This are examples of "stuff" that has to be done (from an IT perspective) in order to be operational as a company. Well - does it really have to be done? This is exactly what Jeff Bezos meant when he spoke about undifferentiated heavy lifting.

To be more precise the "undifferentiated" in "undifferentiated heavy lifting" refers to that companies oftentimes do not tackle different parts of the value chain depending on how much the parts contribute to it. When we take the process of developing an application in order to automate some business process that business people have to do manually otherwise, there would be sub-efforts like:

* [INFRASTRUCTURE] installing server (physical)
* [INFRASTRUCTURE] installing OS
* [INFRASTRUCTURE] installing DB
* [INFRASTRUCTURE] configuring DB backups
* [INFRASTRUCTURE] configuring app monitoring
* [APPLICATION] develop user authentication
* [APPLICATION] develop database model & connectivity
* [APPLICATION] develop UI for manually entering the required business data
* [APPLICATION] develop automation code that reduces manual work from business people
* [APPLICATION] configure technical logging
* [APPLICATION] develop API to access the data to allow further automation
* [APPLICATION] writing tests to ensure the user can succcessfully enter data into the system
* [APPLICATION] configuring CI/CD environments to automatically deploy new versions


There are several other steps in the different categories that I did not wrote down here. When you take a look at the list, can you see where the "business value" lives? It's pretty hard, right? I'll give you a couple of seconds...

<div>
<img style="height: 150px; float: left; margin-right: 5px;" src="https://media.giphy.com/media/xf20D8HzvTQzu/giphy.gif" />
<img style="height: 150px; float: left; margin-right: 5px;" src="https://media.giphy.com/media/o5oLImoQgGsKY/giphy.gif" />
<img style="height: 150px;" src="https://media.giphy.com/media/aqFRBqGjnznd6/giphy.gif" />
</div>

Did you found it? No? No worries, I will tell you about it. This is the one part that mostly adds business value:

* [APPLICATION] develop automation code that reduces manual work from business people


When you look at all the other tasks I wrote down, we have to be honest: This stuff is something that does not add value to the company. It is *not* some competitive advantage that you gain because you decide to do it.

E.g. you will not outperform your competitors because of how well you manage your Cisco switch firmware. It has to be done - don't get me wrong. There is no excuse to not do it and be in the media because you were hacked due to old firmware and lost all your customer data. But still it will not give you a competitive advantage in most cases.

## Concentrate on the stuff that adds business value

The Amazon answer to this situation was to create AWS (Amazon web services), which removes the burden for the IT operations to do the "plumbing" like setting up physical servers and so on. The history (and the raise of the stock price) showed that this idea was a success. Companies across all industries shift their workloads to the cloud in order to get rid of the stuff that is a _commodity_.

What does that mean in the concrete?

For the IT ops person it does not mean that you don't have to do any of the work anymore. It means that some parts of the plumbing will go away completely. Some of them will be replaced by other, less time consuming tasks and some of them will stay the same.

It means that overall the IT ops person will spend less time on the work that does not add business value. Therefore it opens room for the stuff that adds business value.

In the case of the IT ops person it might mean that (s)he finally finds the time to implement the long awaited secret management system, which will decrease the risk of being part of a security breach (I am just making something up here, but I hope you get the point). Because this is something that adds at least more business value.

It can also mean that the IT ops person has room to find a way to create automative mechanisms to drive down the time to create a new application even further, because the cloud enables heavy automation. With that it helps other people like application developers in the organisation to get rid of related plumbing and therefore free up their time to focus on business value on their side.


## UHL is not tied to IT operations

The idea of "concentrating on the important stuff" is obviously not specific to IT operations. It is globally applicable. So what if we take a look at the development of business applications.

Most likely we will find areas in this area of work, which will bring actual business value but we will also find a lot of stuff, that is more like plumbing. Once again: those kinds of things are _necessary_, just as plumbing is - but they are not _important_ to the business. A subset of the plumbing that is done can also be seen as somewhat of a cargo cult.

### User authentication as an example of UHL

If we look back at the list of tasks to do, you will notice something like "develop user authentication". Let's take this as an example. When we look at it from 10'000 feet, this should be a solved problem, correct? True. Because literally every business application needs a proper user authentication mechanism.

There are variations in what features are needed in particular, that's for sure. But overall this functionality is just something that should not be thought of over and over again.

So why is it the case that developers oftentimes are totally ok with developing user authentication on their own?

Oftentimes the building blocks that an application developer has to deal with for user authentication are pretty "basic". For example one building block would be something like a protocol (like HTTP basic authentication), or an RFC like JSON Web tokens.

They are so "basic", because they represent is the lowest common denominator between all the programming languages, Frameworks and so on.

Oftentimes if the application developer is lucky, there are higher level abstractions on top of those basics provided by each programming environment.

This is a great help for the application developers already. Compared to the situation of implementing a standard protocol in the developers' programming environment, this is already a great win.

In this Java programming environment, there are certain default implementations like "Spring Security", which will provide default functionality in exchange to freedom. If the developer is already in the spring framework ecosystem and also is willing to follow the concepts and conventions from the Spring Security library, this security framework will free the application developer from _some_ UHL.

## UHL is a scale

Above I took the words "Lucky" and "great" to describe the value the usage of higher level abstractions brings. This right there is already interesting. Because those words already imply a value judgement about those higher level abstractions.

However, although a lot of application developers would agree, there would already be a couple of developers who would not.

Why is that, you might think? Well, in fact choosing higher level abstractions is a trade-off instead of a silver bullet.

What basically happens is that the application developer chooses "ease of use" or "short-term speed" with the potential inflexibility of the solution.

This is quite a hard-to-sell exchange for developers. The reason why a lot of developers have problems seeing that as universal good is that normally they will loose flexibility.


## The Flexibility Dilemma

Now it is also worth mentioning that application developers are not all hippie style freedom lovers that shy away from any kind of constraint put on their shoulders.

Instead this behavior normally comes from the experience that come with inflexibility.

Imagine the following situation: At some point someone decided that a particular ETL tool should be used, because it claimed to solve a particular integration problem of transfering data from one system to another. As common for such ETL tools, the data transformations are expressed in a graphical manner. The tool has a quite opinionated approach on how to express data transformations.

Six months into the usage of it the original use case was solved, but related use cases showed already the limitations of that tool quite fast.

Now the developer has to find _workarounds_ within this tool, that solve the use case at hand (at least to a certain degree). To a certain degree the tool is now alieanted in the sense that it is not used anymore in the way the tool creator anticipated. But this misuse causes problems in other downstream areas of the tool.

This is the point where it feels more like trying to make the "Quadratur des Kreises" happen.

Now comes the differece between a software developer and a business person. To a business person this situation feels quite normal. Because they cannot change the tools they use to the same level as software developers. They can to a certain degree (depending on the flexibility of the tool) and they are also much more comfortable with alieanting tools.

Software developers on the other hand are much closer to the idea of changing the tool, because by their vary nature they are tool builders.
