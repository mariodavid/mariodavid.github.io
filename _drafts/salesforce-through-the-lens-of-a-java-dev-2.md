---
layout: sf-lens-red
title: Salesforce trough the Lens of a Java Dev
subtitle: Part 2 - Salesforce Technology Comparison
description: "This blog post series is about the my learnings that I had during working in the domain of Salesforce Development. I will try to put SF into a broader context and compare it to previous experice - in particular CUBA"
modified: 2019-11-18
tags: [cuba, Salesforce]
image:
  feature: salesforce-through-the-lens-of-a-java-dev-part-2/feature.png
---
This blog post series is about the my learnings that I had during working in the domain of Salesforce Development. I will try to put Salesforce into a broader context and compare it to previous experice - in particular CUBAs.

<!-- more -->

After elaborating the mindset and landscape in the first part let's go a little bit deeper into technical details in this part. 

## Declarative vs. Imperative Development

The first topic to talk about is the concept of declarative development. Declarative development mainly means to configure parts of the solution by expressing the intend of what we would like to achieve instead of also express how we would like to get the solution to go there. 

In the land of Salesforce this happens through a more or less user friendly & accessible user interface that sits directly next to the actual application that is build.

This concept is quite big in the Salesforce ecosystem and it is for good reasons. 

On the one hand there is the target audience of tech-savy business people and Administrators, which need and benefit most of this kind of development. 

On the other hand, Salesforce runs the solutions the users design on their behalf as part of the Software-as-a-Service offering. This means that they have to take care that the stuff that some builder / developer creates is controllable when running their solution. Because Salesforce has to take care that an error, that I as a builder / developer within the solution, does not effect any other application that runs on Salesforce.

Declarative development is much more controllable & restricable from the SaaS runtime perspective of Salesforce than imperative development mechanisms are.

Besides those two concrete Salesforce related reasons for declarative development there is an even more fundamental reason: it is oftentimes simply not necessary to use imperative development when the higher-level abstractions of declarative development are sufficient.

In order to make this abstarct explanation about declarative vs. imperativ development a little bit more concrete, let's take a look at a simple example.


### An Example of Configuration: Adding a Field



## Salesforce Technology Landscape

Next let's explore the technology landscape of the Salesforce platform in order to get a better feeling of the overall space. As this topic is so huge to fill multiple books we will just scratch the surface level. But conceptually I will try to at least touch the main options available to give you a full overview of the landscape.

Broadly speaking, it is possible to put the available solutions & technologies into the following categories:

1. User Interface
2. Workflow & Automation
3. Business Logic & Database
4. Integrations & APIs

Let's go through those categories to unfold these somewhat general terms.

### Example Project: Salesforce Petclinic

All the examples that you will see in this blog post series will based on an example project: Petclinic. This example originated from the Spring Petclinic and was ported extended later implemented with CUBA.

I took this example domain in order to show certain concepts in Salesforce, but did not implemented it thoughtly.

More information on the Petclinic example can be found here: [CUBA Petclinic](https://demo10.cuba-platform.com/petclinic/).

### 1. User Interface

Starting off with the first category of user interfaces. In Salesforce this can happen in various shapes and forms or better: at various abstraction levels.


#### Page Layouts
At the very top of the abstraction pyramid, let's look first at a detail screen of a Pet:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/pet-details-screen.png" />

This Screen is defined out of the box for every costum object, that is created within Salesforce. It is possible to configure the arrangement of elements within the screen, as well as fields that should be displayed.

This definition of the screen is expressed within a dedicated Setup section of the SF environment. 

The screen aragement is called "Page Layout". It can be defined within the "Lighning App Builder":

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/pet-details-screen-page-layout.png" />

The way to configure requires a little bit knowledge about the SF concepts, but besides that it does not require any kind of coding capabilities.

#### Flow
The next layer below Page Layouts is something called "Flow". Flow is a capability to define UI workflows / wizards with a brief understanding of programming concepts. 

It still does not require to write source code, but it at least is necessary to have some abstract understanding of the underlying concepts like conditions or variables.  

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/flow.png" />

Instead of defining particular screens / pages, flows main focus is around guided user interactions that are across multiple screens.

#### Lightning Web Components
Going down further in the abstraction pyramid more into the imperative development with source code side of things there are also multiple options available.

It allows to cover all cases that are not available via the higher level abstractions. 

Salesforce offered various technological choices that emerged over the years. The newest incarnation is something called "Lightning Web Components". It is a propriatery Javascript based component library available to the developers that are working with Salesforce.

Those components are used by Salesforce itself in a lot of parts to create the out-of-the-box user interfaces. The goal is that there is a common look and feel independent if the component was created by Salesforce or by an external developer.

{%highlight html%}
<lightning-datatable
        key-field="id"
        data={data}
        columns={columns}
        onrowaction={handleRowAction}>
</lightning-datatable>
{% endhighlight %}

This code (together with some Javascript functionality to load data) will result in the following UI component:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/lightning-web-component-datatable-petclinic.png" />

### 2. Workflow & Automation

Let's turn away a little bit from the user interface part to a related area: Workflow & Automation. 

In order to create a solution that is more than just a structured way of storing data, it is necessary to create some kind of automation to take away the heavy lifting from the user.

In this area there are also multiple solutions on different levels of the abstraction pyramid.

#### Process Builder

One very powerful declarative development mechanism in Salesforce is called "Process Builder". What it does is, that it allows to define automation workflows based on certain environmental event that occur. 

One abitrary example: When a Pet is created in the Petclinic example and the Pet is of type "Electric" a new Task is created for a Electric Vet specialist, to review the new Pet. This Task appears in the Vets Task Management with a reference to the Pet.

Automatically after one week a reminder Email is send. Also a Visit for a regular checkup is scheduled after another week.

Defining this processes is part of the declarative development spectrum as well. Here is a screenshot of the Process Builder configuration user interface:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/process-builder.png" />

It does not require to write source code. That being said it is possible to interact with source code. Oftentimes it will send notifications to the system that then executes a particular piece of source code. 

This intersection is quite common and has the potential to be a real enabler. It allows to let the pieces of the functionality be where they belong the best. Sometimes it is declarative development, sometimes it is imperative development via source code.

Normally such functionality would be embodied in a BMPN workflow engine. Process builder is the propriatery equivalent of that tightly integrated onto other Salesforce capabilities.

### 3. Logic & Database

The next section is all about business logic and the ability to store data. There is also a broad declarative part to it as well as an imperative part.

#### Object Manager

The most obvious declarative mechanism to define data storage is the Object Manager. It allows to define objects (= Database Tables / JPA Entities) and fields of that object. Fields can be either regular datatypes like text, numbers, checkboxes, picklist but also relationships to other objects.

It is also possible instead of creating custom Objects to adjust existing Objects (to a certain degree). As said before Salesforce comes with certain business objects out-of-the-box like <code>Account, Product, Case</code> etc.

One example of a custom object is the above shown <code>Pet</code> object. To give you a sneak-peak of corresponding Configuration user interface as part of the Object Manager, here is a screen shot of the definition of the fields that are configured for the Pet object:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/object-manager.png" />

#### Calculated Fields

A good example of logic is the concept of calculated / formular fields. When defining an Object in Salesforce it is possible to define fields of different types (numbers, text, checkboxes etc.). Those fields can either be editable by the user, or they can be calculated. Those fields can execute simple arithmetic operations like sum building, but they can also be defined as complex formulars.

Those Formulars are conceptually very similar to a function in an Excel cell like <code>=SUM(MIN(A1:A3), MAX(B5:B10))</code>.  Similar to Excel the Salesforce Configuration User Interface supports with explanations and a Click-and-Point tool.

The following configuration UI contains a definition of a calculated field for the Pet object. It calculates a reminder date that can be used for sending out Email reminders to the Vets about an upcoming Pets Birthday:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/formular.png" />

Defining those formulars requires to have an understanding of this dedicated formular language from the user. But conceptually it only requires the same level of understanding as for defining an Excel calculated cell. As seen in the Screenshot Salesforce tries to help the builder / developer here via inline help e.g.


### 4. Integrations & APIs

In the remaining last big category it is all about manual and automatic integrations between the Salesforce application and other applications. 

In the space of integration it starts from something very lightweight like manual / automatic file import. Next there are various out-of-the-box integrations with external Services like Outlook 365, CTI offerings like Amazon Connect e.g. All those integrations are all still on the declarative side of things. 

Here is one simple example of an integration. In this screen an CSV import was started to import new Visits from another system manually:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/csv-import.png" />

Then in the intersection between declarative development & imperative development there are integrations with arbitrary Webservices via SOAP / REST.

Besides the Integration side, where the Salesforce application sits more on the consuming side of things it is also possible to let Salesforce be the Provider of data. Manually this happens via various export & reporting functionalities that are part of the standard Salesforce offering. 

When it comes to automated integration it goes into the direction of APIs. Salesforce provides out-of-the-box APIs via SOAP Webservices and further allows to imperatively develop any kind of APIs to act as the Provider of automated system-to-system integrations.

Through dedicated integration services like Zapier it is possible to extend that list to thousands of Services.

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/zapier-salesforce-to-google-calender.png" />

In this example an integration is configured through Zapier to create a new Google Calender Event in case a new Record (e.g. Pet) is created in the Salesforce System.

### It is all about declarative development?

After looking over the technology landscape in Salesforce you could get the impression that this is all only Click-and-Point. 

This in fact is not true, although Salesforce coveres the main use-cases all with declarative development. Instead for almost all use-cases it is possible to alternatively imperatively develop the solutions through source code.

> The Choice between declarative vs. imperativ development boils down to the abstraction level that is necessary to fulfill the business requirements.

Declarative development in these configuration UI has the following characteristics:

* it increases the abstraction level
* it (highly) limits the flexibility of the solution
* it speeds up initial development due to higher level abstractions
* it openes up the specifics to a broader range of builders/developers

## Salesforce for Developers

I already showed you a small example that source-code based software development is also available in Salesforce. Indeed the area of what can be done in this area is at least as big as or even bigger compared to the declarative development part.

As Salesforce already has a history of technologies that were supported throught the years, there are various options. We will only focus on the current technology stack.

### Salesforce Javascript Fragmentation Approach

When it comes to the user interface, the technology is Javascript, as Salesforce exclusively is a web based solution.

As for all big tech players that market in particular is very problematic technology wise, as it is very fragmented and fast moving (although it slowed down in the last 1-2 years). This highly contradicts with the requirements of enterprises that normally need loooong living technology lifecycles and support.

Salesforce does not have a silver bullet here. In order to not make a big bet on any Javascript framework that will be irrelevant and gone in two years, they develop their own framework, where probably the main reason is that they are control the lifecycle better, which is necessary to fulfill the enterprise requirement of long support.

SAP went into a similar direction with [SAP UI 5](https://openui5.org/).

To balance those two contradictory forces they have multiple technologies in different phases of their lifecycle in their portfolio. Currently those are in particular these two options:

* Lightning Web Components (newer, build ontop of open Webcomponents standard, does not cover all use-cases, open source)
* Aura Components (more mature, older, propriatery)

This blog post from Salesforce shares some more light on this topic: https://developer.salesforce.com/blogs/2019/05/introducing-lightning-web-components-open-source.html

This move of adopting open standard and also embracing open source is aligned with what the majority of the software industrie is doing nowadays. It also shows that Salesforce wants to leverare the community effects in the software development world much more then it used to do before (we will see this other examples later).

