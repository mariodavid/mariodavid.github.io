---
layout: post
title: Run CUBA on AWS ECS - Part 2
description:
modified: 2016-08-09
tags: [cuba, AWS, ECR, cloud, ECS]
image:
  feature: cuba-on-aws-ecs/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
---

After talking about AWS ECS in general and walked through the required setup, in this second part of the AWS ECS article, we will create a simple ECS cluster and deploy the created Docker containers onto it.

<!-- more -->

In the last blog post we left off with a ECR repository filled up with our docker image of the cuba-ordermanagement application. Actually it were two seperate repositories for each application component (middleware and web client).

To recap a little further, let's have a look at the overview diagram:

<figure class="center">
	<a href="{{ site.url }}/images/cuba-on-aws-ecs-part-2/run-cuba-on-aws-overview-part-2.png"><img src="{{ site.url }}/images/cuba-on-aws-ecs-part-2/run-cuba-on-aws-overview-part-2.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-on-aws-ecs-part-2/run-cuba-on-aws-overview-part-2.png" title="run CUBA application of AWS with ECS overview">run CUBA application of AWS with ECS overview</a></figcaption>
</figure>

In the first part we "just" got the ECR binary push working. But with the additional setup we prepared the ground for the steps that have to be taken in this part. So without furhter ado, let's prepare the second arrow of the diagram: the ECS deployment.

## Create a ECS cluster

<img style="float:right; width: 128px; padding: 5px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-2/create-ecs-cluster.png">

The first thing for a successful deployment is that we create a ECS cluster. THe corresponding [ECS management UI](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters) lists all existing clusters. Creating a cluster for now just means defining a name. Nevertheless, the defined name will be needed later on for referencing the cluster in scripts. So, we will take "cuba-ordermanagement-cluster" in this case

<div class="information">Every action that we do in this tutorial can be done easily via the AWS CLI as well. We will switch to the command line later, but for now we will stick to the UI .</div>


#### EC2 instances for ECS cluster

After creating the ECS cluster, we need to assign EC2 instances to it. This is a situation worth mentioning. Since ECS is just the orchestration engine for Docker, nothing prevents as well as saves us from defining the underlying execution system. Due to this we will take a look at the EC2 management UI.


<img style="float:right; width: 128px; padding: 5px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-2/launch-ec2-instance.png">

The launch wizard to create a EC2 virtual machine allows you to select different options to create an instance. The list is pretty comprehensive and i'll therefore i will just mention the relevant settings.

*Amazon machine image*

Generally it is possible to create EC2 instances with any Linux distribution. But that would require a little more handwork. The reason is that in order to communicate with the ECS orchestration engine, the Linux distribution needs to run a specific [ECS agent](https://github.com/aws/amazon-ecs-agent) (as a Docker container).

Due to this we will follow the AWS [recommendation](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_instances.html) and use the pre configured machine image: *Amazon ECS-Optimized Amazon Linux AMI* (can be found via the AWS marketplace). This image already includes everything that is needed to act as a ECS EC2 instance.

*Instance type*

As the instance type a t2.micro instance will work out for the ordermanagement application (which is good because this instance type works with the [free usage tier](https://aws.amazon.com/free/) of AWS).

*Instance configuration*

In step "3. configure instance" the IAM role "ecsInstanceRole" has to be choosen (i assume you have created it already during the setup in the first part - otherwise take a look at the [docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html)).

Additionally make sure that the EC2 instance's configuration option "Auto-assign Public IP" is set to Enabled (or "use subnet setting (enabled)"), so that we can access the web UI later without further problems.

In the section "advanced details" > "user data" the following script will configure the instance to work with the previously created ECS cluster:

{% highlight bash %}
#!/bin/bash
echo ECS_CLUSTER=cuba-ordermanagement-cluster >> /etc/ecs/ecs.config
{% endhighlight %}


If you had problems follow my instructions, you can take a look at the "[Launching a ECS instance](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_container_instance.html)" guide directly from AWS.



#### ECS service for cuba-ordermanagement

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#### ECS task definition

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Deploy application to ECS cluster

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
