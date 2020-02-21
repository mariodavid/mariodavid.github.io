---
layout: sf-lens-red
title: Salesforce trough the Lens of a Java Dev
subtitle: Part 3 - Imperative Technology Solutions
description: "This blog post series is about the my learnings that I had during working in the domain of Salesforce Development. This blog post gives an overview of one side of the Technology coin: the declarative development solutions."
modified: 2020-02-28s
tags: [cuba, Salesforce]
image:
  feature: salesforce-through-the-lens-of-a-java-dev-part-2/feature.png
---
This blog post series is about the my learnings that I had during working in the domain of Salesforce Development. This blog post gives an overview of the other side of the technology coin: the imperative development solutions.

<!-- more -->

## Salesforce for Developers

I already showed you a small example that source-code based software development is also available in Salesforce. Indeed the area of what can be done in this area is at least as big as or even bigger compared to the declarative development part.

As Salesforce already has a history of technologies that were supported throught the years, there are various options. We will only focus on the current technology stack.

### Salesforce Javascript Fragmentation Approach

When it comes to the user interface, the technology is Javascript, as Salesforce exclusively is a web based solution.

As for all big tech players that market in particular is very problematic technology wise, as it is very fragmented and fast moving (although it slowed down in the last 1-2 years). This highly contradicts with the requirements of enterprises that normally need loooong living technology lifecycles and support.

Salesforce does not have a silver bullet here. In order to not make a big bet on any Javascript framework that will be irrelevant and gone in two years, they develop their own framework, where probably the main reason is that they are control the lifecycle better, which is necessary to fulfill the enterprise requirement of long support.

SAP went into a similar direction with [SAP UI 5](https://openui5.org/).

To balance those two contradictory forces they have multiple technologies in different phases of their lifecycle in their portfolio. Currently those are in particular these two options:

* Lightning Web Components (newer, build ontop of open Webcomponents standard, does not cover all use-cases, open source)
* Aura Components (more mature, older, propriatery)

This blog post from Salesforce shares some more light on this topic: [Salesforce Developer - Introducing Lightning Web Components Open Source](https://developer.salesforce.com/blogs/2019/05/introducing-lightning-web-components-open-source.html).

This move of adopting open standard and also embracing open source is aligned with what the majority of the software industrie is doing nowadays. It also shows that Salesforce wants to leverare the community effects in the software development world much more then it used to do before (we will see this other examples later).

