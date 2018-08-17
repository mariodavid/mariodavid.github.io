---
layout: post
title: API integration in CUBA with Xero
description: "In this blog post, we will discover how to integrate with a SaaS software solution like Xero."
modified: 2018-08-03
tags: [cuba, API, integration, xero, REST, OAuth2]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

In this blog post we will discover how to integrate with a SaaS software solution like Xero. For those kinds of integrations there are two main ways to achieve it - API and SDK based. We will compare both approaches and take a look at the concrete integration steps to take.

<!-- more -->

## What is Xero

Xero is an online accounting solution. It also offeres certain features around banking, invoicing as well as inventory management. I will not go into more detail, because in this blog post we want to concentrate on the API integration and not so much around the feature set of this particular SaaS vendor.

## Integration options
Nowadays almost all SaaS products have an Abstract programming interface (API) in order to communicate with them through your own applications automatically. Oftentimes those solutions have an API as well as an SDK which is wrapping and making it nicely accessible to the developers in their language.

So an Integration API is oftentimes a protocol based communication mechanism like HTTP. There you have certain endpoints that you can use in order to achieve your automation purposes.


### API integration with Xero

In the case of Xero, there are planty of API endpoints for the different business cases like:

* https://api.xero.com/api.xro/2.0/banktransactions
* https://api.xero.com/api.xro/2.0/contacts
* https://api.xero.com/api.xro/2.0/employees

Data can be fetched and or changed & stored via sending in HTTP messages. The data that needs to get send in changes for the different use cases. Oftentimes with a HTTP based API you have to use the different HTTP verbs for the different cases.

As a prerequisite it is required to authenticate against the API. This is a little bit different from API to API. Sometimes a username & password combination is sufficient, somtimes a token based approach has to be used.

In the case of Xero, they differentiate between different application that you want to create. When you want to have a private app connecting to Xero, an OAuth2 based authentication with a public / private key combination has to be used.

Besides that they have public & partner application which require a little more sophisticated OAuth workflows.

In this blog post we will use a private app, so the corresponding authentication workflow will be used.


### SDK integration with Xero

As stated above sometimes the SaaS vendor offers an additional SDK for integration. An SDK is normally using the same mechanisms from the API as described above like HTTP communication. What it additionally does is to creating a programming library that can be used in the corresponding language that the integrator is using.

In case of Xero, they have planty SDKs for commong programming languages like Java, Node, C#, PHP and so on.

The main benefit of using an SDK is that the vendor removed some of the heavy lifting for the integration application developer and encapsulated the interaction protocol (for example for the authentication) into the SDK.

As an example lets take a look at the authentication workflow against Xero with a private app.

On the API layer you have to understand how OAuth2 in general works, what particular flavor they use in this case, understand how to sign a request and so on.

On the SDK layer, Xero has mostly removed the neccearity to understand all of this. Instead, they created a Java / Javascript / C# function or object which has a particular interface with particular parameters. The surface area in this case is much smaller, so the integration effort oftentimes becomes easier.

But there are downsides as well. Let's just imagine that you want to log all your HTTP request from your application in a particular form into your cental logging system in order to reason about problems that might occur during production.

In this case, you might want to control what values you want to log. With a direct API integration this is totally possible and up to you, because you write the HTTP interactions anyways. You can decide which HTTP library you want to use and so on.

Since in the SDK case all of the network calls are hidden away from you as a developer, you have less options to change the way the interaction works.

This has to be kept in mind. In this case, we will stick to the SDK based integration.


## Integration implementation with Xero

Now that we have talked about the general approach on how to integrate with a SaaS provider like Xero, let's dig into the details of how to implement the actual integration.

<div>In case you only want to get the code of the integration with Xero: I've made that integration an application component for CUBA, so that you do not have to copy & paste all the source code. You can find it on github: [mariodavid/cuba-component-xero](https://github.com/mariodavid/cuba-component-xero). Also there is an example application which uses the application component: [cuba-example-using-xero](https://github.com/mariodavid/cuba-example-using-xero).</div>

### Adding the Xero SDK dependency

The first thing that we need to do since we want to use their [Java SDK](https://github.com/XeroAPI/Xero-Java) is to add the maven dependency to our project. Most of the sources nowadays are available via a Maven repo, so getting the actual source code is pretty easy:

The <code>build.gradle</code> file first needs to know their maven repository.
According to their docs we have to add the following maven repositories to our application:

{%highlight groovy%}
buildscript {

    repositories {

        maven { url "https://raw.github.com/XeroAPI/Xero-Java/mvn-repo" }
        maven { url "https://raw.github.com/XeroAPI/XeroAPI-Schemas/mvn-repo" }

        // ...
    }
}
{%endhighlight%}

Next is to add the maven dependency and decide where we want to put it in our CUBA application (component). SInce CUBA has this architecture of a multi module gradle project, there are certain modules like "global", "core", "web" and so on, we have to decide in which module we need the source code.

Normally I would say that we only need it in the "core" module, because the interactions with the API happens form our backend encapsulated by a Service in our application. However: as it turns out I needed a particular Interface "com.xero.api.Config" in the "global" module, because I ended up with a [XeroConfiguration](https://github.com/mariodavid/cuba-component-xero/blob/master/modules/global/src/de/diedavids/cuba/xero/configuration/XeroConfiguration.java) CUBA config interface in order to change the settings via the "Application properties" UI.

Therefore I added the dependency to the "global" and the "core" module:



{%highlight groovy%}

configure(globalModule) {
    // ...

    dependencies {
        compile 'com.xero:xero-java-sdk:1.0.9'
    }
    // ...

}

{%endhighlight%}

Note: Since the "global" module is a dependency of the "core" module, the sources from Xero are also available in the "core" module transitevly. This is a little bit implicit, but it works. In case you want to have it a little bit more expressive, you can add a dependency to the "core" module as well.

Once this "infrastructure" work is done, we can use the classes from our own source code. Let's get on with taking a look on how to setup the Xero configuration.

### Mapping the Xero configuration to CUBA world

The SDK needs some kind of configuration parameters in order to take away some of the heavy lifting of the authentication. Also the SDK allows us to configure different HTTP communication options.

[Normally](https://github.com/XeroAPI/Xero-Java#default-configuration) those configuration options are defined in a file called "config.json" that has to be placed at the root directory of your source code directory.

This way of doing it is not super default for Spring applications (as CUBA is), because Spring has certain defaults where to put configuration options. CUBA additionally adds other possibilities to the Table like the Database based Configuration options defined through an Interface definition.

Luckily the SDK provides other options of passing the configuration options to the SDK.

When we take a closer look at the sources of the SDK, we find the main interaction point of the SDK called [XeroClient](https://github.com/XeroAPI/Xero-Java/blob/master/src/main/java/com/xero/api/XeroClient.java) which we finally want to use in order to communicate with the API.

The class has different constructors. One takes no parameters and uses the <code>config.json</code> config file. Another one takes the above mentioned [com.xero.api.Config](https://github.com/XeroAPI/Xero-Java/blob/master/src/main/java/com/xero/api/Config.java) interface.

This means that we are free to create the config interface and pass it in. How we create an instance of this interface - the SDK does not really care if we take this constructor.

The SDK already has an example of doing that via the [SpringConfig](https://github.com/XeroAPI/Xero-Java/blob/master/example/src/main/java/example-spring/SpringConfig.java) example interface, which will natively work with the Spring based approach.

However, as I said, I wanted to give the user the oppurtunity to change the settings on runtime via the "Application properties" UI of CUBA, which means I could not directly uses this example.

Instead I created the [XeroConfiguration](https://github.com/mariodavid/cuba-component-xero/blob/master/modules/global/src/de/diedavids/cuba/xero/configuration/XeroConfiguration.java) Configuration interface in the "global" module which looks like this:




{%highlight java%}


@Source(type = SourceType.DATABASE)
public interface XeroConfiguration extends com.haulmont.cuba.core.config.Config, com.xero.api.Config {


    @Override
    @Property("xero.appType")
    @Default("Public")
    String getAppType();

    @Override
    @Secret
    @Property("xero.privateKeyPassword")
    String getPrivateKeyPassword();

    @Override
    @Property("xero.pathToPrivateKey")
    String getPathToPrivateKey();

    @Property("xero.privateKeyFileDescriptor")
    FileDescriptor getPrivateKeyFileDescriptor();

    @Override
    @Secret
    @Property("xero.consumerKey")
    String getConsumerKey();

    @Override
    @Secret
    @Property("xero.consumerSecret")
    String getConsumerSecret();

    @Override
    @Property("xero.apiUrl")
    @Default("https://api.xero.com/api.xro/2.0/")
    String getApiUrl();

    // ...


{%endhighlight%}

In this interface I combined the two approaches. Through extending <code>com.haulmont.cuba.core.config.Config</code> we make it a regular CUBA config interface, where each getter method will become a configuration property in the UI.
Since the getter methods are also exactly the methods defined in the config interface of Xero this circumstance makes the merging of the two approaches very easy.

The result in the configuration  UI looks like this:

<figure class="center">
	<a href="{{ site.url }}/images/api-integration-in-cuba-with-xero/xero-configuration.png"><img src="{{ site.url }}/images/api-integration-in-cuba-with-xero/xero-configuration.png" style=""></a>
	<figcaption><a href="{{ site.url }}/images/api-integration-in-cuba-with-xero/xero-configuration.png" title="Xero Configuration UI in the CUBA application">Xero Configuration UI in the CUBA application</a></figcaption>
</figure>

In order to retrieve this configuration interface, we can now just inject it into our Spring beans like this:

{%highlight java%}
@Inject
XeroConfiguration xeroConfiguration
{%endhighlight%}

For ease of use I finally created a [XeroClientFactory](https://github.com/mariodavid/cuba-component-xero/blob/master/modules/core/src/de/diedavids/cuba/xero/core/XeroClientFactory.java) as a Spring component in the application component, which has a method <code>public XeroClient createClient()</code>. With this it becomes very easy to create an instance of a xeroClient. Internally it uses the above mentioned constructor with the config reference.

### Storing the personal private key in the CUBA application

The second major reqiurement for the Xero integration to work is to pass the private key file into the SDK so that the authentication can be done successfully.

As said above, it is required to [create a public / private key pair](https://developer.xero.com/documentation/api-guides/create-publicprivate-key) and use it as a way to interact with the API.

Using OpenSSL in the terminal the creation of the files looks like this:


{%highlight bash%}
openssl genrsa -out privatekey.pem 1024
openssl req -new -x509 -key privatekey.pem -out publickey.cer -days 1825
openssl pkcs12 -export -out public_privatekey.pfx -inkey privatekey.pem -in publickey.cer
{%endhighlight%}

The public key will be uploaded on the Xero website when creating the Xero application. The private part of the key (in particular the [PKCS#12](https://en.wikipedia.org/wiki/PKCS_12) or ".pfx" file)
then needs to be used together with a password that has to be defined at key creation time to let the SDK sign the request with the correct information so that Xero can varify that the request comes from the expected caller.

By default the config interface looks for a path to the private key file. Although this works in general, CUBA has an abstraction to get to resources easily with the [Resources](https://github.com/cuba-platform/cuba/blob/master/modules/global/src/com/haulmont/cuba/core/global/Resources.java) interface. Another option would be to use the "external files" feature of CUBA and store the file in the database.

It turns out that this is also achievable because the SDK is extensible in this way too. I will not go into more detail (you can look up the sources of the [CubaConfigBasedSignerFactory](https://github.com/mariodavid/cuba-component-xero/blob/master/modules/core/src/de/diedavids/cuba/xero/core/CubaConfigBasedSignerFactory.java) which enables this behavior), but in the end the application component accepts two ways of adding your private key file:

* in the conf directory of the application
* as an external file (FileDescriptor) through the UI

In the configuration there are the options 

* xero.pathToPrivateKey
* xero.privateKeyFileDescriptor

which activate either of those approches.

With that everything is in place to successfully connect to the Xero API. The last part is to look at how to use the SDK to actually get some data from the Xero API.

## Using the xero application component

In the example application [cuba-example-using-xero](https://github.com/mariodavid/cuba-example-using-xero) we can find a usage of the <code>XeroClient</code> in the [XeroInvoiceService](https://github.com/mariodavid/cuba-example-using-xero/blob/master/modules/core/src/de/diedavids/cuba/ceux/service/XeroInvoiceServiceBean.java)to retrive some invoices from the API and store it in the CUBA application. 

{%highlight java%}

@Service(XeroInvoiceService.NAME)
public class XeroInvoiceServiceBean implements XeroInvoiceService {

    private static final Logger log = LoggerFactory.getLogger(XeroInvoiceServiceBean.class);

    @Inject XeroClientFactory xeroClientFactory;
    @Inject Metadata metadata;
    @Inject DataManager dataManager;

    @Override
    public boolean importInvoices() {

        XeroClient client = xeroClientFactory.createClient();

        try {
            List<com.xero.model.Invoice> xeroInvoiceList = client.getInvoices();

            persistXeroInvoices(xeroInvoiceList);
        } catch (IOException e) {
            log.error("Error while retrieving Invoices from Xero.", e);
        }
        return true;
    }

    private void persistXeroInvoices(List<com.xero.model.Invoice> xeroInvoiceList) {

        for (com.xero.model.Invoice xeroInvoice : xeroInvoiceList) {

            Invoice invoiceEntity = metadata.create(Invoice.class);

            invoiceEntity.setInvoiceNumber(xeroInvoice.getInvoiceNumber());
            invoiceEntity.setInvoiceDate(xeroInvoice.getDate().getTime());
            invoiceEntity.setAmountDue(xeroInvoice.getAmountDue());
            invoiceEntity.setAmountPaid(xeroInvoice.getAmountPaid());
            invoiceEntity.setTotal(xeroInvoice.getTotal());
            invoiceEntity.setSubTotal(xeroInvoice.getSubTotal());
            invoiceEntity.setTotalTax(xeroInvoice.getTotalTax());

            dataManager.commit(invoiceEntity);
        }
    }
}

{%endhighlight%}

As we see here we use the XeroClient instance to call <code>getInvoices</code>. This method will return us all invoices from the API as a list of <code>Invoice</code> instances.

The package <code>com.xero.model</code> is auto generated and contains all the business entities as Java classes. Those classes are generated on the XML schema definitions defined in the [XeroAPI-Schemas](https://github.com/XeroAPI/XeroAPI-Schemas/tree/master/src/main/resources/XeroSchemas/v2.00) project.

In this example there is a Invoice class as a persistent entity in the CUBA application. After receiving every invoice from Xero, it will be converted into a CUBA invoice and then stored in the database.


<figure class="center">
	<a href="{{ site.url }}/images/api-integration-in-cuba-with-xero/xero-invoices-download.png"><img src="{{ site.url }}/images/api-integration-in-cuba-with-xero/xero-invoices-download.png" style=""></a>
	<figcaption><a href="{{ site.url }}/images/api-integration-in-cuba-with-xero/xero-invoices-download.png" title="Xero Configuration UI in the CUBA application">Xero Configuration UI in the CUBA application</a></figcaption>
</figure>

## Summary

As you see, with the SDK in place, the interaction for the application developer becomes trivial. No need to deal with OAuth tokens, private keys, HTTP response codes, data binding and so on.

Although is has downsides of using an SDK, it is oftentimes the most straight forward way of doing an integration.

Luckily with open protocols like HTTP, OAuth and so on using an SDK is just a convinient way of doing it. If this approach is not suitable in a particular case, you always can go one layer below and use the protocols directly.
