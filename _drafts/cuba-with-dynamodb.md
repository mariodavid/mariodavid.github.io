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

In this blog post, we will try to connect a CUBA app with DynamoDB. But before going into the technical details on how to do this, let's take a look at why we might want to use DynamoDB in the first place.


## What is DynamoDB and why would you matter

DynamoDB is a hosted database offered by AWS. In the categorization of NoSQL databases it would probably fit best in the category of "Key-Value stores". It is schemaless, meaning that every entry in the Table might have a different structure.

There are many reasons why to chose a different (NoSQL) database like DynamoDB, but let's look at some major ones which are especially true for DynamoDB:

#### NoSql & NoOps

First, you as the application developer don't need to care about infrastructure at all. In DynamoDB the only thing you care about is the configuration of your table. In particular, you define two values called "read capacity" and "write capacity" which are abstract values for the compute power you expect.

Besides that, there is literally nothing that you can or need to define. No servers, load balancing, storage configurations, high availability etc. Everything will be taken care for you.

#### Schemaless

DynamoDB, as said above, belongs to the category of key-value stores. This means that the access patterns are a little bit more specific compared to general SQL databases. You mostly access the data by its primary key. Additionally, you can query by specific additional indices that have to be configured upfront (like an index in a relational database).

It is possible to query by other attributes, but this requires some kind of full-table scan, which fetches all items and then applies the filtering afterwards. This seems pretty limiting from a relational DB background (although similar constraints exist when having no indices configured).

However, this is the prerequisite to creating a lightning-fast storage engine which scales across hundreds of servers. Although Amazon hides that fact from the user of this database, DynamoDB is based on the idea of a peer to peer network of servers that work hand in hand to provide data access. This way Amazon is capable of offering these very high kinds of SLAs that outcompete every standard relational database offering (and even more: self-hosting).

#### Horizontal scalability

Relational databases can be clustered across a very limited amount of nodes and due to their promise to be transactional & consistent. Relational databases can also work in a leader/follower fashion which softens the consistency so that the followers are informed asynchronously about changes in the leader.

But, to summarize, there is a fundamental problem in horizontal scaling for relational databases - and this is just another point where a NoSQL store like DynamoDB shines.

## Interacting with AWS services from Java

To interact with the Amazon Web Services offerings in general through Java, the easiest way is to use their official SDK.

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

The core module will use the AWS SDK, to interact with the DynamoDB service. As we will use Entity annotations later, we will add the [Spring Data support for Dynamo DB](https://github.com/derjust/spring-data-dynamodb) to the global module.

#### AWS Account setup

The next thing is that you have to have an AWS account and programmatic access configured with access key and secret key. As I will not guide you through this process, here are the required information to get up and running:

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


Another, more secure way is to not store the credentials in the source code. Instead, use system variables to pass it into the running CUBA application. Everything else would be the same.
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

First, there is the <code>amazonAWSCredentials</code> bean which reads the just configured application properties either from the app.properties file or from the system properties (<code>-DmyVar=myVal</code>).

Next, the <code>amazonDynamoDB</code> uses the credentials and the dynamo db endpoint to create a dynamo db client.

Lastly, we enable Spring Data repositories scan for the given package.


With this, the configuration of wiring those two technologies together should be done. Now it is possible to create the code which stores & loads data from Dynamo.

## Creating a DynamoDB Entity

There are several injection points where we could branch our code to interact with the dynamo DB instead of the standard relational database. We could call the API from the UI directly or using a Service that will be used in a custom datasource of CUBA. In this case, I decided to take the lowest CUBA injection point available: a custom datastore.


#### A non-persistent entity for customer
First thing is, that we need to create an entity for our data that we want to store. In this case, we will use the good old Customer entity. In Studio, when creating an entity, it will ask you about the entity type. In this case, we will select "Not persistent", because we want to configure a custom datastore for it. This is only possible if we chose "Not persistent" - although, in fact, we will persist is, just not in the relational database.

We can configure some attributes of this entity, like name, firstName, and email and store the id.

Next thing is, that we define the custom datastore. It is possible to do this in Studio, but I thought it might be a good idea to take a look at the source code files, as this is actually a fairly straightforward way of configuring it directly in code.

First, we go to <code>app.properties</code> and add the following two lines:

{% highlight properties %}
cuba.additionalStores = customerDynamoDb
cuba.storeImpl_customerDynamoDb = cedda_customerDynamoDbDataStore
{% endhighlight %}

The first line registers a new custom store, which is called "customerDynamoDb". Next, we point the store implementation of "customerDynamoDb" to a specific Spring bean: "cedda_customerDynamoDbDataStore", which we will create in just a bit.

This makes the CUBA core application aware of the datastore.

Additionally, we need to tell the frontend of CUBA about it as well - in <code>web-app.properties</code>:

{% highlight properties %}
cuba.additionalStores = customerDynamoDb
{% endhighlight %}

As the frontend does not need to know about the implementation, we will just tell it that there is an additional store that can be referenced.

The last thing (on the configuration/wiring front) that is necessary to do is to connect the Customer entity to this custom data store.

For non-persistent Entities there is a file called <code>metadata.xml</code>. This file registeres non-persistent entities to the CUBA runtime.

As you already created the entity as a non persistent one, the file should contain an entry for the Customer entity:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<metadata xmlns="http://schemas.haulmont.com/cuba/metadata.xsd">
    <metadata-model namespace="cedda"
                    root-package="com.rtcab.cedda">
        <class>com.rtcab.cedda.entity.Customer</class>
    </metadata-model>
</metadata>
{% endhighlight %}

To connect the entity to the custom store, just reference the name of the custom store in the class tag as the store attribute:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<metadata xmlns="http://schemas.haulmont.com/cuba/metadata.xsd">
    <metadata-model namespace="cedda"
                    root-package="com.rtcab.cedda">
        <class store="customerDynamoDb">com.rtcab.cedda.entity.Customer</class>
    </metadata-model>
</metadata>
{% endhighlight %}

### Using the Dynamo DB SDK annotations for Entities

The Dynamo DB SDK for Java has two modes of operation. There is a "low-level" API where you can interact with the database through the core building blocks, e.g. a "document". Additionally, there is a higher-level API which is somewhat similar to an ORM like JPA. In this example, we go with the second approach.

We need, just like in JPA, to tell the mapper some information about our entity and the mapping for specific attributes. Mainly you need to define the PK of your table. In Dynamo terms, this is called the "Hash-key". You just simply annotate our ID attribute with the annotation with @DynamoDBHashKey. Here's a full example of the [customer entity](https://github.com/mariodavid/cuba-example-dynamodb-access/blob/master/modules/global/src/com/rtcab/cedda/entity/Customer.java):

{% highlight java %}
@DynamoDBTable(tableName = "cuba-customers")
public class Customer extends BaseUuidEntity {

    @DynamoDBHashKey
    @DynamoDBAutoGeneratedKey
    @Override
    public UUID getId() {
        return id;
    }

    // ...

    @DynamoDBAttribute
    public String getName() {
        return name;
    }

    // ...

}
{% endhighlight %}


After doing this, everything is setup correctly, so that it is possible to implement the custom store in the next step.

## Implement a custom DynamoDB datastore

In the steps above, there was this line <code>cuba.storeImpl_customerDynamoDb = cedda_customerDynamoDbDataStore</code> which we used to reference the bean that deals with the custom data store implementation.

Therefore, this is our entry point. We create a Java class like [CustmomerDynamoDbDataStore](https://github.com/mariodavid/cuba-example-dynamodb-access/blob/master/modules/core/src/com/rtcab/cedda/core/CustomerDynamoDbDataStore.java#L19) which implements [DataStore](https://github.com/cuba-platform/cuba/blob/master/modules/core/src/com/haulmont/cuba/core/app/DataStore.java) from CUBA.

Being a DataStore means implementing the following methods:

* load
* loadList
* getCount
* commit
* loadValues

Here's the class. Note the <code>@Component</code> Annotation. This makes this class a Spring bean with the given name, which matches the configuration name.

{% highlight groovy %}
@Component("cedda_customerDynamoDbDataStore")
public class CustomerDynamoDbDataStore implements DataStore {

    @Inject
    CustomerRepository customerRepository;

    @Nullable
    @Override
    public <E extends Entity> E load(LoadContext<E> context) {

        return (E) customerRepository.findOne((UUID) context.getId());
    }

    // ...
}
{% endhighlight %}


Instead of doing the heavy lifting on querying the database ourselves, we will use the "spring-data-dynamodb" project to construct the actual queries.

The only thing that we need to do is to create an interface in the core module, which will be used by the CustomerDynamoDbDataStore: the [CustomerRepository](https://github.com/mariodavid/cuba-example-dynamodb-access/blob/master/modules/core/src/com/rtcab/cedda/core/repository/CustomerRepository.java).

The interface looks like this:

{% highlight groovy %}
@EnableScan
public interface CustomerRepository extends CrudRepository<Customer, UUID> {
}
{% endhighlight %}

That is enough to make the magic of Spring Data happen.


The custom datasource is just an example implementation. In the [github example](https://github.com/mariodavid/cuba-example-dynamodb-access/blob/master/modules/core/src/com/rtcab/cedda/core/CustomerDynamoDbDataStore.java#L19), you will see that I only implemented quite easy requests to the datastore. Mainly queries by ID. This is where the different data storage approaches don't really fit together.

## Limited similarities between SQL & NoSQL

As described in the beginning, there are certain differences in the storage technologies. One of those is, that NoSQL datastores are normally more specific about their use case. In the case of DynamoDB, although it has certain query capabilities, in the end, it is a Key-Value store.

This means, that e.g. a UI component like the generic filter, which heavily relies on the SQL based data store with its relations and its general query capabilities is not a very good fit.

Also, in fact, the DataStore interface of CUBA deals with JPQL as its base query technology is not really transferable.

It can be summarized with this: When you decide to use a Key-Value store with limited query capabilities, you should also reflect this in your UI. So instead of using the generic filter component, create an input field for the Customer ID e.g. instead.

Then, although I decided to choose the DataStore interface as the injection point, you should consider to create a service and with this be able to more clearly define your data storage interface.


### Summary

Besides this conceptual mismatch, it was a fairly easy integration. A lot of the configuration dance that I described above can actually be simplified by using CUBA studio. But this blog post was also to show you that in the end studio does only a little bit of file manipulation for a lot of its magic.

DynamoDB has huge advantages over using a relational database. It also has some downsides that might be a KO criteria for using it in your case. However, it is very valuable to have yet another tool in your toolbox.
