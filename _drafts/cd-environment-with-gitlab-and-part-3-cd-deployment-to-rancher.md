---
layout: post
title: Continuous delivery with Gitlab and Rancher
subtitle: Part 3 - CD deployment from Gitlab to Rancher
description: ""
modified: 2017-09-01
tags: [cuba, gitlab, rancher, CD]
image:
  feature: cd-environment-with-gitlab-and-rancher/feature.png
---

The third and last part of this series will combine all the already configured parts. After manually deploying the example application to rancher, we will use the Gitlab CD pipeline to do the job automatically for us.

<!-- more -->

After we insatlled and configured Rancher in the last part of the second blog post, lets use this installation to deploy our application to rancher.

### Deploy to racher manually

In order to deploy an application to rancher, we need a docker-compose.yml file which describes our application and it's dependencies. In the [deployment directory](https://github.com/mariodavid/kubanische-kaninchenzuechterei/tree/master/deployment) of the sample app, there are two files. First the <code>docker-compose.yml</code>:

{% highlight yaml %}
version: '2'
services:
  web:
    image: 'mariodavid/kubanische-kaninchenzuechterei:latest'
    ports:
     - "8082:8080"
  db:
    image: "blacklabelops/hsqldb:latest"
    environment:
      - HSQLDB_USER=kaninchen
      - HSQLDB_PASSWORD=kaninchen
      - HSQLDB_DATABASE_ALIAS=kaninchen
{% endhighlight %}

Besides the web application (which is our Docker image we created earlier), we need a running database. For showing you a running app with least effort, i used a HSQLDB Docker container. In the web-application the connection settings are hard-coded. The environment variables of the HSQLDB container configure the database to be accessible with this credentials.

The next file is the <code>rancher-compose.yml</code>. It is basically a file for all the settings docker-compose is not capable of expressing in its file currently.

{% highlight yaml %}
version: '2'
services:
  web:
    scale: 1
  db:
    scale: 1
{% endhighlight %}

In this case we will just configure the amount of containers that should be running for every service.
