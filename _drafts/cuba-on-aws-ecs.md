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
