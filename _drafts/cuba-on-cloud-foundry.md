---
layout: post
title: Platform on Platform - CUBA apps on Cloud Foundry
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

### Everything as a Service
But running Docker is just one possibility when you want to embrace the Cloud mindset. You can create a Amazon EC2 instance, install Docker by yourself. This would be a IaaS approach. In this case, you have to care about different stuff like orchestration, auto scaling, fault tolerance. Amazon also has a more managed approach like [EC2 Container Service](https://aws.amazon.com/documentation/ecs/) and Docker itself tries to open up this market with solutions like [Docker Datacenter](https://www.docker.com/products/docker-datacenter). These alternatives are not really IaaS anymore, but more *Container as a Service (CaaS)*.

CaaS blurs the lines between classic IaaS and the topic i want to talk about in this article: *Platform as a Service (PaaS)*. 

### Abstracting away your Infrastructure

As you probalby have noticed, it's a little hard to distinguish between the categories, because the lines between them start to blur more and more. To clearify this a little bit, here is a categoriziation of the *\[I\|P\|S\]aaS world*.

<figure class="center">
	<a href="{{ site.url }}/images/cloud-foundry/iaas-paas-saas.png"><img src="{{ site.url }}/images/cloud-foundry/iaas-paas-saas.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cloud-foundry/iaas-paas-saas.png" title="What you / they have to manage in the different migration from self hosting to cloud">What you / they have to manage in the different migration from self hosting to cloud</a></figcaption>
</figure>

What we want to talk about in this article about the third category: Platform as a Service. The differences between what we have seen in the Docker world which is quite similar to IaaS that we have to care about the operating system. We know about and have to care about stuff like filesystem structures, drivers for certain databases, copy files and set system environment variables (see the Dockerfile for an example).


{% highlight dockerfile %}

### Dockerfile

FROM tomcat:8-jre8
ADD war/app.war /usr/local/tomcat/webapps/
ENV CATALINA_OPTS="-Dlogback.configurationFile=/opt/cuba_home/logback.xml"
# ...

{% endhighlight %}


So when comparing it to a PaaS, you don't have to care about this stuff anymore. Which implies, that you also can't care about this stuff as well. The only thing you have to tell is the runtime that you want to use for your application. This is another abstraction layer, that the developer acts upon.