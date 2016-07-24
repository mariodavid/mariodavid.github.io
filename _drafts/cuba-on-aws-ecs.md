---
layout: post
title: Run CUBA with AWS ECS
description:
modified: 2016-03-29
tags: [cuba, crud]
image:
  feature: cuba-on-aws-ecs/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
---

Going from running Docker in the command line to a production scenario can be quite challenging since there is so much more to cover and so much more possibilities to do it right. One solid way of Docker and the Cloud is AWS.

In this article i'll go through the different possibilities AWS has to offer especially regarding Container as a Service. We will deploy the ordermanagement CUBA app on a ECS cluster and use different features of the AWS cloud to leverage the full cloud potential to CUBA.

<!-- more -->

### A brief history on cloud land

AWS is Amazons cloud offering. It is a very popular cloud provider and probably one of the oldest ones as well. Not long ago i [read](https://twitter.com/peakscale/status/674278519732486144) the pharse which sets the scene for the cloud market dominance of AWS quite well:

> Nobody ever got fired for choosing AWS

Reffering to the well known marketing term "[No one ever got fired for buying IBM](http://corporatevisions.com/blog/2007/06/11/no-one-ever-got-fired-for-buying-ibm/)".
AWS started as a Infrastructure as a Service (IaaS) platform. With EC2 it offers for compute capabilities (bisically virtual machines). For storage there are a few more options: S3 / DynamoDB / RDS address filestorage, non-relational and relational data storage. After that a lot of other services pop up. These were not only in the IaaS space, but in the P/SaaS space as well. Things like Elastic Beanstalk or Amazon Work Mail filled the gap between the low level infrastructure services and alternatives like Heroku.

At the end of 2014 AWS [announced](https://aws.amazon.com/de/blogs/aws/category/ec2-container-service/) ECS, which is a service that is a layer on top of Docker containers in order to orchestrate and manage containers.

ECS is a offering in a highly competitve market. Docker Swarm, Apache Mesos, Kubernetes etc. are just a few tools to mention. Although ECS is not open source like the other examples, it is mainly build on top of Docker and therefore a good fit for deploying our open source CUBA application [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement). On a later stage i'll probably take a look at Kubernetes because it has a fairly user base as well and additionally a lot of services, like [OpenShift](https://www.openshift.com/) from Red Hat or [Rancher](http://rancher.com/) use it as a basis for their solutions.

To get back to the topic of this article, let's have a look at the different services that we'll use throughout the deployment with ECS and AWS.

## AWS building blocks relevant for ECS

<img style="float:right; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs/ec2-logo.png">

#### EC2
EC2 is the basis of a lot of AWS services. It is their compute solution. Basically you can create virtual machines on a virtual server that has certain capabilities, like amount of RAM, CPU power, network bandwith and so on. An EC2 instance can go from a VM with 512 mb RAM to a VM with 2 TB of RAM and 128 vCPUs. [Here](https://aws.amazon.com/ec2/instance-types/) is an overview of instance types.


<img style="float:right; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs/elb-logo.png">


#### ELB
Load balancing on AWS can be done through their service "elastic load balancer". It is a service that allows to route traffic to EC2 instances. Additionally it can termininate SSL in case the network traffic is HTTP based. The main selling point on ELB is probably the fact that it is built with a high availability concern in mind. In this case it is just like a few other AWS services. Oftentimes a load balancer can be easily built with the essential building blocks of the AWS environment: EC2 instances. The problem is, that creating a really high avaliable solution with something HAProxy is fairly complicated. Especially if the solution should work across two data centers or alike. This is where ELB shines. [This article](http://www.stackdriver.com/elb-affinity-problems/) describes the situation around AWS ELB and other approaches very good.


<img style="float:right; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs/rds-logo.png">


#### RDS
RDS is another example of a high level service that sits on top of EC2. It is about storing data in relational databases. Sounds simple, but if you have ever tried to install a Oracle installation in cluster mode across two data centers with automatic backup and fail over - you'll probably know how much time it is going to cost. RDS allows people to do this within a few clicks. It supports the main database vendors and even cares about licensing if you don't want to explicitly.


<img style="float:right; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs/vpc-logo.png">


#### VPC
With virtual private cloud: "VPC", AWS allows controlling the different pieces of your infrastructure to be not accessible to everyone in the internet. You can create subnet's within your VPC, connect or disconnect it from the internet, defining security rules about what has access to what within the VPC and so on. Even a dedicated VPN tunnel can be created so that there is no internet access needed at all.


<img style="float:right; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs/ecr-logo.png">

#### ECR
ECR stands for elastic container registry. It is the fully managed solution for the Docker containers. Basically it is the same thing as Dockerhub for storing your Docker images. But it has tight integration in the full fletched security mechanisms on AWS. Besides that it uses the same protocol as the docker registry does. Due to this, the Docker CLI <code>docker pull tomcat:8-jre</code> will seamlessly work with ECR.


<img style="float:right; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs/ecs-logo.png">

#### ECS

ECS is the last building block we will use in this article(s). As described above, it is the solution from AWS for Container orchestration. We'll go into much more detail about this topic because we need to configure the different pieces within ECS, but for know it can be seen as the part that automatically deploys Docker containers to existing EC2 instances and cares about the healthiness of the containers.

## Deploying and running CUBA apps on AWS

To give you a general idea of what scenarios will be covered in the following article(s), here's an overview diagram:

<figure class="center">
	<a href="{{ site.url }}/images/cuba-on-aws-ecs/run-cuba-on-aws-overview.png"><img src="{{ site.url }}/images/cuba-on-aws-ecs/run-cuba-on-aws-overview.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-on-aws-ecs/run-cuba-on-aws-overview.png" title="run CUBA application of AWS with ECS overview">run CUBA application of AWS with ECS overview</a></figcaption>
</figure>

The diagram shows the different building blocks of the AWS infrastructure that are relevant for the ECS deployment. I'll describe on a very broad basis the workflow and we'll go into much more details of the different steps afterwards.

First either the developer or your favorite CI system kicks off the deployment process. To do this, the Docker image get build locally and pushed to the central Docker repository. Next the elastic container service (ECS) gets notified to redeploy the newly created Docker image.

Since ECS is just the orchestration layer but not responsible for actually running the Docker containers, pre configured EC2 instances are contacted in order to redeploy.

To ensure fault tolerance on the EC2 instance level as well as the Docker container level, multiple EC2 instances serve the multiple instances of the Docker image. Thus there is a need for a load balacing mechanism that will shield the docker containers from direct internet connection, terminate SSL and balance requests between the instances.
