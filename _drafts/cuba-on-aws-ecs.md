---
layout: post
title: Run CUBA with AWS ECS
description:
modified: 2016-03-29
tags: [cuba, crud]
image:
  feature: 2015-10-29-my-personal-crud-story/feature.jpg
  feature_source: https://pixabay.com/de/gliederung-karte-kuba-geographie-322490/
---

Going from running Docker in the command line to a production scenario can be quite challenging since there is so much more to cover and so much more possibilities to do it right. On solid way of Docker and the Cloud is AWS.

In this article i'll go through the different possibilities AWS has to offer especially regarding Container as a Service. We will deploy the ordermanagement CUBA app on a ECS cluster and use different features of the AWS cloud to leverage the full cloud potential to CUBA.

<!-- more -->

### A brief history on Cloud land

AWS is Amazons cloud offering. It is a very popular cloud provider and probably one of the oldest ones as well. Not long ago i [read](https://twitter.com/peakscale/status/674278519732486144) the pharse which sets the scene for the cloud market dominance of AWS quite well:

> Nobody ever got fired for choosing AWS

AWS started as a IaaS platform. Is offers EC2 for compute and S3 / DynamoDB / RDS for storage services. After that a lot of other services pop up. These were not only in the IaaS space, but in the P/SaaS space as well. Things like Elastic Beanstalk or Amazon Work Mail filled the gap between the low level infrastructure services and alternatives like Heroku.

At the end of 2014 AWS [announced](https://aws.amazon.com/de/blogs/aws/category/ec2-container-service/) ECS, which is a service that is a layer on top of Docker containers in order to orchestrate and manage containers.

ECS is a offering in a highly competitve market. Docker Swarm, Apache Mesos, Kubernetes etc. are just a few tools to mention. Although ECS is not open source it is mainly build on top of Docker and therefore a good fit for deploying our open source CUBA application [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement).


## AWS building blocks relevant for ECS

* EC2
* ECR
* ECS
* ELB
* RDS
* VPC

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
