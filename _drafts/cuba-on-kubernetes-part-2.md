---
layout: post
title: CUBA on Kubernetes - Part 2
description: "In this blog post, we will discover the insights of CUBA's Security subsystem and how different business security requirements can be achieved."
modified: 2018-08-03
tags: [cuba, kubernetes, deployment]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

Kubernetes has become the de-facto standard when it comes to doing container scheduling. Since it is no omnipresent these days, let's have a look on how to deploy a CUBA application into a Kubernetes cluster. In the second part we will continue with deploying the application to the existing infrastructure.

<!-- more -->


### Step 3: Upload your CUBA app to Google cloud

In order to run our CUBA app on GKE, we need to package it as a Docker container. CUBA itself has a neat [documentation](https://doc.cuba-platform.com/manual-6.8/_gradle_plugin_for_docker.html) for doing that. In comparison to previous approaches I will this time do it all via gradle. You can take a look at the complete [build.gradle](https://github.com/mariodavid/cubarnetes/blob/master/build.gradle) in the example application [mariodavid/cubarnetes](https://github.com/mariodavid/cubarnetes). Here are the relevant parts:


{%highlight groovy%}
buildscript {
    // ...
    dependencies {
        classpath "com.haulmont.gradle:cuba-plugin:$cubaVersion"
        classpath 'com.bmuschko:gradle-docker-plugin:3.2.5'
    }
}

import com.bmuschko.gradle.docker.tasks.image.Dockerfile
import com.bmuschko.gradle.docker.tasks.image.DockerBuildImage
import com.bmuschko.gradle.docker.tasks.image.DockerPushImage
import com.bmuschko.gradle.docker.DockerRegistryCredentials

// ...

apply(plugin: 'cuba')

apply plugin: 'com.bmuschko.docker-remote-api'

cuba {
    // ...
}

// ...

task wrapper(type: Wrapper) {
    gradleVersion = '4.3.1'
}

task buildUberJar(type: CubaUberJarBuilding) {
    singleJar = true
    logbackConfigurationFile = 'etc/uber-jar-logback.xml'
    coreJettyEnvPath = 'modules/core/web/META-INF/jetty-env.xml'
    appProperties = ['cuba.automaticDatabaseUpdate' : true]
}


task createDockerfile(type: Dockerfile, dependsOn: buildUberJar)  {
    destFile = project.file('build/distributions/uberJar/Dockerfile')
    from 'openjdk:8-alpine'
    addFile("cubarnetes.jar", "/usr/src/cuba/app.jar")
    defaultCommand("java", "-Dapp.home=/usr/src/cuba/home", "-jar", "/usr/src/cuba/app.jar")
}

task buildImage(type: DockerBuildImage, dependsOn: createDockerfile) {
    inputDir = createDockerfile.destFile.parentFile
    tags = ['cubarnetes', 'gcr.io/cuba-on-kubernetes/cubarnetes']
}

{%endhighlight%}

Additionally you need to create a [uber-jar-logback.xml](https://github.com/mariodavid/cubarnetes/blob/master/etc/uber-jar-logback.xml) and a [jetty-env.xml](https://github.com/mariodavid/cubarnetes/blob/master/modules/core/web/META-INF/jetty-env.xml) in your project. Those files can be generated through CUBA studio.

I changed the jetty-env.xml file slightly so that jetty uses ENV variables to get the JDBC URL & credentials:

{%highlight xml%}
  <Set name="url"><Env name="DB_URL"/></Set>
  <Set name="username"><Env name="DB_USER"/></Set>
  <Set name="password"><Env name="DB_PASS"/></Set>
{% endhighlight %}


With that in place, to create a docker image locally, use:

{%highlight bash%}
$ ./gradlew buildImage
{% endhighlight %}

to create the image. You can verify that it succeded and created one images with two different tags via:

{%highlight bash%}
$ docker images | grep cubarnetes
cubarnetes                                                            latest              3ba5a7da458f        2 days ago          179MB
gcr.io/cuba-on-kubernetes/cubarnetes                                  latest              3ba5a7da458f        2 days ago          179MB
{% endhighlight %}

In order to push a docker image to Google cloud, we have to authenticate the docker command against the GCR repository. Destails can be found in the [docs](https://cloud.google.com/container-registry/docs/pushing-and-pulling), but the main point is to use another <code>gcloud</code> command to do it:

{%highlight bash%}
$ gcloud auth configure-docker
{% endhighlight %}

Since we tagged the docker image as "gcr.io/cuba-on-kubernetes/cubarnetes" the US based registry will be used (see docs for other locations).

To push the image up to the registry do a:
{%highlight bash%}
$ docker push gcr.io/cuba-on-kubernetes/cubarnetes
{% endhighlight %}

With that, we have a running cluster and a software that we want to run on GKE pre-configured


### Step 4: Create Kubernetes deployment

With all of the above in place, we have everything setup for successfully deploying the CUBA application through Kubernetes.

The way Kubernetes is deployed is through a deployment YAML file. You can find this file (deployment.yml) in the [CUBArnetes repository](https://github.com/mariodavid/cubarnetes/blob/master/deployment.yml).

It looks like this:

{% highlight yaml %} 
---
apiVersion: "v1"
kind: "ConfigMap"
metadata:
  name: "cubarnetes-app-config"
  namespace: "default"
  labels:
    app: "cubarnetes-app"
data:
  DB_URL: "jdbc:postgresql://127.0.0.1:5432/postgres"
---
apiVersion: "extensions/v1beta1"
kind: "Deployment"
metadata:
  name: "cubarnetes-app"
  namespace: "default"
  labels:
    app: "cubarnetes-app"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: "cubarnetes-app"
  template:
    metadata:
      labels:
        app: "cubarnetes-app"
    spec:
      containers:
      - name: "cubarnetes"
        image: "gcr.io/cuba-on-kubernetes/cubarnetes:latest"
        env:
        - name: "DB_URL"
          valueFrom:
            configMapKeyRef:
              key: "DB_URL"
              name: "cubarnetes-app-config"
        - name: "DB_USER"
          valueFrom:
            secretKeyRef:
              name: cloudsql-db-credentials
              key: username
        - name: "DB_PASS"
          valueFrom:
            secretKeyRef:
              name: cloudsql-db-credentials
              key: password
      - name: cloudsql-proxy
        image: gcr.io/cloudsql-docker/gce-proxy:1.11
        command: ["/cloud_sql_proxy",
                  "-instances=cuba-on-kubernetes:europe-west1:cubarnetes-postgres=tcp:5432",
                  "-credential_file=/secrets/cloudsql/credentials.json"]
        volumeMounts:
          - name: cloudsql-instance-credentials
            mountPath: /secrets/cloudsql
            readOnly: true
      volumes:
        - name: cloudsql-instance-credentials
          secret:
            secretName: cloudsql-instance-credentials
---
apiVersion: "autoscaling/v1"
kind: "HorizontalPodAutoscaler"
metadata:
  name: "cubarnetes-app-hpa"
  namespace: "default"
  labels:
    app: "cubarnetes-app"
spec:
  scaleTargetRef:
    kind: "Deployment"
    name: "cubarnetes-app"
    apiVersion: "apps/v1beta1"
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80
---
apiVersion: "v1"
kind: "Service"
metadata:
  name: "cubarnetes-app-service"
  namespace: "default"
  labels:
    app: "cubarnetes-app"
spec:
  ports:
  - protocol: "TCP"
    port: 8080
    targetPort: 8080
  selector:
    app: "cubarnetes-app"
  type: "LoadBalancer"
  loadBalancerIP: ""
{% endhighlight %}

Let's go through it piece by piece. I will not cover all the details, because this is better suited for the documentation of Kubernetes itself, but we will go over the main parts.

The different parts are separated by "---". Let's look at the first part - the ConfigMap.

#### Deployment description
A ConfigMap is used to put configuration options of the deployment to a dedicated place. This way the configurable pieces are held together and are not scattered throughout the whole deployment file.

The next thing defines the main part of the deployment (kind: "Deployment").

In this section the pods (and therefore the containers) are described. The container reference a Docker image (e.g. <code>gcr.io/cuba-on-kubernetes/cubarnetes:latest</code>) which will be used. 

Furthermore the environment variables of the container are defined. Above I said that I externalized the three parameters <code>DB_URL</code>, <code>DB_USER</code> and <code>DB_PASS</code> from the container image. This is the point where the vaules are injected into the containers.

As we do not want to hard-code the credentials in our deployment.yml file, the file references the secrets mechanism from Kubernetes which we put our secrets before.

#### Side car container for connecting to the DB

One thing to mention is the fact, that there is not only one container defined (the CUBA application). Instead two containers are running in it. The second container with the name <code>cloudsql-proxy</code> is a side-car container, which acts as a proxy for the DB connection. Instead of letting the CUBA application directly talk to to DB, it will use the side-car container to actually connect to it. This approach is pretty common, as it allows to seperate different concerns. On the one hand, the application can assume it can always connect to the DB by "localhost:5432". The indirection of the side-car will forward the requests & takes care of finding the correct cloud SQL db. Also it is an easy point for logging and so on.


#### autoscaling through HorizontalPodAutoscaler

The next section creates a "HorizontalPodAutoscaler" in order to automatically scale up and down the amount of parallel containers of our application. There is a CPU threshold defined in the description so that once this threshold is reached, new pods will get started / destroyed.

#### exposing the app through a Kubernetes service

The last remaining part is to create a service, which will take care of exposing the application (in particular a defined TCP port) to the outside world. In this case I used a "LoadBalancer" to achieve this.

This is just a very brief overview and honestly, this is where the Kubernetes magic starts to begin. But for the sake of simplicitly I will leave it at this point with this basic configuration. You can dig much deeper into the configuration options of a kubernetes deployment.

#### deploying the application with "kubectl"

{% highlight bash %} $ kubectl apply -f deployment.yml
{% endhighlight %}