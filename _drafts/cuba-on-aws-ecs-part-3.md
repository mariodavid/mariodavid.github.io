---
layout: post
title: Run CUBA on AWS ECS - Part 3
description:
modified: 2016-08-09
tags: [cuba, AWS, cloud, ECS, VPC]
image:
  feature: cuba-on-aws-ecs-part-3/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
---

In this third part of the ECS article series we'll take a deeper look into the more advanced capabilities of the AWS world in order to enhance our current ECS deployment. Separating the application layers to scale independently is one of the main aspect of this article.

<!-- more -->

When talking about clustering of a CUBA / Docker application with different parts then there are several variants that will solve this problem to different degrees. To get a good overview about them we will first look into these variants and then go into the variant that we will cover in this article. But as these variants differ in what underlying objectives of clustering they cover, we will first have a look at those.

### Objectives for different cluster scenarios

When we want to cluster our application, we do this because of one of the two main aspects:
*scaling* and *availability*.

Scaling (in this case horizontal scaling) means that we can automatically or at least with very little effort react to higher or lower requirements regarding the amount of users and therefore amount of servers.

Availability means that there is roughly no single point of failure. It means that we can guarantee the uptime of the system to a specific percentage (normally a high percentage).
When we want to achieve availability, we have to take at the different components of the application and the environment in order to make sure we have no single point of failure in any of the components. The following components in the AWS infrastructure are parts that we have to take into account:

* Availability zones (as a Service)
* EC2 instances
* RDS instances  (as a Service)
* VPC components (as a Service)
* Container instances
* CUBA middleware cluster communication mechanisms
* ELB load balancer components (as a Service)

There are several other components that are also worth considering but oftentimes the uptime is already guaranteed through the service offering of AWS (like the components with a "as a Service" postfix) or they are out of scope of this article.


# Different variants for clustering

The different approaches distinguish themselves mostly on how to do networking and if the different components are seen as a Service with the corresponding uptime guarantees from AWS.

### Variant A: Using overlay network (docker swarm)
In the fist approach of docker clustering is to not use the underlying network possibilities of the Cloud provider (AWS) at all. In this case we would instead use the network capabilities that Container ecosystem provides. In case of Docker it would either be docker swarm which allows to spin up an overlay network among different docker hosts or something like kubernetes. ECS unfortunately does *not* allow to create overlay networks out of the box. This means, that in this variant we would have to not use ECS or try to bake in our own solution like done [in this example](https://github.com/justinwatkinson/ecs-weave-demo) with an integration between ECS and [weave](https://www.weave.works/).

In this variant the EC2 instances would be seen as a Service that we don't control and therefore don't care about (because all container communication is done via the overlay network).

### Variant B: single ECS cluster, multiple ECS services
In the second variant we would create a single ECS cluster and attach different EC2 instances to it. This pool of EC2 instances are responsible of running Docker containers. Then we would create different ECS services that will handle the different application components like middleware or web application.

This approach allows us stay in the ECS world a little more. To communicate between the application components we would have to use ELBs for every service in order to communicate e.g. between the web application and the middleware component.

Like in the last variant, the EC2 instances would be seen as a Service, meaning that we will just not care about them at all. Probably the EC2 instances would be grouped in a EC2 auto scaling group to handle the load automatically.

### Variant C: multiple ECS clusters with separated EC2 instances

In the last approach that we will take a look at, we use different ECS clusters. The reason is that we would want to launch different EC2 instances in the different ECS clusters.

In this approach we would take full advantage of the AWS networking platform and put the EC2 instances in different security groups within different subnets and different AZs. As we see here, we rely on the AWS networking features for the EC2 instances and the Docker containers within the EC2 instances will not be placed within an overlay network at all.

This approach makes sense if we would want to take control over the underlying EC2 instances and apply security constraints on them.

## The approach for this article
Although the approaches differ in the degree of flexibility and maintenance efforts they are mostly the same from a 10'000 foot view. Also there is no solution that should be taken because it is the best, but instead they have to be chosen because of the different requirements.

For this blog post we will go the route of the third approach with different ECS clusters and different EC2 instances. This is mostly done to talk a little bit about the networking features of AWS. Another interesting approach would be to use Kubernetes (either with or without ECS) to get the benefit of an overlay network. With the 1.12 release of Docker they introduced another very interesting approach regarding deployment to AWS, which is described in this blog post: [Docker for AWS](https://blog.docker.com/2016/06/azure-aws-beta/). Probably i'll do another blog post on one of these approaches.

But for now we'll stick to the third approach.


## Enhancements for the current deployment

There are several topics to cover in this advanced part. The first enhancement in the ECS deployment is that the different layers of the CUBA application are currently deployed with the same task definition. This is really straight forward but it requires to scale the application as a whole, meaning always have the same amount of frontend instances as middleware instances. Although this might also a valid scaling options, we will not go that route. Instead we will first deploy the layers on individual ECS clusters.

# Deploy application layers independently

In order to deploy the layers in different ECS clusters the first thing we need to create another ECS cluster.


### creating different ECS clusters

### the missing overlay network
Within a task definition docker containers can directly talk to each other because they use the overlay network within the same docker host. This is equivalent to use docker-compose directly.


### middleware cluster through JGroups Gossip router


# VPC for different security requirements on different

### VPC subnets

### NAT instance for outgoing internet access

### security groups for FE and BE


# Load balance front-end application through ELB


# Central logging with CloudWatch Logs

Instead of logging into the EC2 instance to get the Docker container logs we will instead use [AWS cloud watch logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/CWL_GettingStarted.html). ECS has a integration to cloud watch logs. It takes STDOUT and STDERR from the Docker container and pushes it into the logging system, where we can easily retrieve the information from tomcat in a web UI. To get that going the only thing that has to be done is to [create](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logs:) a log group in your preferred AWS region. Then you define the name of the log group in the JSON key *awslogs-group*. After that the logs will be send to the central logging system.
