---
layout: post
title: Run CUBA on AWS ECS - Part 1
description:
modified: 2016-08-02
tags: [cuba, AWS, ECR, cloud, ECS]
image:
  feature: cuba-on-aws-ecs/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
---

Going from running Docker in the command line to a production scenario can be quite challenging since there is so much more to cover and so much more possibilities to do it right. One solid way of Docker and the Cloud is AWS.

In the next three articles i'll go through the different possibilities AWS has to offer especially regarding Container as a Service. We will deploy the [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) CUBA app on an ECS cluster and use different features of the AWS cloud to leverage the full cloud potential to CUBA.

<!-- more -->

The three parts of the article will cover the following content of the overall topic:

1. introduction to AWS services, creating the docker image and pushing it to ECR
2. creating a simple ECS cluster and running the cuba app on it
3. using different AWS features to extend the ECS cluster towards HA, cluster the different CUBA layers independently


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

ECS is the last building block we will use in this articles. As described above, it is the solution from AWS for Container orchestration. We'll go into much more detail about this topic because we need to configure the different pieces within ECS, but for know it can be seen as the part that automatically deploys Docker containers to existing EC2 instances and cares about the healthiness of the containers.

## Overview on the deployment process

To give you a general idea of what scenarios will be covered in the following articles, here's an overview diagram:

<figure class="center">
	<a href="{{ site.url }}/images/cuba-on-aws-ecs/run-cuba-on-aws-overview.png"><img src="{{ site.url }}/images/cuba-on-aws-ecs/run-cuba-on-aws-overview.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-on-aws-ecs/run-cuba-on-aws-overview.png" title="run CUBA application of AWS with ECS overview">run CUBA application of AWS with ECS overview</a></figcaption>
</figure>

The diagram shows the different building blocks of the AWS infrastructure that are relevant for the ECS deployment. I'll describe on a very broad basis the workflow and we'll go into much more details of the different steps afterwards.

First either the developer or your favorite CI system kicks off the deployment process. To do this, the Docker image get build locally and pushed to the central Docker repository. Next the elastic container service (ECS) gets notified to redeploy the newly created Docker image.

Since ECS is just the orchestration layer but not responsible for actually running the Docker containers, pre configured EC2 instances are contacted in order to redeploy.

To ensure fault tolerance on the EC2 instance level as well as the Docker container level, multiple EC2 instances serve the multiple instances of the Docker image. Thus there is a need for a load balacing mechanism that will shield the docker containers from direct internet connection, terminate SSL and balance requests between the instances.

For database access of the CUBA application the deployment uses a postgres RDS instance that is clustered on a cross availability zone basis. An availability zone or a AZ as called in AWS is basically a data-center. The biggest unit of computing on AWS is a region (like eu-west-1: EU Ireland). Within a region there are multiple AZs that are fully isolated but have a high bandwith connection between them (to allow synchron database cluster e.g.). More information can be found at the [AWS AZ docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html).

After every part of the diagram has been talked through very slightly, we will have a look at the different steps and how to implement them in order to deploy our CUBA application to ECS in the next articles. In this first article we will build the application and deploy the binaries to ECR. This will be the requirement to take a look at how to create a ECS cluster (second part) so that we are able to deploy the application in the ECS cluster (third part)

For this first part we cover the image building and registry configuration and deployment.

### Build cuba-ordermanagement Docker image

The first step towards this is to actually deploy the docker image to ECR.
Based on the docker image i created in the initial [docker cuba blog post](https://www.road-to-cuba-and-beyond.com/put-a-island-into-a-box-how-to-dockerize-your-cuba-app/) here's a slightly changed Dockerfile. First of all i in fact created two different Dockerfiles. The reason is that for every application component we want to create a Docker container that we can create and deploy independently.


{% highlight dockerfile %}
### Dockerfile

FROM tomcat:8-jre8

# ...

ADD container-files/war/app-core.war /usr/local/tomcat/webapps/ROOT.war

# ...

ADD container-files/start.sh /start.sh
RUN chmod +x /start.sh

# ...

CMD ["/start.sh"]


{% endhighlight %}

This is just a subpart of the Dockerfile. In the [repository](https://github.com/mariodavid/cuba-ordermanagement/blob/cuba-on-aws-ecs/deployment/docker-image/app-core/Dockerfile) you'll find the whole listing. In comparison to the original Dockerfile i created a startup file <code>start.sh</code> that basically ensures certain environment variables are available and copy the values to the tomcat installation so that the CUBA application is configured correctly. This is done via checking for existence of environment variables:

{% highlight bash %}

: ${CUBA_DB_HOST?"CUBA_DB_HOST required (docker run -e 'CUBA_DB_HOST=myHost)'"}
echo "db.host=${CUBA_DB_HOST}" >> /usr/local/tomcat/conf/catalina.properties

{% endhighlight %}

In case the Docker container is not started with <code>docker run -e CUBA_DB_HOST=dbServer</code> the container will complain about the absense of the variable and stop the server. If the variable is set, the value will be copied to catalina.properties so that the CUBA application is configured via the environment variables.

After the image is described correctly, it can be build locally and deployed to the central AWS Docker registry (ECR) afterwards. To build the Docker image i created the shell script [docker-build.sh](https://github.com/mariodavid/cuba-ordermanagement/blob/cuba-on-aws-ecs/deployment/docker-build.sh) which lets us build both images. It mainly does the following steps (example for the app.war):

{% highlight bash %}

./gradlew buildWar
cp build/distributions/war/app.war deployment/docker-image/app/container-files/war/
docker build -t cuba-ordermanagement-app:latest deployment/docker-image/app/

{% endhighlight %}

The script takes three parameters to build the image:

1. the name of the application (cuba-ordermanagement)
2. the component name (app / app-core / app-portal)
3. the docker tag / version of the component (latest / 0.1 / 0.2...)

The resulting call from the root of the project for the cuba-ordermanagement application should look like this:

{% highlight bash %}

$ deployment/docker-build.sh cuba-ordermanagement app-core latest
$ deployment/docker-build.sh cuba-ordermanagement app latest

{% endhighlight %}

After this, the Docker images should be created and listed via <code>docker images</code>:
{% highlight bash %}

$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
cuba-ordermanagement-app        latest              e5e0498669dc        2 minutes ago        388.7 MB
cuba-ordermanagement-app-core   latest              2446085b946a        2 minutes ago        380.8 MB

{% endhighlight %}

### Deploy docker image to ECR

Next up the last step is to actually transfer the Docker image to ECR.

<!--<div class="information">-->
There have to be a few steps have to be taken before you can communicate with ECR. Detailed information about this can be found in the <a href="http://docs.aws.amazon.com/AmazonECR/latest/userguide/ECR_GetStarted.html">ECR getting started guide</a>. Here are the main points:

1. create a AWS account
2. follow the <a href="http://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html">ECS setup instructions</a>
3. create a ECR registry for each component (app-core and app)
4. install the <a href="http://docs.aws.amazon.com/cli/latest/userguide/installing.html">AWS CLI</a>
5. login to ECR via the Docker CLI: <code>$(aws ecr get-login)</code>

It took qiute some time at least for me to go through the different steps, but this is just a one time effort in order to get going with AWS. Additionally, a lot of steps are requirements for ECS as well.

I created another shell script called [docker-deploy-to-ecr.sh](https://github.com/mariodavid/cuba-ordermanagement/blob/cuba-on-aws-ecs/deployment/ecs/docker-deploy-to-ecr.sh)
which will to the job for us. Since ECR is quite seamlessly integrated into the normal docker workflow, the script just uses the normal docker commands <code>docker tag</code> and <code>docker push</code> to transfer the images to the repository. It takes the same arguments as the last script but one additional parameter: "ECR_REGISTRY_HOST".

With this commands you will transfer the local docker images to the AWS ECR repository:

{% highlight bash %}

$ deployment/ecs/docker-deploy-to-ecr.sh cuba-ordermanagement app-core latest 123456789101.dkr.ecr.us-east-1.amazonaws.com/cuba-ordermanagement-app-core
$ deployment/ecs/docker-deploy-to-ecr.sh cuba-ordermanagement app latest 123456789101.dkr.ecr.us-east-1.amazonaws.com/cuba-ordermanagement-app

{% endhighlight %}

When you have a look at your ECR repository list, both repositories should host at least one Docker image now.

With this steps inplace you are ready to take on the next major topic. In this we will take what we have achieved, that our application is available to the ECS environment so that we can now create a ECS cluster that will run the docker containers.

This will be covered in the second part of this three part series about AWS ECS.
