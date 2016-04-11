---
layout: post
title: CUBA goes open source with a handful of features
description: CUBA Platform version 6.1 was released in the beginning of april. In this blog post i'll give you a quick overview about new features as well as the new license policy
modified: 2016-04-9
tags: [cuba, open-source]
image:
  feature: cuba-6-1/feature.png
  feature_source: https://pixabay.com/en/long-island-iced-tea-cocktail-drink-880919/
  background: cuba-6-1/bg.png
---

Since the latest release from CUBA - Version 6.1 has a fair amout of features and due to the fact that this version of CUBA is the first open source release, here's a quick overview of what has changed.

<!-- more -->

It's been quite a while since CUBA Platform released a minor version of the platform. So, I was curious about what 6.1 Version will bring to the table. Let's start with the first and the most straightforward topic, that lies on the surface: The license change.

## From now on: Open Source with Apache

<img style="float:right; padding: 10px; width: 200px;" src="{{site.url}}/images/cuba-6-1/open.png">
With Version 6.1 CUBA platform is no longer under a commercial license but rather choosed the [Apache 2 license](https://github.com/cuba-platform/cuba/blob/master/LICENSE.txt). Some of the features remain commercial but on a developer basis.

This is a fairly large shift and I think it's worth noting. Since going from a license where the customer has to pay per *running instance & entity quantity & concurrent user* to a model where the base variant is free and the commercial part is based as a developer fee, is quite heavy in terms of mindset shift from the people behind CUBA. 

But this is only the first part. Besdies opening sources of the platform, the CUBA Team also open the development process, which is explicitly announced on their [website](https://www.cuba-platform.com/framework):

> The source code of the platform is also available on [GitHub](https://github.com/cuba-platform/cuba). We encourage developers to submit pull requests to contribute to the CUBA Platform development.

This is a pretty bold statement. I am curious how this will work out and i'm really looking forward to it.

My personal opinion is, that although commercial developer tools have their place in the world of software for developers, it is quite limited. This is especially true because of the rise of open source software in the past few years. 

Due to the fact that software developers are most familiar with the concept of open source, it's not suprising that the acceptance for open source is most prominent in this space. Or to put it the other way around: the willing to go for non-open source software tools from developers decreases from year to year.

When you look at the size of the resulting community for CUBA Platform that was created since they came up with it for the english speaking world, it seems that it reflects this observation.

This all leads to a situation where I think going open source with CUBA Platform makes a whole lot of sense. The amount of potential users will increase dramatically and perhaps there will be contributions from the community that will push everything forward.

After adding my two cents to this topic, let's get to the technical changes. I will just go through some of the features from the 6.1 release as well as the 2.1 relase of CUBA Studio. If you want to get information about all the changes, you will find the changelogs for the [platform 6.1 release](http://files.cuba-platform.com/cuba/platform/platform-6.1-changelog.html#6.1.1) as well as for [studio 2.1 release](http://files.cuba-platform.com/cuba/studio/studio-2.1-changelog.html#2.1.1).

## Built-in Groovy support

The first thing I found was the built-in groovy support in studio 2.1. When you have a look at the project properties of your project, you can activate groovy support. 


<figure class="center">
	<img src="{{ site.url }}/images/cuba-6-1/groovy-support-studio.png" alt="">
	<figcaption>Activate groovy support for the project in the project <i>Project properties > Advanced</i> tab</figcaption>
</figure>

After doing this, when you create a screen or a service, studio will ask you about the language for generation. It's a little more convenient as the solution i described in the [blog post](https://www.road-to-cuba-and-beyond.com/groovify-cuba-app-integrate-with-cuba/) about Groovy and CUBA.

But it's a little more limited as well. For a particular reason i'm not aware of you can just create Controller classes as well as Services in Groovy. Enums, Entities and Entity listeneres are created in Java. But this seems not to be a big deal, because the template approach from the other post still works. So it's up to you to customize it to your particular needs.

## Custom UI elements

One thing that has not been that accessible in the past is how to customize the UI in a way that you can use custom elements. Before it was possible to pull in Vaadin add-ons to get around that or if a really custom UI is required, to create some hand crafted UI in the portal module.

Since Version 6.1 CUBA allows the developer to pull custom elements in three forms: *Vaadin addon*, *Javascript Library* or as a *GWT Component*. Especially the Javascript part is interessting from my point of view, because it opens up CUBA to a very broad set of components.

I will not go through the details of this in here, mostly because the docs have a good [overview article](https://doc.cuba-platform.com/manual-6.1/own_components.html) and details examples for all of them: [Javascript Component](https://doc.cuba-platform.com/manual-6.1/js_library_sample.html), [Vaadin Addon](https://doc.cuba-platform.com/manual-6.1/vaadin_addon_sample.html) and the [GWT Component](https://doc.cuba-platform.com/manual-6.1/gwt_component_sample.html). But to give you a brief impression of what is possible, here is the JS example from the docs:

<figure class="center">
	<img src="https://doc.cuba-platform.com/manual-6.1/img/ui_component/product_edit.png" alt="">
	<figcaption>Example of a custom Javascript Component (Jquery UI Slider)</figcaption>
</figure>

## Generate models from existing databases

The next thing that got my attention is the ability to create an application from an existing database. This is a very interesting approach when you already have a application up and running with a "reasonably normal" database structure. The existing application can be a Java application but it clearly does not have to be one.

Studio will guide you through a wizard that will read the structure from the database you configured in the *project properties*. During the process, you can adjust the mapping from the database to the generated models.

To get an idea about the model generation wizard, you can try the following:

1. Download HSQLDB sample database from [here](http://hsqldb.org/doc/verbatim/sample/sampledata.sql)
2. Create a new project from studio
3. Run the server so that the automcatically defined HSQLDB will be started
4. Connect to the database with a generic database tool like the embedded *DatabaseManagerSwing* from HSQLDB or the IntelliJ IDEA Database Connection tool
5. Execute the downloaded SQL script on the database
6. Start the Model Generation from Studio (Entities > Generate Model) 

After doing so, you will see something similar to the image below. The wizard has a few steps to adjust the mapping, generate screens and finally see and adjust the database migration scripts.

<figure class="center">
	<a href="{{ site.url }}/images/cuba-6-1/generate-model-from-db.png"><img src="{{ site.url }}/images/cuba-6-1/generate-model-from-db.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-6-1/generate-model-from-db.png" title="Model Generation from existing database structure">Model Generation from existing database structure</a></figcaption>
</figure>

## Deploy to cloud services from studio

A pretty cool thing is the ability to directly deploy CUBA applications to cloud services.

When you have a look at *cloud provider settings*, you will see some basic information to login. As I looked throuh the provider picker, I surprisingly didn't know any provider from the list. It turns out that the list is a list of implementation providers for [Jelastic Cloud](https://jelastic.com/) where you can select different providers that all run Jelastic in their datacenter.

Jelastic is probably not the most obvious choice when the label is "Cloud deployment" in the market of big players like AWS, Azure or Google. But I think it is defenitly a step in the right direction and I personally will give Jelastic a chance. That said, there are a lot of possibilities in the cloud path for CUBA Studio and i'm looking forward on what will come next regarding cloud integration.


## API for entity graphs export/import

In order to import or export data it is now possible to use JSON files as the way to go. The <code><a href="https://github.com/cuba-platform/cuba/blob/579360fe491f2c5f14ad9c51a9de78ba063d31b6/modules/global/src/com/haulmont/cuba/core/app/importexport/EntityImportExportService.java">EntityImportExportService</a></code> interface allows you to do exactly this.

Also you can find a UI for this feature in a number of screens, for example the *Entity inspector* screen, where you can import or export arbitrary data right out of the box.

If you would like to embed the same feature in the workflow or screens you created by yourself, you can have a look at <code><a href="https://github.com/cuba-platform/cuba/blob/579360fe491f2c5f14ad9c51a9de78ba063d31b6/modules/gui/src/com/haulmont/cuba/gui/app/security/group/browse/GroupBrowser.java">GroupBrowser</a></code> for a running example.

## Row-level security constraints updates

The last thing that is quite an important change in the new release is row-level security. There has been row-level security before (as i described in the [cuba-security blog post](https://www.road-to-cuba-and-beyond.com/cuba-security-subsystem-distilled/), but with the new release it has become much more comprehensive.

In the [first](https://www.road-to-cuba-and-beyond.com/my-personal-crud-story-or-how-i-came-to-cuba/) blog post, where I talked about the security aspects of the system, one of my examples was:

> Managers in TX (Headquarter) see all customers from all locations, but cannot edit them

It turns out, that actually at this point in time CUBA was not really capable of doing it. With the new possibility of creating constraints with groovy, this exact feature becomes possible to achieve. I'll give you an example of this in the next security blog post, so stay tuned.

To wrap this up, I think there a few very cool features packed into this release. The license change is probably the most important one.

So, I encourage you to play with the changes. Just go to the [download](https://www.cuba-platform.com/download) section  and start from there.
