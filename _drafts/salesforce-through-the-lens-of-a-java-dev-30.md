---
layout: sf-lens-red
title: Salesforce trough the Lens of a Java Dev
subtitle: Part 3 - Imperative Technology Solutions
description: "This blog post series is about the my learnings that I had during working in the domain of Salesforce Development. This blog post gives an overview of one side of the Technology coin: the declarative development solutions."
modified: 2020-02-28s
tags: [cuba, Salesforce]
image:
  feature: salesforce-through-the-lens-of-a-java-dev-part-2/feature.png
---
This blog post series is about the my learnings that I had during working in the domain of Salesforce Development. This blog post gives an overview of the other side of the technology coin: the imperative development solutions.

<!-- more -->

## Salesforce Imperative Technologies

I already showed you a small example that source-code based software development is also available in Salesforce in the last post of the series. Indeed the area of what can be done in this area is at least as big as or even bigger compared to the declarative development part.

As Salesforce already has a history of technologies that were supported throught the years, there are various options. We will only focus on the current technology stack.

### Salesforce Javascript Fragmentation Fighting Approach

When it comes to the user interface, the technology is Javascript, as Salesforce exclusively is a web based solution.

As for all big tech players that market in particular is very problematic technology wise, as it is very fragmented and fast moving (although it slowed down in the last 1-2 years). This highly contradicts with the requirements of enterprises that normally need loooong living technology lifecycles and support.

Salesforce does not have a silver bullet here. In order to not make a big bet on any Javascript framework that will be irrelevant and gone in two years, they develop their own frameworks. Probably the main reason for that is that they are control the lifecycle better, which is necessary to fulfill the enterprise requirement of long support.

SAP went into a similar direction with [SAP UI 5](https://openui5.org/) a couple of years ago.

To balance those two contradictory forces they have multiple technologies in different phases of their lifecycle in their portfolio. Currently those are in particular these two options:

* Lightning Web Components (newer, build ontop of open Webcomponents standard, does not cover all use-cases, open source)
* Aura Components (more mature, older, propriatery)

This blog post from Salesforce shares some more light on this topic: [Salesforce Developer - Introducing Lightning Web Components Open Source](https://developer.salesforce.com/blogs/2019/05/introducing-lightning-web-components-open-source.html).

This move of adopting open standards and also embracing open source is aligned with what the majority of the software industrie is doing nowadays. It also shows that Salesforce wants to leverare the community effects in the software development world much more then it used to do before (we will see this other examples later).

### Backend Code: APEX

All of the non UI related functionality, where source code is required is built in a programming language called "APEX". This programming language allows developers to create functionality and business logic that is executed within the Salesforce multitenant platform. 

Although it is to be classified as a general purpose language, it has a specific focus on the needs of business applications. Syntactically it is very similar to earlier versions of Java / C#. 

But as the programs are run on top of the Salesforce Multitenant application runtime, Salesforce puts heavy governor limits on what can be executed in which way. E.g. there are limits on how many DB interactions per Transactions are allowed or how many big the maximal number of rows for a given batch run is. 

In this respect, programming in that environment feels a little bit like developing embedded systems and not enterprise applications, because of all the runtime constraints that require certain different programming patterns.

## Declarative & Imperative Symbiose

I already mentioned it earlier, but as this topic is quite important, it is worth going a little bit more into detail on the correlation between the declarative and imperative parts of a Salesforce application.

The declarative part and the imperative part of a solution are treated as equal from a runtime perspective of Salesforce. Once a validation rule is developed either via a declarative formular or as part of a Trigger implementation as a APEX class, Salesforce does not make any kind of difference between them.

But is not only about how the configuration is treated. It is also about how those two can integrate with each other. Let's go through two examples where it is possible either go from declarative to imperative development or the other way around.

### Platform Events from Process Builder

As part of the process builder, it is possible to define the communication between the process and the invocation of a particular logic that was implemented through source code.

The example here is, that once a new Electric Pet is created, a new Task for a vet that is specialized in Electric Pets is created to prepare the first visit.

Let's further assume that the creation of a Task is not possible declaratively. Therefore some specific logic was defined in source code, which allows to do this.

The communication between those two worlds is now established through a platform event like this:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-3/process-builder-platform-event.png" />

Now, on the source code side of things: this platform event can be consumed by imperative source code.

{% highlight java %}
trigger ElectricPetCreatedTrigger on Electric_Pet_Created__e (after insert) {    
    List<Task> tasks = new List<Task>();
    for (Electric_Pet_Created__e event : Trigger.New) {
            
            Id vet = findVetForElectricPet(event);

            tasks.add(
              new Task(
                IsHighPriority = true,
                Status = 'Open',
                Subject = 'Prepare for new Electric Pet: ' + event.Pet__r.Name,
                WhatId = event.Pet__c,
                WhoId = vet
                // ...
              )
            );
        }
    }

    if (tasks.size() > 0) {
        insert tasks;
    }
}
{% endhighlight %}

### Custom UI Components in Page Builder

Another example of this symbiose is the usage of a custom imperatively created UI component within the Page Layout definition.

Let's assume we have developed a particular complex UI component, which should build a tree of the pet types and highlight the pet type of a Pet in a particular way. The component source code would look like this:

{% highlight html %}
<aura:component description="Pet Classification Tree"
                implements="force:appHostable,flexipage:availableForRecordHome,force:hasRecordId">

    <lightning:tree items="{! v.items }" header="Pet Classification"/>

</aura:component>
{% endhighlight %}

In order to leverage this Component now in a Screen, by implementing the above mentioned interfaces (like <code>flexipage:availableForRecordHome</code>) the Component is made available in the Page Layout Designer:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-3/custom-lightning-component.png" />

With that now once again, the declarative part can leverage the custom sophisticated source-code driven functionality and orchestrate that within declarative page layouts.

## Test Automation

Test Automation is one of the topics that has not been part of any discussion up until that point. But since it is a crucial part of every software development endevor let's take a look at this topic as well.

Every professional software development efforts go hand-in-hand with automated testing. Reason is simple: software gets complex in functionality very quickly. Verifying that the software is still behaving as it should be at every change in a manual fashion very fast becomes an effort that is cheaper to automate away.

Now in Salesforce land, this software is highly configuable with all the declarative development that we have seen. Also the target audience for these kinds of declarative development is mainly tech-savy business people / administrators.

What has that to do with test automation? Well it turns out that those two situations are somehow contradictory or at least interfere with each other.

If you remember all these lovely declarative functionalities that I showed you in the last blog post. How "easy" it is to "just" make a change and reduce the time-to-market by deploying this configuration change. And that a lot of more people have access to it, because it has become so easy to do through a configuration UI.

So from a quality assurance point of view it does not really matter how the change was made. The same mechanisms and quality gates apply. Quality assurance for the most part, in this declarative world seems to be tackled by governance / processes.

### Unit Testing

### Functional Testing




## Development Lifecycle

### Metadata & Metadata API

### Salesforce Orgs

### Deployment




## Salesforce Approach to 2020 Developer Ecosystem

"Making Developers a first class target audience"

### Adopting Developer Mainstream Approaches
- Serverless Evergreen
- Lightning Web Components

### Salesforce CI / CD

- SFDX
- CI
- Version Control