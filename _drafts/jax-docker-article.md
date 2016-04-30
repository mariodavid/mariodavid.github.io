---
layout: post
title: Docker for the rest of us
possible-titles:
  - Docker for the rest of us
  - 
description: "In this article "
---

Docker is omnipresent in the media and the tech industriy for the last few years. Just recently ThoughtWorks encourages to *adopt* Docker in it's [technology radar](https://www.thoughtworks.com/radar/platforms/docker). This is due to very good reasons. But when hearing about it, one could assume that this piece of technology is mostly relevant for environments with *Microservices*, *Continous Delivery* or *DevOps* inplace. This article will try to show that this is a false assumption. The use cases for Docker are much broader and even in a non-cutting edge environment it can be either benefitial for Development or Operations or both.

In this article a Docker installation with a fairly classical monolithic Java Web application in a Tomcat Server will be created. Additionally a relational database will be used as the datastore. Before going into the details, there will be a high-level overview on the benefits and the idea behind Docker.

<!-- more -->

## The docker 10,000 feet overview

Docker is a tool that arised about 2013. Since that time it pretty much whirls the industry of IT. It is a way to isolate your server applications. Each application defines its own environment without interfering with other applications. From an isolation point of view it is similar to hypervisor based virtual machine (like VMWare). 

But from a performance angle the Docker approach is quite different. In a hypervisor based virtual machine model the hardware is emulated with an abstraction layer. With Docker this is not the case. Instead the different containers share the Kernel of the operating system of the host. This implies that no hardware needs to be emulated which means that the processes inside of a container run at native speed.

This performance improvement drastically increases the packing density on a per server-hardware basis, because of the discontinoued existence of an operating system per virtual server and therefore cuts the operational costs.

Sharing the Kernel of the underlying operating system has some downsides as well. Currently Docker is more or less related to Linux as the Host OS, although Microsoft is pushing really hard to achieve a really great user experience with Docker on Windows in different shapes like integration into Windows Server 2016 e.g. 

The Docker approach, which is called paravirtualization is not virtualization in the sense most people are familiar with. It is more a way of segmenting your operating system. Predecessors of Docker like LXC or Solaris Containers were build on top of the same ideas and so it is fair to say that the idea is not new. In fact, where hypervisor based virtualization at least in the x86 world had their beginnings at the end of 1990's, the beginnings of paravirtualization started in the 1970's with IBM's VM/370 on the mainframe.

So, the main benefits of Docker in comparison to a hypervisor based virtual machine are lower overhead, faster execution and potential immutability.

## The benefits of running an application in a container

There are a few worth mentioning benefits for using container technologies as a way to distribute the application.

First of, to get a running "production like" environment to test on becomes a once-off effort. This is due to the basic principle **[infrastructure as code](https://www.thoughtworks.com/de/insights/blog/infrastructure-code-reason-smile)** which Docker fundamentally relies on. In this case the installation or configuration of a server is described in a file. Due to this infrastructure can be created automatically and repeatably. Additionally priciples of software engineering can be applied to the world of operations.

Defining the initial state of a server installation has a lot of benefits. It is simply not required any more to install the required software on the system as well as the operation system itself. In fact, in some cases it will be not only seen as a effort saver but not doing it as an [anti-pattern](http://martinfowler.com/bliki/ConfigurationSynchronization.html).

The [Dockerfile](https://docs.docker.com/engine/reference/builder/) is the representation of this principle in the Docker ecosystem. The effort to define a server installation with the required software with these files is pretty low. The reason for this lies in a technical detail: the union filesystem. It allows Docker to define a Docker Image as a set of layers. Due to this, when creating Dockerfile it will be normally based on an existing Docker image (another layer). 

These images can be found on [Docker Hub](https://hub.docker.com/). It is a similar offering for infrastructure code what GitHub is for application code. On Docker Hub there are plenty of predefined images for most popular open source software. From infrastcure images like [Ubuntu](https://hub.docker.com/_/ubuntu/) or [Busybox](https://hub.docker.com/_/busybox/) to applications like [Wordpress](https://hub.docker.com/_/wordpress/), [Jenkins](https://hub.docker.com/_/jenkins/) or [MongoDB](https://hub.docker.com/_/mongo/) can all be found on Docker Hub.

The next benefit is, that creation these defined servers or environments is very fast. For a test environment it is possible to create a something like a HAProxy in front of two tomcat instances, all backed up by a postgres cluster and a redis key-value store as a caching layer. 

These different servers will be up and running in a matter of seconds and with one line command. To remove this environment its the exact same effort and you are back in a clean state of your development environment without having to maintain different Java versions or Postgres database installations on your OS installation directly by yourself.

The environment i described above is not only fast to create but it is also reproducable. It is so valuable to create a situation where **development environment = production environment**, because a whole category of problems in software development disappear when no one is able to say "it works on my machine" anymore. With tools like Docker and again, *infrastructure as code* this goal is at least possible.

There are some additional advantages that wasn't covered here. In this [DZone article](https://dzone.com/articles/5-key-benefits-docker-ci) it goes a little more into depth. 

## Create a Container for a java web application

Before creating the container here is the desired outcome of the application architecture:

The developed web application is a [order management solution](https://github.com/mariodavid/cuba-ordermanagement). It is based on [CUBA platform](https://www.cuba-platform.com/), a Java application platform for creating business applications. The application will be hosted inside of a Tomcat servlet container. The data will be stored in a PostgreSQL RDBMS. 

At the end both servers will be running in a seperate container.

The first container will be the tomcat instace together with our application. The basis of the container is the official [Apache Tomcat image](https://hub.docker.com/_/tomcat/).

To get going, Docker has to be [installed](https://docs.docker.com/engine/installation/). Running Docker on Linux is not necessary required, since Docker runs on Windows or Mac as well via [Docker Machine](https://docs.docker.com/machine/).

As a first impression an emtpy tomcat can be created via the following command:

{% highlight bash %}

$ docker run -it --rm tomcat:8-jre8 bash

{% endhighlight %}


The first time the image will be downloaded and cached on the computer. After the download is finished, there is a bash prompt (because <code>bash</code> was explicitly set as the command) where a look at the newly created docker container can be taken. Browsing the directory structure, creating and changing config files is possible. After typing <code>exit</code> the bash process will do so and due to the option <code>--rm</code>, the container will be destroyed with all the data in it. 

Executing the same command line once again, a new container will be created (this will just take a few seconds, because the image is already cached). All the changes that were made last time will be gone. This is the default operation mode of a container. If something has to be changed in the container, the Dockerfile is the place for changing the configuration, not the running instance of a container. 


### The Dockerfile of the tomcat ordermanagement application

The [Dockerfile](https://raw.githubusercontent.com/mariodavid/cuba-ordermanagement/master/docker-image/Dockerfile) contains of a few instructions about the creation of the container. The first instruction is the name of the base image the image will be based upon. Like above <code>tomcat:8-jre8</code> is used.

{% highlight dockerfile %}
### Dockerfile

# Base Image: official tomcat 8 image, with jre 8 underneath
FROM tomcat:8-jre8

# Add tomcat users file for manager app
ADD container-files/tomcat-users.xml /usr/local/tomcat/conf/

# Add context config file for the application
ADD container-files/context.xml /usr/local/tomcat/conf/

# Add generated app war files to the webapps directory
ADD war/app.war /usr/local/tomcat/webapps/
ADD war/app-core.war /usr/local/tomcat/webapps/

# copy logback.xml config in the container
ADD container-files/logback.xml /opt/cuba_home/

# set CATALINA_OPTS env variable to point to logback config file and app home directory
ENV CATALINA_OPTS="-Dlogback.configurationFile=/opt/cuba_home/logback.xml -Dapp.home=/opt/cuba_home"

# Add postgres jdbc driver lib
ADD https://jdbc.postgresql.org/download/postgresql-9.3-1101.jdbc41.jar /usr/local/tomcat/lib/postgresql.jar

{% endhighlight %}

The *ADD* instruction is used to copy files from outside the container into it at build time. Configuration files will be copied into the container that were created in the [container-files](https://github.com/mariodavid/cuba-ordermanagement/tree/master/docker-image/container-files) directory. Due to the layered filesystem, the layer that this Dockerfile creates overrides the existing files of the base image.

Then the actual application war files are copied into the container. 

The *ENV* instruction will create operating system environment variables that tomcat will uses as configuration 

### Build the described image and start the first container

In the application structure the sub directory [docker-image](https://github.com/mariodavid/cuba-ordermanagement/tree/master/docker-image)  contains the Dockerfile as well as the files that are beeing added. To build the Docker image from the Dockerfile, the following steps have to be executed:

{% highlight bash %}

git clone https://github.com/mariodavid/cuba-ordermanagement && cd cuba-ordermanagement

./gradlew buildWar

cp build/distribution/war/*.war docker-image/war/

docker build -t cuba-ordermanagement docker-image/

{% endhighlight %}

First, the repository with the application is cloned and the war files are built via gradle. Next the war files are copied into the docker-image directory. After this the actual docker container will be created from the Dockerfile above. Additionally the docker image is called "cuba-ordermanagement" with the option <code>-t</code>, so it is possible to be referenced later.

With this the build step of the docker image is complete. Now it is possible to create a running instance of this image with the following command:

{% highlight bash %}

docker run -it --rm -p 8080:8080 cuba-ordermanagement

{% endhighlight %}

<code>-p</code> is the option for port mappings. Docker by default does not expose any ports on the host that is running docker. This is why the port mapping for this container is activated. Its essentially the exact same idea as [NAT](https://en.wikipedia.org/wiki/Network_address_translation). The syntax is <code>-p outer-port:inner-port</code>.

After executing this command, the tomcat server will start and chrash  with a <code>UnknownHostException</code>. This happens due to the database connection settings that have been populated in the [context.xml](https://github.com/mariodavid/cuba-ordermanagement/blob/master/docker-image/container-files/context.xml) from the repository. This file is used in the Dockerfile and it requires the host <code>postgres</code> to be available. 

Normaly this parameters will be externalized via environment variables e.g. But for this demonstration it is acceptable to do, in particular because this allows to go through another tool in the docker ecosystem: [docker compose](https://docs.docker.com/compose/).

## Orchestrate your containers with docker compose

Docker compose is a tool to create containers and orchestrate them. When different features of docker like *Volume mounting* or *container linking* are used, the corresponding command line options become long and tedious. Additionally these settings are not persisted in any way. Docker compose instead uses a configuration file called *docker-compose.yml* that describes the different containers, their configuration options and the container relationships.

The ordermanagement application repository contains a [docker-compose.yml](https://github.com/mariodavid/cuba-ordermanagement/blob/master/docker-image/docker-compose.yml) file that describes the start of the tomcat application server as well as a PostgreSQL database container:


<div style="color:red">Docker Compose Example auf docker network umbauen</div>

<div style="color:red">Überarbeiten START</div>

{% highlight yaml %}
version: '2'

services:
  web:
    image: cuba-ordermanagement
    ports:
     - "8080:8080"
    networks:
      - ordermanagement-network
  postgres:
    image: postgres
    networks:
      - ordermanagement-network

networks:
  - ordermanagement-network

{% endhighlight %}

The <code>web</code> container describes the actual tomcat instance. It defines the port mapping and the network it should be placed into. This <code>postgres</code> container is another official Docker container (see [here](https://hub.docker.com/_/postgres/)). It runs a standard postgres database with default settings and is in the <code>ordermanagement-network</code> available as well.

When both containers are placed into the same network, the containers are able to connect to each other. Name resolution is done via DNS within the network through Docker. The network setting allows the tomcat instace to open the required JDBC connection to the PostgreSQL database.


<div style="color:red">Überarbeiten ENDE</div>

To run this, <code>cd</code> into the directory of the <code>docker-compose.yml</code> file and run:


{% highlight bash %}

docker-compose up

{% endhighlight %}

After this both containers are starting up and after a few seconds the application is ready to be used via <code>http://localhost:8080/app</code>.


<figure class="center">
	<img src="{{ site.url }}/images/2015-12-31-put-your-island-into-a-box/cuba-start.png" alt="">
	<figcaption>The running CUBA application as a docker container</figcaption>
</figure>


## Wrapping up
... (max three paragraphs)
