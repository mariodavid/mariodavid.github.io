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

As you probalby have noticed, it's a little hard to distinguish between the categories, because the lines between them start to blur more and more. To clearify this a little bit, here are two ways of describing the differences in the *\[I\|P\|S\]aaS world*.

<figure class="center">
	<img src="{{ site.url }}/images/cloud-foundry/iaas-paas-saas.png" alt="">
</figure>


<div style="margin: auto auto 25% 25%; width: 50%">
<blockquote class="twitter-tweet" data-lang="de"><p lang="en" dir="ltr">The entire PaaS vs Container (e.g. Docker) debate explained in one simple diagram - <a href="https://t.co/B0oib5gihz">pic.twitter.com/B0oib5gihz</a></p>&mdash; swardley (@swardley) <a href="https://twitter.com/swardley/status/663089099989889024">7. November 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>  
</div>

<div style="margin-top:-175px">&nbsp;</div>

What we want to talk about in this article about the third category: Platform as a Service. The differences between what we have seen in the Docker world which is quite similar to IaaS that we have to care about the operating system. 

The application developer knows about and have to care about stuff like filesystem structures, drivers for certain databases, copy files and set system environment variables. (see the Dockerfile for an example).


{% highlight dockerfile %}

### Dockerfile

FROM tomcat:8-jre8
ADD war/app.war /usr/local/tomcat/webapps/
ENV CATALINA_OPTS="-Dlogback.configurationFile=/opt/cuba_home/logback.xml"
# ...

{% endhighlight %}


Additionally you have to care about monitoring, backup, fault-tolerance, orchestration and much more. If you do this with some kind of combination of bash scripts and docker commands, you probably sould consider this:
  
<div style="margin: auto auto 10% 10%; width: 75%">
    <blockquote class="twitter-tweet" data-lang="de"><p lang="en" dir="ltr">Sure, you can choose to build your own <a href="https://twitter.com/hashtag/PaaS?src=hash">#PaaS</a>.. Comparing build your own to a structured platform be like.. <a href="https://t.co/KBW9KINYHn">pic.twitter.com/KBW9KINYHn</a></p>&mdash; Dan Mearls (@DanMearls) <a href="https://twitter.com/DanMearls/status/657961157114875905">24. Oktober 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

</div>


<div style="margin-top:-75px">&nbsp;</div>

So when comparing it to a PaaS, you don't have to care about this stuff anymore. Which implies, that you also can't care about this stuff as well (just to keep in mind). The only thing you have to tell your environment is the runtime that you want to use for your application.

To get a sense of how that looks like, let's move to Cloud Foundry and try to deploy a CUBA app on this platform.

<img style="float:right; padding: 10px;" src="{{site.url}}/images/cloud-foundry/cloud.png">

### Cloud Foundry 

Cloud Foundry is an open-source PaaS solution backed up by a lot of big companies. Initially started at VMWare / Pivotal is has become a solution for different Cloud offerings like IBM Bluemix, HP helios and the Cloud solution from Pivotal itself.

When comparing it to other solutions like Heroku, the basic difference is that since CF is open source, you can run it in your own datacenter if you want to. Another possiblity is to the PaaS on AWS or any other IaaS provider on your own.

//....


### Get ready to go via Pivotal Web Services
So let's get started with Cloud Foundry. To do so, i have created a free acc. at [Pivotal Web Services](https://run.pivotal.io/), the Pivotal CF Cloud offering, which let's me play with CF for 2 month for free.

After creating the acc. and installing their [CLI](http://docs.run.pivotal.io/cf-cli/) you are ready to login via the command line like this:

{% highlight bash %}
cf login -a https://api.run.pivotal.io
{% endhighlight %}

