---
layout: post
title: Testing of CUBA applications - Part 1
description:
modified: 2016-12-19
tags: [cuba, Testing]
image:
  feature: testing-of-cuba-application-part-1/feature.jpg
---

One of the main parts of today's application development that is not very prominent in CUBA applications is automated testing. This is the reason we'll take a deep look into this topic in this blog post.

<!-- more -->

### The story about automated testing

Automated testing of software has a pretty long history. Since there is a general need to know if a certain program does what it is supposed to do, the producer of the program might either test the result manually or, through another program that does the job.

Programmers are normally lazy since this is the very nature of their craft and the results of their craft are things that enable other people to be lazy as well (we normally call these programs "productivitiy tools" but they could also be seen as "laziness tools"), it is not a big suprise that doing manual testing of the resulting program is not a very good fit.

The problem of ensuring that the resulting program does its job right can be tackled from two different angles.

The first is to delegate this task to other persons. This approach has been done very successfully (at least for the programmers) for quite a long time. QA teams evolved next to the dev teams and the over-the-wall syndrom was cultivated just like with the operations people.

The second approach is to leave the responsibility for QA at the developers. This leads (since the above described laziness) to automation of this task.

*This is obviously a highly over simplified description of the situation and therefore should be treated with a ;)*

Because the first approach has quite a few downsides, we'll take a closer look at the second approach to the problem: How can we test the software we wrote in an automated fashion?


## General ways of testing

When we look at general options on how to test a software in an automated fashion, normally there are diferent kinds of testing. One dimension is the granularity of the test and the system under test (SUT).

Starting with *unit testing*, which is the testing style with the smallest scope. In a unit test, a unit is tested in isolation. In a object orientated language a unit oftentimes means a class, but this one to one mapping is not necessary. It could be a method or a package as well, depending on your definition of a "unit", but for now we can stick to the "unit = class" mapping.
In order to test a unit in isolation it is required to cut off the dependent units. This is normally done through a technique called *Mocking* or *Stubbing*.

The next granularity step would be an *integration test*. In an integration a unit is not tested in isolation anymore, but instead different units are combined together. The resulting system of the different units are tested in conjunction. Multiple units can be something like multiple spring beans or classes, but it can also mean the database access code in the application server together with a real database.

In comparison to a unit test, where the dependencies are stubbed out, the real dependent units are used. This normally leads to a more "real" test scenario. On the other hand it will make a test slower and more brittle, because of more moving parts that are involved. Additionally it makes it a bit harder to reason about, because if a test shows that a certain feature does not work, it is harder to break down the problem to the actual unit with the bug.

On the other end of the spectrum of the granularity there would be something like a functional test. Functional tests (or sometimes called "system tests", "UI tests") normally differ from the first two options in that it treats the system under test as a black box, meaning that there doesn't need to be information about internal structures of the system in order to setup or execute the test.

Such a test treats a system as it is, meaning that is normally uses the exposed interface to interact with it. This means that a system has to be up and running completely as well as potential dependent systems. As the size of the system under test increases, the setup costs as well as the brittleness of the test execution increase as well. On the other hand a succeeding functional test gives you much higher level of confidence compared to a unit test.

#### Testing pyramide

#### TDD

## How to do unit testing

#### spock

#### simple unit tests

#### mocking

## Testing CUBA artefact types

#### Entities

#### Services

#### Controllers
