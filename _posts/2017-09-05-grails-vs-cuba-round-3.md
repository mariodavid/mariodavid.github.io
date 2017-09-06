---
layout: post
title: Grails vs CUBA
subtitle: Round 3 - Testing, plugin architecture and ecosystem
description: "In this blog post series we'll compare the two JVM based web frameworks: Grails and CUBA. This time will cover the missing bits like testing, the plugin architecture and the ecosystem"
modified: 2017-09-05
tags: [cuba, business applications]
image:
  feature: grails-vs-cuba/feature.png
---

In the last part of the Grails vs. CUBA series, it is all about the remaining parts like Testing as well as the ecosystem. After covering the last parts of the comparison there will be a summary.

<!-- more -->

## Testing

The testing support for both framework is a topic, where again the approaches differ fairly heavy.

Grails traditionally has great testing support. In the [docs](http://docs.grails.org/latest/guide/testing.html) it states "Automated testing is a key part of Grails". For every level of the testing pyramid Grails offers essential support and or syntactic sugar in order to create unit tests in a fast and readable way. It uses the testing frameworks spock and Geb to provide and enhances the integration with some Grails specifics.

CUBA in comparison has some testing support, but it seems to be less into this topic. CUBA uses JUnit and adds some common classes for testing different parts like unit / integration testing of controllers.

Technically there is a difference between CUBA and Grails in the area of integration testing.

In CUBA an integration test means that the spring context is configured. There is the possibility to start up the database either manually or automatically when the first integration test starts. But this generally is layed upon the user to decide what configuration is most suitable for them. Therefore there is also no native support for someting like automatic Rollback of transaction in order to not let different integration tests to interfere with each other.

## Built time plugin architecture

The plugin architecture is present in both frameworks, and the ways they work are actually very close to each other. Basically a Grails plugin or a CUBA application components is a fully functional application that is added to the real application at built time. It can be started and contains all the application artifacts like UI definitions, Spring beans, database entities etc.

CUBA has a bit more advanced features of extending functionality like creating a Customer class in an app component as a basis and extending and basically replacing it in the application or another application component with something like a Customer that has additional attributes.

Grails has a public plugin repository or marketplace that CUBA currently does not have. Therefore the distribution of open source plugins is a little bit harder.

As a result of this, there are only a very few of these application components in CUBA while Grails has a fairly big variety of different plugins available at your fingertips. Since in a couple of weeks CUBA will start a marketplace as well, there is pretty much feature parity in this space.

## platform features and eco system

Generally the approach to shift essential parts into plugins is a double-edged sward. On the one hand it is great to have a viral plugin ecosystem, because it will make the processs of enhancing the framework easier. The barier to entry for getting stuff into the core of a framework is pretty high, where as creating a plugin on your own will not need any upfront effort at all.

Nevertheless a plugin ecosystem oftentimes leads to the situation where it is not always clear what to chose, which plugins are in good shape, which one are a safe bet in terms of functionality and reliablitly. This means, that having all-plugin approach is not very valuable either. Grails strives more towards the camp of all-plugin where as CUBA is more in the all-core camp.

This is probably the reason why CUBA is called "platform" and not framework. If you take a look at a running application when you look at the "Administration" menu, you'll probably notice a lot of so called "platform" features that will come out of the box with it. This would be something like the "entity inspector", a "JMX console", a "user session list" etc. Grails does not have anything close to that (mainly because there is no shared UI approach, which is a big requirement for something like this).  

Everyone has to decide on their own if feature richness is more valuable than a small footprint here.



## Summary

With this we have covered pretty much all relevant aspect of both frameworks. There are some parts I did not pay too much attention to, because either both approaches are too similar or I just forgot about it :)

To summarize the comparison, it is safe to say that both frameworks are very close to each other when we look at the whole universe of web based frameworks especially beyond the borders of the JVM. When I would have chosen to compare CUBA with something like Phoenix, it would be much more difficult to compare. Therefore the transition between those two frameworks is pretty simple (in both ways).

The main difference between those two is the mindset behind it. Where Grails position itself more as a general purpose web based framework, CUBA has a clearer focus of the usage pattern.

Therefore Grails oftentimes will chose its decisions based on broader use cases like creating an online shop or a mobile UI with a REST backend. CUBA in this regard often takes the business application into account and develops features more around this topic.

One example of this broad description would be the recent addition of reactive features for the app itself as well as the connection to datastores like MongoDB. This is a very reasonable approach for the general purpose usage. NoSQL databases as well as asynchronous IO is getting bigger and bigger overall. But the market of business applications has comparably small usage percentage (compared to other areas) of these approaches.

I hope I could bring you both frameworks a little bit nearer. I personally think both of them have their right to exist. Which one is more suitable in your context can just be answered by yourself.

<style type="text/css">
article.hentry {
  background-color:#ac43ba;
}
</style>
