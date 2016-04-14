---
layout: post
title: Put your island into a box - how to dockerize your CUBA app
description: "In this blog post i will talk about how to dockerize your CUBA application and the benefits behind isolating your applications with containers."
modified: 2015-12-15
tags: [cuba, docker, tomcat, docker compose]
image:
  feature: 2015-12-31-put-your-island-into-a-box/feature.jpg
  feature_source: https://pixabay.com/de/t%C3%BCr-container-boot-container-1088459/
---

Since docker is really omnipresent in the last few month for very good reasons i want to pick this up for this blog post. I will show how to dockerize a [CUBA](https://www.cuba-platform.com/) app and run it as a container together with the corresponding database. If you are not familiar with the basics of Docker, i additionally will scratch the surface of this technologie a little bit for you.

<!-- more -->


<img style="float:right; padding: 10px; margin-right:-150px;" src="{{site.url}}/images/2015-12-31-put-your-island-into-a-box/container.png">

## The docker 10,000 feet overview

Docker itself is a tool that arised about 2013. Since that time it pretty much whirls the industry of IT. It is a way to isolate your applications in a way, that allows each application to define their own environment without interfering other applications. Just think of it like virtual machines that you might know from VMWare. But instead of emulating the whole hardware with an abstraction layer, this emulation layer goes away. 

This brings some fundamental speed improvements as well as some restrictions. You can create a (Docker-)Container within a few seconds instead of an hypervisor based VM that will launch within a few minutes. On the other hand, you can currently only create Docker containers on a host maschine that's running Linux, because the containers share the kernel. Actually it's not virtualization at all, it's more a clever way of namespacing your OS. Additionally the idea is not new. Where hypervisor based virtualization at least in the x86 world are about 15 years old, the beginnings of paravirtualization (which docker is) started their journey back in the mainframe world.

Since docker is a pretty famour topic for blog posts and tutorials these days, i'll  reference some good resources to look at, if you're not familiar with the concepts. 

First you can start of at the [What is docker?](https://www.docker.com/what-docker) page which will give you an elevator pitch of containers and the differentiation to hypervisor based virtualization. Next up there is the offical [docker docs](https://docs.docker.com/), which is pretty great comparing to other offical documentations. It starts with an installation guide and then goes on with a [quickstart on containers](https://docs.docker.com/engine/userguide/basics/). 

When you want to dive really deep while still getting the 10,000 feet overview i encourage you to check out [Nigel Poulton's](https://twitter.com/nigelpoulton) pluralsight course called "[Docker deep dive](https://www.pluralsight.com/courses/docker-deep-dive)", which starts with a very good overview on container technologie space before discussing the different technical sub-topics of docker.


## The benefits of putting your CUBA app into a container

Before showing the different steps to create a running CUBA container i want to emphasize the benefits in doing that. Actually it has nothing to do with CUBA in particular, but nevertheless its worth mentioning.

First of, to get a running "production like" environment to test on becomes a O(1) alike effort. Why? This is due to the basic principle **infrastructure as code** which is described in depth in this [ThoughtWorks article](https://www.thoughtworks.com/de/insights/blog/infrastructure-code-reason-smile). Scripting the creation of a server installation like you do with a [Dockerfile](https://docs.docker.com/engine/reference/builder/) is a one time effort. This effort is even very little, because in case of docker there is this thing called [Docker Hub](https://hub.docker.com/) which let users share their work in infrastructure scripting like Github does for code.

The next thing is, that it's **lightning fast**. With a one liner in the shell you can start up a HAProxy in front of two tomcat instances, all backed up by a postgres cluster and a redis key-value store. This all will be up and running in a matter of seconds. If you want to remove this test environment its the exact same effort and you are back in a clean state of your development box without having to maintain different Java versions or Postgres database installations on your OS installation directly by yourself.

The environment i described above is not only fast to create but it is also reproducable. It is so valuable to create a situation where **development environment = production environment**, because a whole category of problems in software development go away. With tools like Docker and again, *infrastructure as code* this goal is at least possible.

There are some additional advantages i didn't cover in this [DZone article](https://dzone.com/articles/5-key-benefits-docker-ci). So lets get our hands dirty and create a docker image that will contain our app.

## Create a Container for our CUBA app

Docker's definition of a container is done via a file called Dockerfile. In this file you normally start with a base image. These images are often stored in [Docker Hub](https://hub.docker.com/). There are plenty of images for operating systems as well as images for application servers, databases and applications. In this case we'll create an image on the basis of the official [Apache Tomcat image](https://hub.docker.com/_/tomcat/). 

If you wonder what all of this actually means, you can give it a try. 

<div class="well">I assume you are on Linux here. If not, check out <a href="https://docs.docker.com/machine/">docker-machine</a>.</div>

Given you have Docker installed [correctly](https://docs.docker.com/engine/installation/), you run an instance of tomcat with the following command:

{% highlight bash %}

docker run -it --rm tomcat:8-jre8 bash

{% endhighlight %}




The first time the tomcat image will be downloaded and cached on your computer. After this you have a bash were you can look at the newly created docker container. Browse the directory structure, create and change config files, look and the tomcat installation and so on. After typing <code>exit</code> the bash process will do so and due to the option <code>--rm</code>, the container will be destroyed with all the data in it. 

Executing the same command line once again, a brand new container will be created (this will instead just take a few seconds, because the image is already downloaded). All the changes you made last time will be gone. If you think this is crazy, well this is how containers normally work. If you want to change something, don't change it in the container itself, but in the Dockerfile that creates it. Otherwise you are back in the old days with [configuration drift](http://kief.com/configuration-drift.html), !(reproduceable environments) and !(insfrastructure as code).


### Docker container vs image
If you wonder why i sometimes talk about *image* and sometimes about *container*, here's a short description:

<p class="well">Containers are the running instances of images. The image is the binary like the war file (or exe) and the container is the running process of this binary.</p>


After this brief introduction in image inheritance and container behavior, lets have a look at the dockerfile that decribes our app.

### The Dockerfile of our ordermanagement app

The [example project](https://github.com/mariodavid/cuba-ordermanagement) that i want to dockerize is the ordermanagement app that i created in previous blog posts. Since it is a CUBA 6 app, it will use JRE8 together with Tomcat 8. Lets have a look at the [Dockerfile](https://raw.githubusercontent.com/mariodavid/cuba-ordermanagement/master/docker-image/Dockerfile), that describes our image:

{% highlight dockerfile %}
### Dockerfile

# Base Image: official tomcat 8 image, with jre 8 underneath
FROM tomcat:8-jre8

# Add tomcat users file for manager app
ADD container-files/tomcat-users.xml /usr/local/tomcat/conf/

# Add generated app war files to the webapps directory
ADD war/app.war /usr/local/tomcat/webapps/
ADD war/app-core.war /usr/local/tomcat/webapps/

# Add context config file for the application
ADD container-files/context.xml /usr/local/tomcat/conf/

# copy logback.xml config in the container
ADD container-files/logback.xml /opt/cuba_home/

# set CATALINA_OPTS env variable to point to logback config file and app home directory
ENV CATALINA_OPTS="-Dlogback.configurationFile=/opt/cuba_home/logback.xml -Dapp.home=/opt/cuba_home"

# Add postgres jdbc driver lib
ADD https://jdbc.postgresql.org/download/postgresql-9.3-1101.jdbc41.jar /usr/local/tomcat/lib/postgresql.jar

{% endhighlight %}

First off, you see that the image will be based on the Tomcat 8 image. With *ADD* you can copy files from outside the container into it at build time. In this case, we want to override some configuration files for the tomcat. Next, we add the war files into the webapps directory of the tomcat.

With *ENV* we will create operating system environment variables that tomcat will pick up. In this case we will just point the tomcat to our home directory and the logback config file. 

Since we want to connect to a Postgres DB, we'll add the postgres drivers into the lib directory of the tomcat installation.

### Build the described image and start the first container

In the example app, there is the subfolder [docker-image](https://github.com/mariodavid/cuba-ordermanagement/tree/master/docker-image) that contains the Dockerfile as well as the files that are beeing added. If you want to try it out by yourself follow these steps:

{% highlight bash %}

git clone https://github.com/mariodavid/cuba-ordermanagement && cd cuba-ordermanagement

./gradlew buildWar

cp build/distribution/war/*.war docker-image/war/

docker build -t cuba-ordermanagement docker-image/

{% endhighlight %}

First, clone the repo and build the war files via gradle. Next we copy the war files into the docker-image folder. After this we create the actual docker container with the Dockerfile above. Additionally we call the image "cuba-ordermanagement" with the option <code>-t</code>, so we can reference it later with docker.

That's basically it. Now we can create a running instance of this image with the following command:

{% highlight bash %}

docker run -it --rm -p 8080:8080 cuba-ordermanagement

{% endhighlight %}

<code>-p</code> is the option for port mapping. Since we want to connect into the container, we have to tell the docker host (in this case our local computer), that it should map a certain port on our host to the container. Its essentially the exact same idea as [NAT](https://en.wikipedia.org/wiki/Network_address_translation). The syntax is <code>-p outer-port:inner-port</code>.

After executing this command, the tomcat will start and chrash wonderfully with a <code>UnknownHostException</code>. Why is this the case? Well, i decided to bake the database connection settings into the image binary. When looking at the [context.xml](https://github.com/mariodavid/cuba-ordermanagement/blob/master/docker-image/container-files/context.xml) from the repo which is used in the Dockerfile, it requires the host <code>postgres</code> to be available. It might think a little strange at first sight to not externalize this parameter. But with this i can show you another tool in the docker ecosystem. It's called [docker compose](https://docs.docker.com/compose/).

## Orchestrate your containers with docker compose

Docker compose is a tool to create containers and orchestrate them. When you use different features of docker like *Volume mounting* or *container linking* the corresponding command line options become a little tedious. Additionally these settings are not persisted in any way. Docker compose instead uses a configuration file called *docker-compose.yml* that describes the different containers and their relationships.

I created a [docker-compose.yml](https://github.com/mariodavid/cuba-ordermanagement/blob/master/docker-image/docker-compose.yml) file that describes the start of our app as well as a postgres database container:


{% highlight yaml %}

web:
  image: cuba-ordermanagement
  ports:
   - "8080:8080"
  links:
   - postgres:postgres

postgres:
  image: postgres

{% endhighlight %}

The <code>web</code> container describes the actual tomcat instance. It defines the port mapping and an additional link to another container. This <code>postgres</code> container is another official Docker container (see [here](https://hub.docker.com/_/postgres/)). It runs a standard postgres database with default settings.

The linking on the web container does a few things. But most importantly it creates an entry in the <code>/etc/hosts</code> file with the ip address of the container and the hostname <code>postgres</code>.

To run this, <code>cd</code> into the directory of the <code>docker-compose.yml</code> file and run:


{% highlight bash %}

docker-compose up

{% endhighlight %}

After this you should be able to fire up your app at <code>http://localhost:8080/app</code>


<figure class="center">
	<img src="{{ site.url }}/images/2015-12-31-put-your-island-into-a-box/cuba-start.png" alt="">
	<figcaption>The running CUBA app as a docker container</figcaption>
</figure>


I hope you get a little insight why docker can be valuable for you. If you have any questions or notes on this tutorial, please leave a comment below.

If you look at what i just presented to you, what we did was basically a lot of plumbing. If this blog post had been written at the end of 2013 everthing would be fine. But since its already 2016 and the world of infrastructure automation is moving so fast, a lot of tools around docker or related to this technologie have arisen. Orchestration mechanisms like [docker swarm](https://docs.docker.com/swarm/), [mesosphere](https://mesosphere.com/) or [Kubernetes](http://kubernetes.io/) that are like docker compose with more production environments in mind.

Since the PaaS technology [Cloud Foundry](http://cloudfoundry.org/) got a lot of attraction in the enterprise world in the last few years and was [mentioned](https://www.cuba-platform.com/blog/2015-10-07/446) as a new feature in the CUBA 6 release as beeing supported, i will tackle this topic in another blog post.


<img style="float:left; padding: 10px; margin-left:-150px;" src="{{site.url}}/images/2015-12-31-put-your-island-into-a-box/container.png">



