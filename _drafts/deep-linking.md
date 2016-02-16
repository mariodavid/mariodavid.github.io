---
layout: post
title: Deep Links for Resources
description: "A little hidden feature in CUBA platform is deep linking to resources"
modified: 2015-12-15
tags: [cuba]
image:
  feature: deep-link/feature.jpg
  feature_source: https://pixabay.com/de/treffpunkt-treffen-pfeile-zetrum-755795/
---

Here's a thing i stumbled upon lately in the CUBA Platform docs, that seems not to be very common but nevertheless very valuable: Deep Links to resources.

<!-- more -->

Deep linking in CUBA Platform seems not to be the most used feature, but i think it's a pretty important one, so i'll give you a hint about this feature.

When opening up a CUBA application in your browser, you'll end up with a static URL like <code>http://localhost:8080/app/#!</code>. It is *one* endpoint. In fact, it is *literally* and *end-point*, because it's kind of the end of the web. There is no possibility to get down further into the application via a link (at least, it is not shown in the URL of the browser). This is kind of sad and is based on the architecture that Vaadin is based upon, but also very common nowadays when looking into the whole *single page application land*.

CUBA Platform currently does not support URL changes via *HTML5 History API* out of the box, meaning that it will change the shown URL when navigating through the application. 


<img style="float:right; padding: 10px; margin-right:-50px;" src="{{site.url}}/images/deep-link/labyrinth.jpg">

## CUBA Platform understands deep links

Although CUBA is not updating the browsers url bar, it is in fact capabale of dealing with deep links. What i mean by that is, when you want to directly link to a certain customer entry, CUBA is able to handle that request and display e.g. the editor screen of the customer.

The part of the docs is called: [Screen Links](https://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#link_to_screen). With Screen Links you can address parts of the application, in particular a certain screen. An example of this would be:

<code>http://localhost:8080/app/open?screen=om$Customer.edit&item=om$Customer-*UUID*</code>

The parameters *screen* defines the required screen to show and the *item* parameter qualifies the desired customer entity. The docs have some other examples like additional parameters that will be passed into the <code>init()</code> method so the controller of the view is capable to handle them.


## Generate Links via the UI
Since hand-crafting these URLs seems not to be the best approach, we can create a mechanism in the application, which at least generates this url for us. Obviously it would be better if we could just copy the URL from the address bar, but since this is currently not possible, we can try to achieve something similar.

...

With this little generator in-place, we can easily create links that the user can copy and paste to directly talk about a specific resource with a co-worker. In this sense - i wish happy linking!