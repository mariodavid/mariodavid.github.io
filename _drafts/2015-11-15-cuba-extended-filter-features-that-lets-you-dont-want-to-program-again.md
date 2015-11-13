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

After going through the filter possibilities for myself the first time, it felt a little strange. This first thing i thought about it, was something like *"ok, so WTF did i spent my time on the last few years?"*. Because in the first place it is a little too good to be true. Don't need to write this simple, boring and similar filter mechanisms by yourself?


*all orders that are in state X; all products that are in category Y; all orders that have been placed between t0 and t1;  yada yada yada.*

From a developer point of view this is just boring stuff. As you'll get the schema behind it, it becomes pointless. On the other hand, to wrap this generic schema into code that is efficient in terms of SQL statements and slick from UX perspective is actually pretty hard. This is why oftentimes the developer does not go from the specific problem to a more general implementation that can then be *used* instead of *developed*.

Going back to the solution we have at hand from the last blog post. After the first attempts to do some stuff with the filtering mechanism, i wasn't aware what to make of it. At this point it feels a little like *fast food*.

> OK at the exact moment. But after eating it, you often have a queasy conscience. First of all you probably know that it's going to bite you in the long run when buying a big mac every second day. Additionally you know that two hours later you probably will be hungry again.

The question basically is: *Is this is healthy well tasting meal?*. Meaning: Are you on the right track with this solution, or will this tool fall off after the first non-obvious filter problems?

For this to figure out i will go through the two objections from the last blog post.

## 1. pre-define filter combinations in behalf of the user

Many times there is a need for the developer to pre-define filter possibilities on behalf. You can think of it as different categories in terms of intervene through the user as well as the developer. Below you'll see a diagram that shows this classification.

<figure class="center">
	<a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/pre-defied-filter-options.png"><img src="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/pre-defied-filter-options.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/2015-11-15-cuba-extended-filter-features/pre-defied-filter-options.png" title="Different categories for pre-defined filter implementations">Different categories for pre-defined filter implementations</a></figcaption>
</figure>

### 1.1 pre-defined generic filters

The first option that CUBA gives the user, it that a filter which is defined by the user as we saw in the last blog post, can be saved within the application. To do this, the Filter section in the table has a little *Config* icon in the left, that lets the user either 

<<<<WEITERMACHEN>>>>

## 2. support for complex filters that go beyond attribute values of the or related entities



