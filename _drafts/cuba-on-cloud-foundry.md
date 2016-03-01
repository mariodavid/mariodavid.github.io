---
layout: post
title: Platform to Platform - CUBA apps on Cloud Foundry
description: "..."
modified: 2016-03-05
tags: [cuba, PaaS, Cloud Foundry]
image:
  feature: cloud-foundry/feature.jpg
  feature_source: https://pixabay.com/de/funkturm-berlin-nacht-geb%C3%A4ude-490032/
  
---

In this blog post i will [continue](http://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/) to take a look at the operations side of things and talk about running the CUBA Platform on a PaaS that gains much momentum these days: [Cloud Foundry](https://www.cloudfoundry.org/).

<!-- more -->

In the [last](http://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/) blog post about Docker i wrote about how to use container technologies. Docker is a great technologie that enables the Road to *infrastructure as code*. 

But running Docker is just one possibility when you want to embrace the Cloud mindset. You can create a Amazon EC2 instance, install Docker by yourself. This would be a IaaS approach. In this case, you have to care about different stuff like orchestration, auto scaling, fault tolerance. Docker has different tools for this and constantly trying to open up this market with solutions like [Docker Datacenter](https://www.docker.com/products/docker-datacenter). Nowadays this is called something like "Container as a Service (CaaS)". 

CaaS blurs the lines between classic IaaS and the topic i want to talk about in this article: "Platform as a Service (PaaS)". 






