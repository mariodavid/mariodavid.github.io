---
layout: post
title: Run CUBA on AWS ECS - Part 3
description:
modified: 2016-08-09
tags: [cuba, AWS, ECR, cloud, ECS]
image:
  feature: cuba-on-aws-ecs-part-3/feature.jpg
  feature_source: https://pixabay.com/de/alaska-gletscher-eis-bergen-810433/
---

...

<!-- more -->

### Cloud watch logs for central container logging

Instead of logging into the EC2 instance to get the Docker container logs we will instead use [AWS cloud watch logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/CWL_GettingStarted.html). ECS has integration to cloud watch logs. It takes STDOUT and STDERR from the Docker container and pushes it into the logging system, where we can easily retrieve the information from Tomcat in a web UI. To get that going the only thing to do is to [create](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logs:) a log group in your preferred AWS region. Then you define the name of the log group in the JSON key*awslogs-group*. After that the logs will be sent to the central logging system.
