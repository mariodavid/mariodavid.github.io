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
So let's get started with Cloud Foundry. To do so, i have created a free acc. at [Pivotal Web Services](https://run.pivotal.io/), the Pivotal CF Cloud offering, which let's me play with CF for 2 month for free. Selecting PWS for CF is just out of convenience reasons, so if you want to try it in your own datacenter (via OpenStack or VMWare vShpere) or on AWS EC2 instances, feel free to do so.

After creating the acc. and installing their [CLI](http://docs.run.pivotal.io/cf-cli/) you are ready to login via the command line:

{% highlight bash %}
cf login -a https://api.run.pivotal.io
{% endhighlight %}

After doing so your command line is ready to go to push your application. But first, let's have a look at the web UI which allows you to view the running instances. 


<a href="{{ site.url }}/images/cloud-foundry/pws-ui.png"><img style="width: 50%; float:right" src="{{site.url}}/images/cloud-foundry/pws-ui.png"></a>

After a successful [login](https://login.run.pivotal.io/login) you will see a screen similar to this.

First, i created an organisation called "cuba-ordermanagement". This oragnisation contains one *space*. A *space* is an aggregation of services and apps that are scoped via project or an environment. I created a space called "development", which should describe the phase of my imaginary continous delivery pipeline pipeline. 

As you see on the right, i already started my apps and one service, so let's get started with deploying an app on CF.


## Make CUBA Cloud-ready &trade;

There are a few things that have to be changed from the traditional deployment model. We'll go through it step by step so you should be able adjust your app right as we go.

### Create a postgres database in Cloud Foundry

First of all, we have to create a datastore for our cloud version of the cuba app. In *IaaS world* you would set up a vanilla postgres instance, figure out connection settings and tell your app where to find your datastore.

In Cloud Foundry your database is *just another* service, that the application can depend upon. The Pivotal offering of Cloud Foundry has a couple of services available for stuff like caching, search, (non-) relational datastores, monitoring tools and so on. 

When you want to take a look at what are the different offerings, just use the marketplace cmd option.

{% highlight bash %}
$ cf marketplace
{% endhighlight %}

We will choose a postgres installation for now. You can either use the web UI or the command line interface:

{% highlight bash %}
$ cf create-service elephantsql turtle cuba-ordermanagement-postgres
{% endhighlight %}

*elephantsql* is the name for the service (Postgres as a Service) and *turtle* the service plan (turtle is the free one). *cuba-ordermanagement-postgres* is the name of the service instance, which we'll need for later reference.

Now we have a running postgres db installation. Next thing is that we have to configure our application to use this datastore.


### Springs Cloud Connector for DataSource creation

Within the spring ecosystem there are different packages that handle integration with cloud systems in general and PaaS like Cloud Foundry in particular. In the <code>build.gradle</code>, the <code>coreModul</code> needs the following two additional dependencies:

{% highlight groovy %}

configure(coreModule) {
	//...
    dependencies {
        //...
        compile('org.springframework.cloud:spring-cloud-spring-service-connector:1.2.1.RELEASE')
        compile('org.springframework.cloud:spring-cloud-cloudfoundry-connector:1.2.1.RELEASE')
    }
    //...
}
{% endhighlight %}

Next, since we want the the database to be looked up from the cloud, we need to override the creation of the <code>dataSource</code> bean. This can be done via dependency injection. In the <code>spring.xml</code> of the <code>core</code> module we tell Spring to use a Factory bean for the creation of the <code>cubaDataSource</code> bean, which is used throughout the CUBA application. 


{% highlight xml %}

<bean id="cloudFoundryDataSourceFactory" class="com.company.ordermanagement.cloud.CloudFoundryDataSourceFactory">
    <property name="dbServiceName" value="cuba-ordermanagement-postgres" />
</bean>

<bean id="cubaDataSource" factory-bean="cloudFoundryDataSourceFactory"
      factory-method="createDataSourceForPostgresDbService">
</bean>
{% endhighlight %}

The <code>cubaDataSource</code> bean will now be created via the factory method <code>cloudFoundryDataSourceFactory.createDataSourceForPostgresDbService()</code>. The value of <code>dbService</code> attribute of the factory is the service name of the postgres instance we created earlier.

Since this Factory class does not exist yet, let have a look at the [implementation](https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/modules/core/src/com/company/ordermanagement/cloud/CloudFoundryDataSourceFactory.groovy):


{% highlight groovy %}
//...

class CloudFoundryDataSourceFactory {

    String dbServiceName

    DataSource createDataSourceForPostgresDbService() {

        Cloud cf = new CloudFactory().cloud
        def postgresSerciveInfo = cf.getServiceInfo(dbServiceName)
        cf.getServiceConnector(postgresSerciveInfo.id, DataSource, null);

    }
}

{% endhighlight %}

With this little glue code inplace, the application is ready to connect to the database via a service name instead of DNS names through the cloud platform.

### Create deployment information for Cloud Foundry

Since we can't really get down to the underlying infrastructure of the app, we are not able to change the way the tomcat (or whatever servlet container is underneath your app in this case) works. But certain metadata has to be given to the PaaS in order to run our application properly, e.g. the java version, the amount of memory required, external services (like datastores) and so on.

For these kind of information, CF required a file called *manifest.yml* to be in place. This file is the entry point for the deployment.

So the file that we are going to create looks pretty much like [this](https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/manifest.yml):

{% highlight yaml %}
---
applications:
- name: cuba-ordermanagement
memory: 1024M
instances: 1
host: cuba-ordermanagement
path: build/distributions/war/app.war
services:
- cuba-ordermanagement-postgres
env:
  JBP_CONFIG_OPEN_JDK_JRE: '{jre: { version: 1.8.0_+ }}'
{% endhighlight %}











