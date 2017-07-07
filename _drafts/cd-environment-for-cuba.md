---
layout: post
title: Continous delivery for CUBA applications
subtitle: Gitlab
description: ""
modified: 2017-07-07
tags: [cuba, gitlab]
image:
  feature: grails-vs-cuba/feature.png
---

In this blog post i would like to show how to create a self-hosted continous delivery pipeline for CUBA applications.
<!-- more -->

When developing applications in a more or less professional setting, it requires
to have something like a continous integration / continous delivery pipeline in place.
The reason for that is, that

When you look around at how to solve these problems, you'll quickly find online services
that do the job very very well. Take a look at [Github](https://github.com/) as a source code repository or [Travis CI](https://travis-ci.org/) as a CI tool. These are all really good options, if you are either having the luxury working
on open source software or you are willing to pay for these SaaS tools (which you probably really should thinking about).

Nevertheless, in other scenarios where for whatever reason want to self-host some of these tools, there are options as well. In this blog post series I will do exactly that. I'll go only to a certain amount of detail regarding how to configure these tools, but give you further links to dive deep if you want to.

Let's start with one of the first and probably most important tools for a professional CI / CD pipeline: The source code repository.

## Source code repository

A repository where your application source code is crucial and it is a must have for almost 30 years in the software industry. One very example of a VCS (version control system) is [Git](https://git-scm.com/) and since it has become so dominant in the last years, we will focus on that.

As i already said, a lot of online hosted git respository options are available. However, we will take a look at an open source version, self-hosted version of it, called: [Gitlab](https://www.gitlab.com/). Although Gitlab offers online hosting, it is also possible to self-host the software - and this is what we will do.

Gitlab consists of different parts: a web application, the actual storage of the source code, a relational database for the web application etc.

Luckily Gitlab offers two distribution packages that will make handling a Gitlab installation much easier: *The Omnibus package* and *a Docker container*. The omnibus package, just like the name suggests, has everything packed into a single thing sothat you as a user don't really have to care about a lot of stuff. The Docker container packages this all together so that you can start it with a single command.

You can find the Gitlab CE docker container on [Dockerhub](https://hub.docker.com/r/gitlab/gitlab-ce/).

To start up a Gitlab instance, you have to execute the following command:

{% highlight bash %}
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
{% endhighlight %}
