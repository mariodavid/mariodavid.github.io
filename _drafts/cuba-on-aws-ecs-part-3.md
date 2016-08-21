---
layout: post
title: Run CUBA on AWS ECS - Part 3
description:
modified: 2016-08-09
tags: [cuba, AWS, cloud, ECS, VPC]
image:
  feature: cuba-on-aws-ecs-part-3/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
  feature-top: cuba-on-aws-ecs-part-3/feature.jpg
---

In this third part of the ECS article series we'll take a deeper look into the more advanced capabilities of the AWS world in order to enhance our current ECS deployment. Separating the application layers to scale independently is one of the main aspect of this article.

<!-- more -->

When talking about clustering of a CUBA / Docker application with different parts then there are several variants that will solve this problem to different degrees. To get a good overview about them we will first look into these variants and then go into the variant that we will cover in this article. But as these variants differ in what underlying objectives of clustering they cover, we will first have a look at those.

### Objectives for different cluster scenarios

When we want to cluster our application, we do this because of one of the two main aspects:
*scaling* and *availability*.

Scaling (in this case horizontal scaling) means that we can automatically or at least with very little effort react to higher or lower requirements regarding the amount of users and therefore amount of servers.

Availability means that there is roughly no single point of failure. It means that we can guarantee the uptime of the system to a specific percentage (normally a high percentage).
When we want to achieve availability, we have to take at the different components of the application and the environment in order to make sure we have no single point of failure in any of the components. A few components that we control and are not parts of the Service of AWS are *EC2 instances, Container instances, CUBA middleware cluster communication mechanisms*


# Different variants for clustering

The different approaches distinguish themselves mostly on how to do networking and if the different components are seen as a Service with the corresponding uptime guarantees from AWS.


<img style="float:left; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-3/variant-a.png">


### Variant A: Using overlay network (docker swarm)

In the fist approach of docker clustering is to not use the underlying network possibilities of the Cloud provider (AWS) at all. In this case we would instead use the network capabilities that Container ecosystem provides. In case of Docker it would either be docker swarm which allows to spin up an overlay network among different docker hosts or something like kubernetes. ECS unfortunately does *not* allow to create overlay networks out of the box. This means, that in this variant we would have to not use ECS or try to bake in our own solution like done [in this example](https://github.com/justinwatkinson/ecs-weave-demo) with an integration between ECS and [weave](https://www.weave.works/).

In this variant the EC2 instances would be seen as a Service that we don't control and therefore don't care about (because all container communication is done via the overlay network).


<img style="float:left; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-3/variant-b.png">

### Variant B: single ECS cluster, multiple ECS services
In the second variant we would create a single ECS cluster and attach different EC2 instances to it. This pool of EC2 instances are responsible of running Docker containers. Then we would create different ECS services that will handle the different application components like middleware or web application.

This approach allows us stay in the ECS world a little more. To communicate between the application components we would have to use ELBs for every service in order to communicate e.g. between the web application and the middleware component.

Like in the last variant, the EC2 instances would be seen as a Service, meaning that we will just not care about them at all. Probably the EC2 instances would be grouped in a EC2 auto scaling group to handle the load automatically.


<img style="float:left; padding: 10px; width: 64px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-3/variant-c.png">

### Variant C: multiple ECS clusters with separated EC2 instances

In the last approach that we will take a look at, we use different ECS clusters. The reason is that we would want to launch different EC2 instances in the different ECS clusters.

In this approach we would take full advantage of the AWS networking platform and put the EC2 instances in different security groups within different subnets and different AZs. As we see here, we rely on the AWS networking features for the EC2 instances and the Docker containers within the EC2 instances will not be placed within an overlay network at all.

This approach makes sense if we would want to take control over the underlying EC2 instances and apply security constraints on them. Another reason might be that you have specific requirements regarding the hardware that powers the EC2 instance like SSD storage with provisioned IOPS.

## The approach for this article

Although the approaches differ in the degree of flexibility and maintenance efforts they are mostly the same from a 10'000 foot view. Also there is no solution that should be taken because it is the best, but instead they have to be chosen because of the different requirements regarding security, performance and ease of use.

For this blog post we will go the route of variant C with different ECS clusters and separate EC2 instances. This is mostly done to talk a little bit about the networking features of AWS. You can find the imaginary security requirements for our deployment a little bit below in this article.

It's worth mentioning that with the 1.12 release of Docker they introduced another very interesting approach regarding deployment to AWS, which is described in this blog post: [Docker for AWS](https://blog.docker.com/2016/06/azure-aws-beta/). Probably i'll do another blog post on this one because it eliminates the missing overlay network in the ECS deployment.

But for now let's take a closer look at what we have to do in order to achieve Variant C.


<figure class="center">
	<img src="{{site.url}}/images/cuba-on-aws-ecs-part-3/variant-c.png" style="width: 64px;">
</figure>


# Enhancements for the current deployment

There are several topics to cover in this advanced part. The first enhancement in the ECS deployment is that the different layers of the CUBA application are currently deployed with the same task definition. This is really straight forward but it requires to scale the application as a whole, meaning that the application always has the same amount of frontend instances as middleware instances.

Although this might also a valid scaling options, we will not go that route. Instead we will first deploy the layers on different ECS clusters. To deploy the application layers independently we will seperate the middleware part of the [cuba-ordermanagement](
https://github.com/mariodavid/cuba-ordermanagement/blob/cuba-on-aws-ecs/) application from the web application.

### creating second ECS cluster for middleware

To create another ECS cluster we can use the AWS cli for the middleware application layer like this (cluster name: "cuba-ordermanagement-cluster-core"):

{% highlight bash %}
$ aws ecs create-cluster --cluster-name=cuba-ordermanagement-cluster-core
{
    "cluster": {
        "status": "ACTIVE",
        "clusterName": "cuba-ordermanagement-cluster-core",
        "registeredContainerInstancesCount": 0,
        "pendingTasksCount": 0,
        "runningTasksCount": 0,
        "activeServicesCount": 0,
        "clusterArn": "arn:aws:ecs:us-east-1:123456789010:cluster/cuba-ordermanagement-cluster-core"
    }
}

{% endhighlight %}


This will give us the possibility to create another EC2 instance that should join this new cluster. The next thing is that we create this EC2 instance, but first we will think about the security requirements for this backend instance.

### Middleware security requirements

In the EC2 instance creation wizard we have to make certain decisions regarding networking and security, so before going right into creating the instance, we should take a look at the security goals that we want to achieve for the middleware layer. The middleware servers should...

* have no incoming internet access
* only be accessible from the web application servers
* be the only layer with access to the RDS service

To achieve this goals we will take a look at the already mentioned VPC feature of AWS.

# AWS Virtual private cloud - VPC
I talked about VPC already briefly in the [first part](https://www.road-to-cuba-and-beyond.com/cuba-on-aws-ecs-part-1/) of the article.
Your AWS account has one default VPC per region. Every resource like a EC2 instance is created within this VPC by default. The default VPC one or multiple subnets and can have multiple security groups associated to it.

To start with use the "create VPC" button on the [VPC overview](https://console.aws.amazon.com/vpc/home?region=us-east-1#vpcs:). The name for our VPC will be "cuba-vpc". The CIDR block describes the ip address range that is used for this VPC. We will use <code>10.0.0.0/16</code> for now. If your not familiar with wht CIDR is, basically the <code>/16</code> describes the bit mask that defines the network part. So in this case the first two numbers (10.0) will be our subnet part and the following two numbers are for our hosts within the network. You might take a look at the [CIDR wiki article](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) for more information.

### Subnet creation for CUBA-VPC

Next we will create two subnets within this vpc. You have to define a unique name for the subnet and the CIDR block that has to be places within the VPC CIDR block.

<figure class="center">
	<a href="{{ site.url }}/images/cuba-on-aws-ecs-part-3/create-subnet-for-vpc.png"><img src="{{ site.url }}/images/cuba-on-aws-ecs-part-3/create-subnet-for-vpc.png" style="width: 500px;"></a>
	<figcaption><a href="{{ site.url }}/images/cuba-on-aws-ecs-part-3/create-subnet-for-vpc.png" title="Subnet creation for the newly created VPC cuba-vpc">Subnet creation for the newly created VPC "cuba-vpc"</a></figcaption>
</figure>

We create the following four subnets:

* *cuba-vpc-middleware-subnet-1a*, CIDR: <code>10.0.0.0/24</code>, AZ: *us-east-1a*
* *cuba-vpc-middleware-subnet-1b*, CIDR: <code>10.0.1.0/24</code>, AZ: *us-east-1b*
* *cuba-vpc-web-subnet-1a*, CIDR: <code>10.0.2.0/24</code>, AZ: *us-east-1a*
* *cuba-vpc-web-subnet-1b*, CIDR: <code>10.0.3.0/24</code>, AZ: *us-east-1b*

With this we have for the middleware as well as the web layer two subnets that are in different availability zones. We have the basis for creating EC2 instances that will basically live in two datacenters, so that an outage of an AZ will not affect the application.

### Configure internet access for subnets

In order to communicate with the internet and with AWS services like the ECS scheduler, the instances have to be able to talk outside the <code>10.0.0.0/16</code> network. There are different approaches to enable that. Since this is another broad topic i'll take a little shortcut here: All subnets have to have an entry in their "Route Table" Tab with <code>0.0.0.0/0</code> that points to an Internet Gateway (one IGW should already be there due to the default VPC in your account). You have to adjust this in the [Route Table](https://console.aws.amazon.com/vpc/home?region=us-east-1#routetables:) section.


Another common (and more suitable) approach is to use the NAT Gateway that will grand access to the instances of the VPC via network address translation. For more information you might take a deeper look into the [VPC NAT Gateway section](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html).

<div class="information">
Generally the documentation on AWS is so deep and so comprehensive that i absoloutely encourage you to look at these docs. Here is an <a href="http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenarios.html">overview</a> on different VPC networking implementations that will cover a lot of more scenarios.
</div>

### Security groups for FE and BE
After setting up the networking part correctly we can adjust the firewall settings so that we can define what traffic is allowed for the different subnets & resources.

When we go to [Security Groups](https://console.aws.amazon.com/vpc/home?region=us-east-1#securityGroups:) via the management UI we can create the following two security groups: *cuba-vpc-middleware-security-group* and *cuba-vpc-web-security-group*.
In the *Inbound Rules* tab of the middleware security group we can define what traffic is allowed for incoming requests for resources associated with this security group. Let's create the following rules:


# Load balance front-end application through ELB


# Central logging with CloudWatch Logs

Instead of logging into the EC2 instance to get the Docker container logs we will instead use [AWS cloud watch logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/CWL_GettingStarted.html). ECS has a integration to cloud watch logs. It takes STDOUT and STDERR from the Docker container and pushes it into the logging system, where we can easily retrieve the information from tomcat in a web UI. To get that going the only thing that has to be done is to [create](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logs:) a log group in your preferred AWS region. Then you define the name of the log group in the JSON key *awslogs-group*. After that the logs will be send to the central logging system.
