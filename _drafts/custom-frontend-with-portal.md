---
layout: post
title: Custom front-ends for your CUBA app with the portal module
description: 
modified: 2016-04-18
tags: [cuba, portal, bootstrap]
image:
  feature: 2015-10-29-my-personal-crud-story/feature.jpg
  feature_source: 
---

In this blog post i want to show you how you can create really custom front-ends for your CUBA based application. If you want to have customer facing UI as part of your software, here's how you can create them.

<!-- more -->

When you look through the CUBA [docs](https://doc.cuba-platform.com/manual-6.1), you'll find a part which is often mentioned but not described in great detail: [the portal module](https://doc.cuba-platform.com/manual-6.1/portal.html). This is why i want to cover the topic in this blog post.

## What is the portal module?
The elevator pitch for the portal module is that you can create custom user interfaces which are not directly part of the CUBA application. Instead it can be deployed independently and communicate with the [platform middleware](https://doc.cuba-platform.com/manual-6.1/app_tiers.html). 

Further it does not use the generic user interface of CUBA. Instead the platform gives your the possibility to go down one abstraction level lower. The portal module allows you to create plain Spring MVC based applications. With this you have total control over the "plain" HTTP based communication. With Spring MVC you could either create HTTP based APIs (if you don't want to use the [generic REST API](https://doc.cuba-platform.com/manual-6.1/rest_api.html)) or HTML based user interfaces. For the latter one [FreeMaker](http://freemarker.org/) is included into the portal module as the template engine of choice.

## cuba-ordermanagement's public product catalog
As a basis for this topic we'll use [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement).
The addition that we want to achieve is the following:

* list all available products
* show details of a product in a detail view
* let the user login on the front-end 

This will be achieved by creating a [Boostrap](http://getbootstrap.com) based user interface. Here is the result of the UI:

<img src="https://www.road-to-cuba-and-beyond.com/images/2015-12-31-put-your-island-into-a-box/cuba-start.png">

To get to this point, let's have at look at the different steps to achieve this.

### activate portal module

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### create public product catalog

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### show details of a product

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### Activate front-end login

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.


