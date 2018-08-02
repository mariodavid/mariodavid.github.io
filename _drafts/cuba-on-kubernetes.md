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


### Step 1: setting up a kubernetes cluster on GKE

However, we will use GKE in this blog post. This means, that you have to prepare certain stuff for using it. In the [quick-start guide](https://cloud.google.com/kubernetes-engine/docs/quickstart) you will find all relevant information on how to create an account, create a project, install the command line tools.
