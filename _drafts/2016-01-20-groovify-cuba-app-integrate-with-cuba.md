---
layout: post
title: Groovify CUBA - Integrate Groovy with CUBA
description: "In this blog post i’d like to show you how to change the normal Java CUBA App to a Groovy CUBA App to increase the developer productivity even further"
modified: 2015-1-20
tags: [cuba, groovy, java]
image:
  feature: groovify-cuba-app/feature-2.jpg
  feature_source: https://pixabay.com/de/neujahr-silvester-silvester-2015-1090770/
---

In this second blog post about groovify your [CUBA](https://www.cuba-platform.com/) app, after introducing Groovy it's all about integrating Groovy with CUBA in different shapes.


<!-- more -->

## Give it to my CUBA project

To activate Groovy in the project is pretty [straight forward](https://www.cuba-platform.com/support/topic/using-groovy-to-implement-a-service-interface) when you know a little about Gradle. 

In the <code>build.gradle</code> you have to add the groovy plugin in the section of configure <code>coreModule</code>. Then you have to tell groovy where to find the <code>src</code> and the <code>test</code> directories.
{% highlight groovy %}
configure([globalModule, coreModule, guiModule, webModule]) {
    
    // ...

    apply(plugin: 'groovy')
    sourceSets {
        main {
            groovy {
                srcDirs = ['src']
            }
        }
        test {
            groovy {
                srcDirs = ['test']
            }
        }
    }
{% endhighlight %}

With this setting inplace, you are able to create a Groovy file inside of your IDE in the core module.

### Let CUBA Studio know about Groovy

When using CUBA Studio the problem is, that the files that get generated are Java classes. To change that, we have the possibility to [change the templates](https://www.cuba-platform.com/support/topic/customization-of-code-generation-produced-by-studio) that are used for code generation. In the installation directory of CUBA studio there is a directory called <code>templates</code>, where you find the different template files for entities, enums, listeners, screens and so on like this:

{% highlight bash %}

mario@mycomputer:~/cuba/studio-2.0.4$ tree templates
templates
├── ed
│   └── newEntity.java
├── enumd
│   └── newEnum.java
├── fts
│   └── fts.xml
├── listener
│   └── newEntityListener.java
├── sd
│   ├── collectionAttributeTable.xml
│   ├── fieldGroup.xml
│   ├── mainWindow.java
│   ├── mainWindow.xml
│   ├── newScreen.java
│   ├── newScreen.xml
│   ├── standardScreens
│   │   ├── ${entity.className}Browse.java
│   │   ├── ${entity.className}Edit.java
│   │   ├── ${lowercaseEntityClass}-browse.xml
│   │   └── ${lowercaseEntityClass}-edit.xml
│   └── standardScreensSingleMode
│       ├── ${entity.className}Browse.java
│       └── ${lowercaseEntityClass}-browse.xml
└── serviced
    ├── newServiceBean.java
    └── newServiceInterface.java


{% endhighlight %}

You can just walk through these files and change the file extension to <code>groovy</code>, because Groovy syntax is a super set of the java syntax which means, that quite every java code is valid groovy code. Actually there is one exceptions since Java 8 introduces Lambdas, which will probably be fixed some time in the future. Nevertheless, after renaming the files, as an example we take a look at a the file <code>${entity.className}Browse.java</code> and see what can be changed towards Groovy:

{% highlight groovy %}

<%
if (copyright) {
        println '/*'
        println " * ${copyright.replace('\n', '\n * ')}"
        println ' */'
} else {
        print ""
}
%>package ${currentPackage};

import com.haulmont.cuba.gui.components.AbstractLookup;

${classComment}
public class ${entity.className}Browse extends AbstractLookup {
}

{% endhighlight %}

Now it get's a little bit meta, because the the content of the file is [Groovy Template Engine Code](http://docs.groovy-lang.org/latest/html/documentation/template-engines.html#_simpletemplateengine), because studio has to replace some of the content in that file with something dynamic like the class name. This dynamism is expressed in this <code>${}</code> expressions. We don't care about that little strage situation and go straight in to groovify the class:

{% highlight groovy %}

<%
if (copyright) {
        println '/*'
        println " * ${copyright.replace('\n', '\n * ')}"
        println ' */'
} else {
        print ""
}
println '/*'
println " * This is a a generated Groovy class"
println ' */'   
%>package ${currentPackage}

import com.haulmont.cuba.gui.components.AbstractLookup

${classComment}
class ${entity.className}Browse extends AbstractLookup {
}

{% endhighlight %}

In this case it's not that big of a change, because we can only remove the semicolons and the public keyword. Nevertheless with this changes (and you can go through the list of files and adjust them based on your needs) studio will generate the correct filetypes for us so we can directly jump into them and start coding in Groovy.

Even if you are not plan to use Groovy, you can use the template stuff either way to adjust (with Groovy code :-)) the generated content.


## Using Groovy for Testing

After getting the Groovy foot into the CUBA door, there a different things that can be used. One thing that i would like to talk about is Unit testing. There is a framework that got a lot of attraction during the last few years which is called *Spock*.

<a href="https://flic.kr/p/eBpwHY"><img style="float:right; padding: 5px; padding-right:0px;" src="{{site.url}}/images/groovify-cuba-app/spock.jpg"></a>

[Spock](https://github.com/spockframework/spock) is a unit testing framework like JUnit, just with superpower. It combines a normal unit testing framework like JUnit with a Mocking framework like [Mockito](http://mockito.org/) and puts very much of syntactic sugar on top of it like [RSpec](http://rspec.info/) does.

Here is a quick example of a spock specification:

{% highlight groovy %}


class Customer {
    
    String firstName
    String lastName
    List<Order> orders = []

    boolean isGoodCustomer() {
        orders.size() > 0
    }

}

class Order {
    int amount
}

def 'A customer is good if there a at least one related order'() {

    given: 'a customer'
    def mario = new Customer(firstName: "Mario", lastName: "David")

    and: 'the custoemr has one order'
    mario.orders << new Order(amount: 100)

    expect: 'the customer is good'
    mario.isGoodCustomer()

}   
{% endhighlight %}

We see different things in here. First of all, instead of having to write this horrible CamelCase JUnit test names, the method name can be a string which lets you express the intend of the test much better.

Next, with the [GivenWhenThen](http://martinfowler.com/bliki/GivenWhenThen.html) syntax spock leans towards a specification by example syntax which improves the reability of the test further. In this case i used <code>given</code> and <code>expect</code>, but <code>given</code> <code>when</code> <code>then</code> is also a valid syntax. Additionally you can use the strings next to the keywords for documentation purposes.

A barely notable feature is the use of Spock's power assertions in the <code>then</code> block. Every expression will be evaluated against an <code>assert</code>. [Here](http://mrhaki.blogspot.de/2010/07/spock-spotlight-assert-magic.html) is a very good explanation of this feature.

The next feature of Spock is Mocking. Here is an example of a unit test, where certain parts of the implementation are mocked (this is obvioisly a ridiculous example, but it shows the point of mocking pretty good):


{% highlight groovy %}

class Customer {

    String firstName
    String lastName
    List<Order> orders = []

    @Inject
    OrderPlacementService orderPlacementService

    void placeOrder(Order order) {
        orders << order
        orderPlacementService.placeOrder(order)
    }

}

class Order {
    int amount
}

// ...

def 'When an order is placed, the the order placement sytem will get notified'() {

    given: 'a mocked order placement system'
    def orderPlacementServiceMock = Mock(OrderPlacementService)

    and: 'a customer'
    def mario = new Customer(
        firstName: "Mario", 
        lastName: "David", 
        orderPlacementService: orderPlacementServiceMock
    )

    and: 'an order'
    def order = new Order(amount: 100)

    when: 'the order is placed'
    mario.placeOrder(order)

    then: 'the order placement system will get notified'
    1 * orderPlacementServiceMock.placeOrder(order)
}   
{% endhighlight %}

In this case, the <code>OrderPlacementService</code> is a part of the implementation that has to be mocked, because we are not going to plan a integration test here. We want to assure, that the communication with the OrderPlacementService works as expected. 

In the <code>given</code> block the objects get wired together. The <code>then</code> block describes the expected behavior: <code>OrderPlacementService</code> has a method <code>placeOrder</code> which has to be called exactly once (<code>1 *</code>) with the correct parameter (in the case the order). [Here](http://meetspock.appspot.com/script/5095752674050048) is the running example.

Spock has pretty advanced features regards Mocking, so i encourage you to check out the docs on [Interaction based testing](http://spockframework.github.io/spock/docs/1.0/interaction_based_testing.html) if you're interested.

If you want to play around with Spock, [here is a REPL](http://meetspock.appspot.com/) that lets you play with it nicely.