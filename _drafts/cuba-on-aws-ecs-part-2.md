---
layout: post
title: Run CUBA on AWS ECS - Part 2
description:
modified: 2016-08-09
tags: [cuba, AWS, ECR, cloud, ECS]
image:
  feature: cuba-on-aws-ecs-part-2/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
---

After talking about AWS ECS in general and walked through the required setup, in this second part of the AWS ECS article, we will create a simple ECS cluster and deploy the created Docker containers onto it.

<!-- more -->

In the last blog post we left off with a ECR repository filled up with our docker image of the cuba-ordermanagement application. Actually it were two seperate repositories for each application component (middleware and web client).

In the first part we "just" got the ECR binary push working. But with the additional setup we prepared the ground for the steps that have to be taken in this part. So without furhter ado, let's prepare the second arrow of the diagram: the ECS deployment.

## Create a ECS cluster

<a href="https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters">
<img style="float:right; width: 128px; padding: 5px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-2/create-ecs-cluster.png"></a>

The first thing for a successful deployment is that we create a ECS cluster. The corresponding [ECS management UI](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters) lists all existing clusters. Creating a cluster for now just means defining a name. Nevertheless, the defined name will be needed later on for referencing the cluster in scripts. So, we will take "cuba-ordermanagement-cluster" in this case

<div class="information">Every action that we do in this tutorial can be done easily via the AWS CLI as well. We will switch to the command line later, but for now we will stick to the UI .</div>


#### EC2 instances for ECS cluster

After creating the ECS cluster, we need to assign EC2 instances to it. This is a situation worth mentioning. Since ECS is just the orchestration engine for Docker, nothing prevents as well as saves us from defining the underlying execution system. Due to this we will take a look at the EC2 management UI.

<a href="https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LaunchInstanceWizard:">
<img style="float:right; width: 128px; padding: 5px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-2/launch-ec2-instance.png"></a>

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

That are the main points. The other settings can be left to the defaults for now. We will change it in the next parts towards a more sophisticated setup, but for the start it is enough.
If you had problems follow my instructions, you can take a look at the "[Launching a ECS instance](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/launch_container_instance.html)" guide directly from AWS.

After setting up the EC2 image, it can be started. After a couple of seconds the instance should be up and running. Additionally, when you take a look at the ECS cluster details view within the tab "ECS instances" the EC2 instance should be listed.


#### ECS task definition

A task definition in the ECS world is very similar to the docker-compose.yml file introduced in the [last Docker blog post](https://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/). A task definition defines how one or many different Docker image should be run. Port mappings can be defined as well as environment variables and volume mountings. In this regard there is no difference to a docker-compose.yml file.
One difference about the two is there specific AWS features IAM roles or the log configuration for the Containers that are used for certain containers.

In the [repository](
https://github.com/mariodavid/cuba-ordermanagement/blob/cuba-on-aws-ecs/deployment/ecs/task-definitions/cuba-ordermanagement-task-definition-templage.json) i created a template task definition that looks like this:

{% highlight json %}
{
  "family": "cuba-ordermanagement",
  "containerDefinitions": [
    {
      "essential": true,
      "name": "cuba-ordermanagement-app-core",
      "image": "1234567890.dkr.ecr.us-east-1.amazonaws.com/cuba-ordermanagement-app-core:%CUBA_VERSION%",
      "environment": [
        {
          "name": "CUBA_ECS",
          "value": "true"
        },
        {
          "name": "CUBA_CLUSTER_ENABLED",
          "value": "false"
        },
        {
          "name": "CUBA_DB_NAME",
          "value": "cuba-ordermanagement"
        },
        {
          "name": "CUBA_DB_HOST",
          "value": "cuba-ordermanagement.xhgs6jjsnl.us-east-1.rds.amazonaws.com"
        },
        {
          "name": "CUBA_DB_PASSWORD",
          "value": "1234567890"
        },
        {
          "name": "CUBA_DB_USERNAME",
          "value": "cuba-ordermanagement"
        },
        {
          "name": "CUBA_DB_PORT",
          "value": "5432"
        },
        {
          "name": "CUBA_WEB_PORT",
          "value": "80"
        }
      ]
    },
    {
      "memory": 256,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "name": "cuba-ordermanagement-app",
      "image": "1234567890.dkr.ecr.us-east-1.amazonaws.com/cuba-ordermanagement-app:%CUBA_VERSION%",
      "environment": [
        {
          "name": "CUBA_ECS",
          "value": "true"
        },
        {
          "name": "CUBA_WEB_PORT",
          "value": "80"
        },
        {
          "name": "CUBA_CORE_URL_CONNECTION_LIST",
          "value": "http://cuba-ordermanagement-app-core:8080"
        }
      ],
      "links": [
        "cuba-ordermanagement-app-core"
      ]
    }
  ]
}
{% endhighlight %}

The task definition creates two containers: *cuba-ordermanagement-app-core* and *cuba-ordermanagement-app*. They reference an image from the previously created ECR repository (thus you have to replace the registry host name with the real one). The version *%CUBA_VERSION%* is used for later replacement either manually or via a script (as we will see later).

Both define different environment variables (CUBA_) that are required to start the container. For the middleware app (app-core) the db connection has to be setup (CUBA_DB_). For the web application the url for the middleware server(s) have to be set (CUBA_CORE_URL_CONNECTION_LIST).

Additionally the tomcat has to know which is the host name (CUBA_WEB_HOST_NAME) it s running on as well as the port that the application is listening on (CUBA_WEB_PORT). In case of ECS is active the host name can be determined automatically (CUBA_ECS).

In the *links* section of the web application references the middleware application so that the docker containers can communicate with each other. For the web application a port mapping has to be setup so that the web application is accessible from outside the EC2 instance.

To create the task definition you can go to the [management console](https://console.aws.amazon.com/ecs/home?region=us-east-1#/taskDefinitions/create) and click "configure via JSON" and paste the template in (and replacing the *%CUBA_VERSION%* template with latest), or you can use the UI to create the task definition.

After the successful creation of the task definition, we will go to the last missing piece: the ECS service.

#### ECS service for cuba-ordermanagement

The last thing on the way to run our CUBA application on ECS is the ECS service. A service is responsible for running a certain task definition within a ECS cluster.

To create a service, open the ECS cluster detail view. In the Tab "Services", the button "Create" will lead you to this page:


<figure class="center">
	<a href="{{ site.url }}/images/cuba-on-aws-ecs-part-2/create-service.png"><img src="{{ site.url }}/images/cuba-on-aws-ecs-part-2/create-service.png" style="width: 400px;"></a>
	<figcaption><a href="{{ site.url }}/images/cuba-on-aws-ecs-part-2/create-service.png" title="Form to create ECS service">Form to create ECS service</a></figcaption>
</figure>

Define a service name "cuba-ordermanagement-service". The cluster and the task definition should be obvious. The ":1" is the revision for the task definition. A task definition can't be changed but a new revision will be created in this case.

The number of tasks defines how many instances of the task definition and therefore the number of docker containers have to be running at any given time. For now we will use "0", so that the ECS service is not directly going to deploy the application, as we will do this within the deployment script later. In a more high availability scenario the number would be higher. In case one docker container or EC2 instance goes down, the ECS service will notice this and start another Docker container / ECS task.

There are other optional possibilities for a ECS service, like to configure a auto-scaling group or a load balancer. We will go into a little more detail on that in the third part of the article.

## Create a RDS instance


<a href="https://console.aws.amazon.com/rds/home?region=us-east-1#launch-dbinstance:ct=dbinstances"><img style="float:right; width: 150px; padding: 5px;" src="{{site.url}}/images/cuba-on-aws-ecs-part-2/launch-rds-instance.png"></a>

When you look at the diagram from the last part i showed that we will use RDS as the relational database backend for the cuba-ordermanagement application. To achieve this goal, we will use the relational database service from AWS to get a fully managed DBMS instance.

First of all, navigate to the RDS managament UI. Then choose "launch DB instance". Next you select "PostgreSQL" as the database server (you can use other vendors as well - basically everything that CUBA / JDBC supports, but i'll stick with PostgreSQL for now). Then there a a couple of options. First the wizard asks if it will be a production or a testing database. There are a few settings behind that decision like multiple AZ, automatic backup etc. We will select testing for now. Next up there are a couple of options that are more fine grainted that before. Version, Licensing, db instance type, multi AZ, storage type (provisioned SSD, general purpose SSD, magnetic), storage allocation, backup are just a few to mention.

As a db instance identifier i used "cuba-ordermanagement-rds" and in db master username i used "cuba-ordermanagement" as well as the database name (in the next step).

In the last page of the wizard there are the VPC and subnet settings. You can choose the same VPC (should be only one right now) and subnet as your EC2 instance is in.

The security group defines what resources can access the db instance. Just choose "Create new security group" - we might change this later. THe attribute "publicly accessible" can be set to false, because only the EC2 instance need to access the instance.

The backup retention period can be set to "0" for now, because we will not have any data that would be worth taking backups of.

The configurability is very comprehensive. Additionally when you think about it, AWS allows a run-of-the-mill developer like me to create a DB cluster that is so powerful, secure and reliable with the effort of a 3 step wizard - this is quite empowering.

With this in place, the DB instance should start and be ready to go in a few seconds. After setting up the correct credentials in the task definition from above, it means we can actually deploy our application.

## Deploy application to ECS cluster

After everything is setup correctly, we can actually deploy the application to the ECS cluster.


The overview shows what we already achieved in the first part of the article as well as what we covered in this part:

<figure class="center">
	<a href="{{ site.url }}/images/cuba-on-aws-ecs-part-2/run-cuba-on-aws-overview-part-2.png"><img src="{{ site.url }}/images/cuba-on-aws-ecs-part-2/run-cuba-on-aws-overview-part-2.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-on-aws-ecs-part-2/run-cuba-on-aws-overview-part-2.png" title="run CUBA application of AWS with ECS overview">run CUBA application of AWS with ECS overview</a></figcaption>
</figure>

The arrow from the cool developer to the ECS service is what we will do next. The steps before this have all been requirements to do this. Nevertheless they allowed us, with just one shell script, activate a whole bunch of these arrows.

For doing that i created one main shell script and a few helper scripts.  The main script [deploy-to-ecs.sh](https://github.com/mariodavid/cuba-ordermanagement/blob/cuba-on-aws-ecs/deployment/ecs/scripts/deploy-to-ecs.sh) does the following things:

1. retrieve the application name and application version from the command line
2. checks if there is a docker image for the given application name and version in the ECR
3. uses the task definition template to register a new task definition revision
4. stops the ECS service
5. updates the ECS service task definition
6. starts the ECS service

For doing that i uses the AWS CLI as well as some pretty cool command line tool for json parsing: [JQ](https://stedolan.github.io/jq/) (some grep, sed and awk as well).

I'll not go into the details of this script, because first of all because there is some error handling and comments going on there which would blow up the listing here. Secondly it would be a little bit too much detail. The main point is that is uses the CLI of the cloud provider in order to automatically deploy your application.

<div class="information">
To be fair, the script relies on the naming conventions for the different parts. So in case you are stuck executing it, just write a comment below with the error message or open up your vim in order to see where the problem is.
</div>

Here are the main AWS CLI parts:

{% highlight bash %}

# ...
sed -e "s;%CUBA_VERSION%;${CUBA_VERSION};g" ${TEMPLATE_FILE} > CUBA-${CUBA_VERSION}.json
aws ecs register-task-definition --family ${TASK_FAMILY} --cli-input-json file://CUBA-${CUBA_VERSION}.json

# ...
aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --task-definition ${TASK_FAMILY}:${TASK_REVISION} --desired-count ${DESIRED_COUNT}

# ...
{% endhighlight %}

This is quite a straight forward way of API i would say.

When executing the script like this:

{% highlight bash %}
$ deployment/ecs/scripts/deploy-to-ecs.sh cuba-ordermanagement latest
{% endhighlight %}

it should use what is currently the latest Docker image(s) and deploy them straight to ECS. You can access them via http://public-ip-of-EC2-instance/ like this:

<figure class="center">
	<img src="{{ site.url }}/images/cuba-on-aws-ecs-part-2/cuba-start.png" alt="">
	<figcaption>The running CUBA app in a ECS cluster</figcaption>
</figure>

## Summary

So, what did we achieve in this second part of the article. We created the actual ECS building blocks with the cluster, the task definition and the service. During that we additionaly created the underlying EC2 and RDS insctances in order to have a runtime for the ECS cluster.

We saw the different options on how to communicate with the AWS cloud. In the initial phase we used the management UI to create the resources. Later in the script, we saw that everything that can created via the UI can also be created and edited via the AWS CLI.

The script gave you an easy opportunity to deploy latest release of your application to the alredy existing ECS cluster.

#### What's left?

As i wrote, this is a three part series on AWS and how to create a production like environment with ECS. When we look at the currently implemented cluster, there are a few things that could have been done better. This is what the last part will cover.

We will use ELB in order to load balance traffic to more then one web application. Then we will start to cluster (on a CUBA level) the application parts. There are a few gotchas to hop over when we want to cluster the middleware layer of CUBA on AWS.
In case you have different security requirements on the different parts of your application, you might also be interested in the different VPC features of AWS and how we can arrange our ECS cluster so that certain parts of the application layers are not internet accessible.

So, with this i would like to close this second part and hope you enjoyed it. If you have any comments, feedback or opinions on the article, please let me know about it.
