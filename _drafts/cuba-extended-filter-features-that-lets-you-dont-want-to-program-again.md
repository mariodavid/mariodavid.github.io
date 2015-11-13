---
layout: post
title: CUBA extended Filter features that lets you don't want to program again
description: "..."
modified: 2015-11-13
tags: [cuba, filtering]
---

Last time, i wrote about the generic filter possibilities that CUBA provides out of the box. I left off with two requirements as a developer, that are necessary to be confident that the generic solution does not fall down with real world problems.

<!-- more -->

## Going from fast food to a real healthy meal

After going through the filter possibilities for myself the first time, it felt a little strange. This first thing that i thought was something like *"ok, so WTF did i spent my time on the last few years?"*. Because in the first place it just pretty much too good to be true. Don't need to write this simple filter mechanisms by yourself?


*all orders that are in state X; all products that are in category Y; all orders that have been placed between t0 and t1;  yada yada yada.*

This is just boring stuff from a developer point of view. As you'll get the schema behind it, it becomes pointless. On the other hand, to wrap this generic schema into code that is efficient in terms of SQL statements and slick from UX point of view is actually pretty hard. This is why oftentimes the developer does not go from the specific problem to a more general implementation that can then be used. However, CUBA did exactly that and so there is less need to thing about it.

Going back to the solution we have at hand. After the first attempts to do some stuff with the filtering mechanism, i wasn't aware what to make of it. At this point it feels a little like *fast food*.

OK at the exact moment. But after eating it, you often have a queasy conscience. First of all you probably know that it's going to bite you in the long run when buying a big mac every second day. Additionally you know that two hours later you probably will be hungry again.

So coming back from this bad analogy. the question basically is: Are you on the right track with this solution? Or: Is this is healthy well tasting meal?

Lets recap the two objections i wrote down last time:

* pre-define filter combinations in behalf of the user
* support for complex filters that go beyond attribute values of the or related entities