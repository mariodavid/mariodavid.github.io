---
layout: post
title: CUBA with DynamoDB
description: "In this blog post, we will discover the insights of CUBA's Security subsystem and how different business security requirements can be achieved."
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

In a lot of cases, a relational SQL based database will not fit the needs of the data access use-case. 

<!-- more -->

In this blog post we will try to connect a CUBA app with DynamoDB. But before going into the technical details on how to do this, let's take a look on why we might want to use DynamoDB in the first place.


## What is DynamoDB and why would you matter

DynamoDB is hosted database offering from AWS. In the categorization of NoSQL databases it would probably fit best in the category of "Key-Value stores". It is schemaless, meaning, that every entry in the Table might have a different structure.

There are many reasons why to chose a different (NoSQL) database like DynamoDB, but let's look at some major ones which are especially true for DynamoDB:

#### NoSql & NoOps

First, you as the application developer don't need to care about infrastructure at all. In DynamoDB the only thing you care about is the configuration of your table. In particular, you define two values called "read capacity" and "write capacity" which are abstract values for the compute power you expect. 

Besides that there is literally nothing that you can or need to define. No servers, load balancing, storage configurations, high availablity etc. Everything will be taken care for you.

#### Schemaless

DynamoDB, as said above belongs to the category of key-value stores. This means that the access patterns are a little bit more specific compared to general SQL databases. You mostly access the data by its primary key. Additionally you can query by specific additional indices, that have to be configured upfront (like a index in a relational database).

It is possible to query by other attributes, but this requires some kind of full-table scan, which fetches all items and then applies the filtering afterwards. This seems pretty limiting from a relational DB background (although similar constraints exists when having no indices configured). 

However, this is the prerequisite to create a lightning fast storage engine, which scales across hundreds of servers. Although Amazon hides that fact from the user of this database, DynamoDB is based on the idea of a peer to peer network of servers that work hand in hand to provide data access. This way Amazon is capable of offering these very high kinds of SLAs that outcompete every standard relational database offering (and even more: self hosting).

#### Horizontal scalability

Relational databases can be clustered across a very limited amount of nodes and due to their promise to be transactional & consistent. Relational databases can also work in a leader / follower fashion which softs the consistency so that the followers are informed asynchronously about changes in the leader. 

But, to summarize, there is a fundamental problem in horizontal scaling for relational databases - and this is just another point where a NoSQL store like DynamoDB shines.

## Interacting with AWS services from Java

To interact with the amazon webservice offerings in general through Java, the easiest way is to use their official SDK.

To make that happen in a CUBA application, you can define a dependency in your <code>build.gradle</code>:

{% highlight groovy %}
configure(coreModule) {
    // ...

    dependencies {
        // ...
        compile group: 'com.amazonaws', name: 'aws-java-sdk-dynamodb', version: '1.11.251'
    }

configure(globalModule) {
    task enhance(type: CubaEnhancing)

    dependencies {
        compile group: 'com.github.derjust', name: 'spring-data-dynamodb', version: '4.5.0'
    }
}
{% endhighlight %}

The core module will use the AWS SDK, to interact with the DynamoDB service. As we will use Entity annotations later, we will  add the [Spring Data support for Dynamo DB](https://github.com/derjust/spring-data-dynamodb) to the global module.

#### AWS Account setup

The next thing is that you have to have a AWS account and programmatic access configured with access key and secret key. As I will not guide you through this process, here are the required information to get up and running:

* [starting with AWS](aws.amazon.com/free) (Account creation etc.)
* [How do I create an AWS access key?](https://aws.amazon.com/de/premiumsupport/knowledge-center/create-access-key/)

Once everything is configured correctly, you are ready to use CUBA with Dynamo DB.

I created an example application, where you can take a look at the configuration: [cuba-example-dynamodb-access](https://github.com/mariodavid/cuba-example-dynamodb-access).

#### Wiring CUBA to AWS

You have to tell the CUBA app which credentials to use. To do that, configure the credentials in the <code>app.properties</code> file:

{% highlight properties %}
###############################################################################
#                                  AWS                                        #
###############################################################################

# Configure your AWS region, if not using US-EAST-1
# see: https://docs.aws.amazon.com/en_en/general/latest/gr/rande.html#ddb_region
amazon.dynamodb.endpoint = dynamodb.us-east-1.amazonaws.com

# AWS credentials
amazon.aws.accesskey = <<YOUR_AWS_ACCESS_KEY>>
amazon.aws.secretkey = <<YOUR_AWS_SECRET_KEY>>
{% endhighlight %}


Another, more secure way is to not store the credentials in source code. Instead use system variables to pass it into the running CUBA application. Everything else, would be the same. 
Just run your CUBA app with 

<code>-Damazon.aws.accesskey=ACCESS_KEY&nbsp;-Damazon.aws.secretkey=SECRET_KEY</code>

After that, we can use these values for creating the corresponding Spring beans for interacting with DynamoDB. Define the following beans in your <code>spring.xml</code>:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dynamodb="http://docs.socialsignin.org/schema/data/dynamodb"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
                           http://docs.socialsignin.org/schema/data/dynamodb
                           http://docs.socialsignin.org/schema/data/dynamodb/spring-dynamodb.xsd">

    <!-- Annotation-based beans -->
    <context:component-scan base-package="com.rtcab.cedda"/>

    
    <!-- AWS credentials -->
    <bean id="amazonAWSCredentials" class="com.amazonaws.auth.BasicAWSCredentials">
        <constructor-arg value="${amazon.aws.accesskey}" />
        <constructor-arg value="${amazon.aws.secretkey}" />
    </bean>
    
    
    <!-- the AWS DynamoDB client -->
    <bean id="amazonDynamoDB" class="com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient">
        <constructor-arg ref="amazonAWSCredentials" />
        <property name="endpoint" value="${amazon.dynamodb.endpoint}" />
    </bean>


    <!-- repositories scan for spring data dynamo db -->
    <dynamodb:repositories base-package="com.rtcab.cedda.core.repository" amazon-dynamodb-ref="amazonDynamoDB" />

</beans>
{% endhighlight %}

First there is the <code>amazonAWSCredentials</code> bean which reads the just configured application properties either from the app.properties file or from the system properties (<code>-DmyVar=myVal</code>).

Next, the <code>amazonDynamoDB</code> uses the credentials and the dynamo db endpoint to create a dynamo db client.

Lastly, we enable Spring Data repositories scan for the given package.


With this the configuration of wiring those two technologies togeter should be done. Now it is possible to create the code, which stores & loads data from dynamo.

## Creating a DynamoDB Entity

There are several injection points where we could branch our code to interact with the dynamo db database instead of the standard relational database. We could call the API from the UI directly, or using a Service that will be used in a custom datasource of CUBA. In this case, I decided to take the lowest CUBA injection point available: A custom datastore.

