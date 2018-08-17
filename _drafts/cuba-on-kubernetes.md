---
layout: post
title: CUBA on Kubernetes
description: "In this blog post, we will discover the insights of CUBA's Security subsystem and how different business security requirements can be achieved."
modified: 2018-08-03
tags: [cuba, kubernetes, deployment]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

Kubernetes has become the de-facto stanrad when it comes to doing container scheduling. Since it is no omnipresent these days, let's have a look on how
to deploy a CUBA application into a Kubernetes cluster.

<!-- more -->


### Why to look at Kubernetes

When it comes to deployment of an application living in a docker container, a lot
has changed in the recent year. Back in 2016 when I wrote another container related blog post series dealing with running CUBA on ECS - the propritaery elastic container service from AWS, the container orchestration wars were at its all time high. Back then the main competitors were Docker Swarm & Apache Mesos. But even then Kubernetes stood out for its great and vibrant community.

Fast forward to mid 2018 - Kubernetes has mainly got adopted by all major cloud vendors, which have hosted kubernetes offerings and took Kubernetes more or less as the basis for their container workloads. Also the ecosystem around kubernetes itself as well as the CNCF in general is very mature and growing like crazy.

But not only that, also previous competitors like Openshift and Rancher (or at least software solving similar problems) switched to Kubernetes as their underpinning. Additionally a lot of CI/CD tools like Gitlab or GoCD and even [Jenkins](https://jenkins-x.io/) have a tight integration into Kubernetes as their platform for deploying applications through the pipeline.

There are many voices saying that in comparison to PaaS which never good the popularity and adoption to become ubiquitous in the deployment sphere, Kubernetes
is on its way to become THE common abstraction for deployment and infrastructure.


This all leads me to take a deeper look into Kubernetes and with this blog post
I will show you how to deploy a CUBA application onto kubernetes.

### Step 0: selecting a compute provider

In order to get up and running with Kubernetes, we have to choose which compute provider should be used. Kubernetes in general can run on premise as well as in the cloud. Since setting up (as well as maintaining) a Kubernetes can be a challenging task, most of the cloud providers offer a hosted environment for Kubernetes.

Normally this means, that the only resources that the user has to care about are the worker nodes. The cluster management is taken care of by Google, AWS, Digitalocean, Azure and so on.

In this tutorial we will leverage the Google Kubernetes enginge (GKE) in order to setup our Kubernetes cluster.

Just to remember - compared to the AWS ECS blog post series I did, here is the main difference: With Kubernetes you do not lock in into a particular vendor and its propritaery container orchestrator.

Instead you have a certain lock-in into an open source tool (kubernetes), which still is a lock-in, but it is much more universal. As I said earlier, you have the freedom of choice on which Cloud provider you want to run Kubernetes, hosted or self managed, on premise, in the cloud or even on the [edge](https://azure.microsoft.com/de-de/blog/manage-azure-iot-edge-deployments-with-kubernetes/).

This means that the choice of GKE as the provider for the Kubernetes cluster is more or less irrelevant compared to picking ECS, because you can pretty much lift & shift your kubernetes workloads from one cloud provider to another.


### Step 1: setting up a Kubernetes cluster on GKE

However, we will use GKE in this blog post. This means, that you have to prepare certain stuff for using it. In the [quick-start guide](https://cloud.google.com/kubernetes-engine/docs/quickstart) you will find all relevant information on how to create an account, create a project, install the command line tools.

What is also relevant for this blog post is the installation of [Terraform](https://www.terraform.io/). Terraform is a tool which allow to declarativly describe infrastructure in a infrastructure as code style. This allows us instead of creating the cluster through the Google cloud API / CLI / UI, describe the cluster through Terraform. The tool will then go ahead and do the heavy lifting of the interaction with the cloud provider.

Using Terraform has the same advantages regarding reducing the vendor lock-in. Although it does not hide away the feature of a concrete cloud provider (as Kubernetes somewhat does), it still homogenizes the sytax of managing infrastructure resources. Compared to programmatic tools like Chef or puppet, Terraform takes a declarative approach in order to describe the destination state of the infrastructure.

In order to let Terraform interact with the Google Kubernetes API, we need to allow this in the [Google cloud console](https://console.developers.google.com/apis/api/container.googleapis.com/overview). After the Kubernetes API is activated for this project, the only thing left before starting off with Terraform is to create a JSON based Token for Terraform to authenticate.
In the “Credentials” section of the console, choose “Create Credentials” and then “Service account key”. In the selection for "Service account" you can choose "Compute Engine service account". Use Key Type "JSON". This will give you a JSON file, which contains all necessary information to connect to GKE via Terraform.

#### Creating a GKE cluster through Terraform

Once everything is setup, let's create the kubernetes cluster.

As an example for doing that, I created a repository which contains the infrastructure code for this blog post: [cuba-on-kubernetes-infrastructure](https://github.com/mariodavid/cuba-on-kubernetes-infrastructure).

Is uses an existing Terraform module called [kubernetes-engine](https://registry.terraform.io/modules/google-terraform-modules/kubernetes-engine/google/1.15.0), which will create a Kubernetes cluster for GKE on our behalf. It more or less will take away another set of heavy lifting operations from us.

Terraform modules are a way to encapsulate certain infrastructure resource definitions through a abstraction mechanism with its own API. It allows you to define modules for reoccuring patterns of infrastructure that should be used. The [Terraform module registy](https://registry.terraform.io/) then is an easy way to share common terraform modules just like the Docker registry for container images.


{% highlight terraform %}
module "cuba_on_kubernetes_cluster" {
  source = "google-terraform-modules/kubernetes-engine/google"
  version = "1.15.0"

  general = {
    name = "cuba-on-kube"
    env  = "prod"
    zone = "europe-west1-b"
  }

  master = {
    username = "admin"
    password = "${random_string.password.result}"
  }

  default_node_pool = {
    node_count = 3
    remove     = false
  }

}
{% endhighlight %}

When we look at this usage of the module, we find certain parameters that we can pass into the module. In the <code>general</code> section, you can define the name of the cluster, the environment and the zone the cluster should be located in.

Then there are certain other options which can be configured, but I will not go through right now. This can be looked up in the documentation of this [module](https://registry.terraform.io/modules/google-terraform-modules/kubernetes-engine/google/1.15.0).


<div class="well">
Note: In order to have a production ready Kubernetes cluster, you should be aware how your cluster is configured. Altough the Terraform module is an API abstraction, at the end of the day it is key to understand the infrastructure and the possible noobs you are running in a production environment.</div>
