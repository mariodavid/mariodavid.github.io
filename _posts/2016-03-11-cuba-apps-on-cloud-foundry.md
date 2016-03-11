---
layout: post
title: Platform on Platform - CUBA apps on Cloud Foundry
description: "Deploy a CUBA app on Cloud Foundry is slightly different from the IaaS approach. In this blog post i'll highlight the differences and the way to get to a running CUBA app on Cloud Foundry."
modified: 2016-03-11
tags: [cuba, PaaS, cloud, Cloud Foundry]
image:
  feature: cloud-foundry/feature.jpg
  feature_source: https://pixabay.com/de/funkturm-berlin-nacht-geb%C3%A4ude-490032/
  
---

In this blog post I will [continue](http://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/) surfing the clouds and will have a look at the operations side of things. It's about running the CUBA Platform on the *Platform as a Service* that gains much momentum these days: [Cloud Foundry](https://www.cloudfoundry.org/).

<!-- more -->

In the [last](http://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/) blog post about Docker I highlighted how to use container technologies. Docker is a great technology that enables the road to *infrastructure as code*. 

### Everything as a Service
But running Docker is just one option to embrace the Cloud mindset. You can create a Amazon EC2 instance, install Docker on your own - this is a IaaS approach. In this case, you have to care about different stuff like orchestration, auto scaling, fault tolerance. Amazon also has more managed approach like [EC2 Container Service](https://aws.amazon.com/documentation/ecs/) and Docker itself tries to open up this market with solutions like [Docker Datacenter](https://www.docker.com/products/docker-datacenter). These alternatives are not really IaaS anymore, but more *Container as a Service (CaaS)*.

CaaS blurs the lines between classic IaaS and the topic I want to talk about in this article: *Platform as a Service (PaaS)*. 

### Abstracting away your Infrastructure

As you have probalby noticed, it's a little hard to distinguish between the categories, because the lines between them are dissolving more and more with time. To clearify this a little bit, you can follow this [link](http://cloudacademy.com/blog/cloud-foundry-benefits/) to get a traditional categorization. I personally like this tweet, that pretty much sums up the differences:


<div style="margin: auto auto 25% 25%; width: 50%">
<blockquote class="twitter-tweet" data-lang="de"><p lang="en" dir="ltr">The entire PaaS vs Container (e.g. Docker) debate explained in one simple diagram - <a href="https://t.co/B0oib5gihz">pic.twitter.com/B0oib5gihz</a></p>&mdash; swardley (@swardley) <a href="https://twitter.com/swardley/status/663089099989889024">7. November 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>  
</div>

<div style="margin-top:-175px">&nbsp;</div>

In the IaaS / Docker World the application developer has to know and care about stuff like filesystem structures, drivers for certain databases, copy files and set system environment variables Here's the Dockerfile from the [last](http://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/) post that shows this fact pretty good:

{% highlight dockerfile %}

FROM tomcat:8-jre8
ADD war/app.war /usr/local/tomcat/webapps/
ENV CATALINA_OPTS="-Dlogback.configurationFile=/opt/cuba_home/logback.xml"

{% endhighlight %}


Additionally you have to care about monitoring, backup, fault-tolerance, orchestration and many more things. 

To get that going you have different possibilities. You can achieve this stuff that is on top of basic containers with stuff like bash scripts / jenkins / docker commands & tools and so on. But to be honest, when comaring something like this to PaaS, you should keep this in mind:
  
<div style="margin: auto auto 10% 10%; width: 75%">
    <blockquote class="twitter-tweet" data-lang="de"><p lang="en" dir="ltr">Sure, you can choose to build your own <a href="https://twitter.com/hashtag/PaaS?src=hash">#PaaS</a>.. Comparing build your own to a structured platform be like.. <a href="https://t.co/KBW9KINYHn">pic.twitter.com/KBW9KINYHn</a></p>&mdash; Dan Mearls (@DanMearls) <a href="https://twitter.com/DanMearls/status/657961157114875905">24. Oktober 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

</div>


<div style="margin-top:-75px">&nbsp;</div>

So when comparing it to a PaaS, you don't have to care about this stuff anymore. In fact you are even not able to care about this stuff. The only thing you have to and can tell your PaaS environment is the runtime that you want to use for your application. In Cloud Foundry these environments are called [buildpacks](http://docs.cloudfoundry.org/buildpacks/).

To get a sense of how that looks like, let's move to Cloud Foundry and try to deploy a CUBA app on this platform.

<img style="float:right; padding: 10px;" src="{{site.url}}/images/cloud-foundry/cloud.png">

### Cloud Foundry 

Cloud Foundry is an open-source PaaS solution backed up by a lot of big companies. Initially started by VMWare / Pivotal and has become the base solution for different Cloud offerings like IBM Bluemix, HP helios and the Cloud solution from Pivotal itself.

When comparing it to other solutions like Heroku, the fundamental difference is that since CF is open source. This has a different positive side effects. One is, that you can run it in your own datacenter if you want to.

It is also "easier" to change from one Cloud vendor to the next as long as they all share the PaaS or in case of an IaaS provider it's up to you, to install CF on their infrastructure, because you can install CF on AWS or any other IaaS provider by yourself.


### Get ready to go via Pivotal Web Services

So let's get started with Cloud Foundry. To do so, I have created a free account at [Pivotal Web Services](https://run.pivotal.io/), the Pivotal CF Cloud offering, which allowes me to play with CF for 2 month for free. 

I choosed the Pivotal offering just out of convenience reasons. If you want to try it in your own datacenter (via [OpenStack](http://docs.cloudfoundry.org/deploying/openstack/index.html) or [VMWare vShpere](http://docs.cloudfoundry.org/deploying/vsphere/index.html)) or on [AWS EC2 instances](http://docs.cloudfoundry.org/deploying/aws/), feel free to do so.

After creating the account and their [CLI](http://docs.run.pivotal.io/cf-cli/) installation you are ready to login via the command line:

{% highlight bash %}
$ cf login -a https://api.run.pivotal.io
{% endhighlight %}

Since you logged in your command line is ready to go to push your application. But first, let's have a look at the web UI which allows you to view the running instances. 


<a href="{{ site.url }}/images/cloud-foundry/pws-ui.png"><img style="width: 50%; float:right" src="{{site.url}}/images/cloud-foundry/pws-ui.png"></a>

After a successful [login](https://login.run.pivotal.io/login) you will see a screen similar to this.

First, I created an organisation called "cuba-ordermanagement". This oragnisation contains one *space*. A *space* is an aggregation of services and apps that are scoped via project or an environment. 

I created a space called "development", which should describe the phase of my imaginary continous delivery pipeline. 

As you can see on the right, I already started my cuba app and one service, so let's look at how i came to this point (the shortcut of this is the [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement/tree/cloud-foundry) github repository where you'll find the sources in the branch *cloud-foundry*).


### Create Postgres Database in Cloud Foundry

First of all, we have to create a datastore for our cloud version of the cuba app. In *IaaS world* you would set up a vanilla postgres instance, figure out connection settings and tell your app where to find your datastore.

In Cloud Foundry your database is *just another* service, that the application can depend on. The Pivotal offering of Cloud Foundry has a couple of services available for stuff like caching, search, (non-) relational datastores, monitoring tools and so on. 

When you want to take a look at what the different offerings are, just use the marketplace cmd option.

{% highlight bash %}
$ cf marketplace
{% endhighlight %}

We will choose a postgres installation for now. You can either use the web UI or the command line interface:

{% highlight bash %}
$ cf create-service elephantsql turtle cuba-ordermanagement-postgres
{% endhighlight %}

*elephantsql* is the name for the service (Postgres as a Service) and *turtle* the service plan (turtle is the free one). *cuba-ordermanagement-postgres* is the name for the service instance, which we will need to refer to it later.

Now we have the running postgres db installation. Next thing - we have to configure our application to use this datastore.



## Make CUBA Cloud-ready &trade;

There are a few things that have to be changed from the traditional deployment model. We'll go through these step by step so you should be able to adjust your app right as we go.


### 1. Springs Cloud Connector for DataSource creation

Within the spring ecosystem there are different packages that handle integration with cloud systems in general and PaaS like Cloud Foundry in particular. In the <code>build.gradle</code> file, <code>coreModule</code> needs two additional dependencies as it is shown below:

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

Next, since we want the the database to be looked up from the cloud, we need to override the creation of the <code>dataSource</code> bean. This can be done via dependency injection. In <code>spring.xml</code> of the <code>core</code> module we tell Spring to use the <code>cubaDataSource</code> bean as the Factory bean, which is used throughout the CUBA application. 


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
class CloudFoundryDataSourceFactory {

    String dbServiceName

    DataSource createDataSourceForPostgresDbService() {

        Cloud cf = new CloudFactory().cloud
        def postgresSerciveInfo = cf.getServiceInfo(dbServiceName)
        cf.getServiceConnector(postgresSerciveInfo.id, DataSource, null)

    }
}
{% endhighlight %}

With this little glue code inplace (hope you don't mind me using groovy here), the application is ready to connect to a database via a service name instead of DNS names through the cloud platform.

The only thing that we have to tell to the application is that we actually want to use *postgres* as our DBMS. When we do this we can additionally put the other required configuration inplace.

### 2. Setup the *.properties files of the CUBA app

To tell CUBA that it should use Postgres as the DBMS we have to change the <code><a href="https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/modules/core/src/app.properties">app.properties</a></code> in the core module in the following way:

{% highlight properties %}

cuba.dbmsType = postgres
cuba.automaticDatabaseUpdate = true

{% endhighlight %}

Additionally we have to change the <code>app.home</code> attribute in <code><a href="https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/modules/core/src/app.properties">app.properties</a></code> in the core module and the <code><a href="https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/modules/web/src/web-app.properties">web-app.properties</a></code> file in the web module to <code>..</code>:


{% highlight properties %}

app.home = ..

{% endhighlight %}

### 3. Configure CUBA to be delivered as a single war

To make deployment of the app quite a bit easier, we'll combine the core and the web module into a single war file. This has some benefits but also some drawbacks regarding to scaling and so on. Due to this it's up to you if you want to follow this trail.

If you read the docs about the [single war approach](https://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#build.gradle_buildWar) you'll see, that we have to create another web.xml and use it during war building.

Just copy & paste the XML file from the [docs](https://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#build.gradle_buildWar) and put it in the <code>web</code> module to <code>web/WEB-INF/single-war-web.xml</code>. You have to adjust the Context parameter <code>Web Client Application class</code> to fit to your application class. The created file for cuba-ordermanagement you'll find [here](https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/modules/web/web/WEB-INF/single-war-web.xml).

After doing so, the last thing you have to do is to create a gradle task in the <code><a href="https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/build.gradle">build.gradle</a></code> file. In this task you reference the newly generated <code>single-war-web.xml</code> as the web.xml file like this:


{% highlight groovy %}

task buildWar(type: CubaWarBuilding) {
    appHome = '..'
    webXml = "${webModule.projectDir}/web/WEB-INF/single-war-web.xml"
}

{% endhighlight %}

### 4. Create deployment information for Cloud Foundry

Since we can't really get down to the underlying infrastructure of the app, we are not able to change the way the tomcat (or whatever servlet container is underneath your app in this case) works. But certain metadata has to be given to the PaaS in order to run our application properly, e.g. the java version, the amount of memory required, external services (like datastores) and so on.

For these kind of information, CF requires a file called *manifest.yml* to be in place. This file is the entry point for the deployment. So the file that we are going to create looks pretty much like [this](https://github.com/mariodavid/cuba-ordermanagement/blob/cloud-foundry/manifest.yml):

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

Note the service name that is referenced from the app instance. 

**That's it**. Ok, it took a little bit longer than I would like it to be. This is due to the fact that we have done different things here. Not all of them have directly to do with make it cloud ready. Anyway - now we are ready to take the app and deploy it to Cloud Foundry.

## Pushing it up into the clouds

First we need to create the war file that we want to deploy:

{% highlight bash %}
$ ./gradlew buildWar
{% endhighlight %}

Next, we do the actual deployment via the command cf push and point it our manifest file:

{% highlight bash %}
$ cf push -f manifest.yml
{% endhighlight %}

<img style="float:right; padding: 10px; width: 50%" src="{{site.url}}/images/cloud-foundry/cf-cuba-login.png">


Then the cli will care about uploading the artefact, creating a "server", connecting it to the service and start up the tomcat installation.

When everything worked out fine, you can access your application at [http://cuba-ordermanagement.cfapps.io/](http://cuba-ordermanagement.cfapps.io/) (or whatever Route is defined for the application. You can look it up via the web ui).

<div style="margin-top:15px">&nbsp;</div>

## Deployment is not the end of the road

Ok, when we look at what we have accomplished with this deployment, we see that we really only doing the first step. We have fulfilled some pre requirements like the single war stuff. Then we did some direct stuff to make the CUBA app runnable in the PaaS environment like the DataSource Generation from Spring Cloud.

We left out some parts, like configuring file storage. In a PaaS you can't have access to a local directory structure, because the platform don't lets you know about these kind of infrastructure. You would need something like Amazon S3. Luckily CUBA has a configurable bean *cuba_FileStorage* which you can inject the implementation *AmazonS3FileStorage* (included into CUBA Platform).

Another thing is how to do mailing the PaaS way. When you look through the Cloud Foundry marketplace you'll find services like [SendGrid](https://docs.run.pivotal.io/marketplace/services/sendgrid.html) that might make sense to include.

But let's look at what is missing at a bigger picture. One big benefit from the cloud in general is to be able to scale horizontally. *If the traffic of your app increases, spin up more machines and you are ready to go*. Well, to be able to do this, your software needs an architecture that allows to do that. Is this the case with CUBA? I'll leave it up to you to find it out, or you'll wait until i write about it in a future blog post.

I hope i could give you a good understanding of what it means to use a PaaS and what the differences are compared to approaches like Docker and the whole IaaS space.


<div style="margin-top:150px">&nbsp;</div>


<style type="text/css">

	div.entry-content {
    	background: url('/images/cloud-foundry/road-to-cloud.jpg') #fff;
    	background-position: bottom;
    	background-repeat: no-repeat;
	}

	.entry-meta {
		color:#000;
	}
	.entry-meta .tag {
		background-color:#000;
	}

	.entry-meta a {
		color:#000;
	}
</style>