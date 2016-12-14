---
layout: post
title: Testing of CUBA applications - Part 1
description:
modified: 2016-09-25
tags: [cuba, Testing]
image:
  feature: testing-of-cuba-application-part-1/feature.png
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
