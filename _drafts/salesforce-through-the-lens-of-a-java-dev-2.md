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

The first topic to talk about is the concept of declarative development. Declarative development in the land of Salesforce mainly means to configure parts of the solution by expressing the intend of what we would like to achieve instead of also express how we would like to get the solution to go there. This happens through a more or less user friendly & accessible user interface.

This concept is quite big in the Salesforce ecosystem and it is for good reasons. On the one side there is the target audience of tech-savy business people and Administrators, which benefit most of this kind of development. 

On the other hand, Salesforce runs the solutions the users design on their behalf as part of the Software-as-a-Service offering. This means that they have to take care that the stuff that some user creates is controllable when running their solution. Because Salesforce has to take care that an error that I as a builder / developer of the solution does not effect any other partner that runs on Salesforce.

Declarative development is much more controllable & restricable from the SaaS runtime perspective of Salesforce than imperative development mechanisms are.

But first and foremost I would say there is another reason and that is: it is oftentimes simply not necessary to use imperative development when the higher-level abstractions of declarative development is sufficient.

In order to make this abstarct explanation about declarative vs. imperativ development a little bit more concrete, let's take a look at a simple example.


### An Example of Configuration: Adding a Field



## Salesforce Technology Landscape

Next let's explore a little bit the technology landscape of the Salesforce platform in order to get a better feeling of the overall space.

Broadly speaking, it is possible to put the available solutions & technologies into the following categories:

1. User Interface
2. Workflow & Automation
3. Business Logic & Database
4. Integration

Let's go through the categories to unfold these somewhat general terms.

### User Interface

Starting off with the "User Interface". It is obviously about creating user interfaces. But in Salesforce this can happen in various shapes and forms or better: at various abstraction levels.


#### Page Layouts
Starting from the very top of the abstraction pyramid, let's look first at a detail screen of a Pet:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/pet-details-screen.png" />

This Screen is defined out of the box for every costum object, that is created within Salesforce. It is possible to configure the arrangement of elements within the screen, as well as attributes that should be displayed within a dedicated Setup section of the SF environment. 

This Page Layout can be defined within something called "Lighning App Builder":

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/pet-details-screen-page-layout.png" />

The way to configure requires a little bit knowledge about the SF concepts, but besides that it does not require any kind of coding capabilities.

#### Flow
The next layer below Page Layouts is something called "Flow". Flow is a capability to define UI workflows / wizards with a brief understanding of programming concepts. 

It still does not require to write source code, but it at least is necessary to have some abstract understanding of the underlying concepts like conditions or variables.  


<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/flow.png" />

Instead of defining particular screens / pages, flows main focus is around guided user interactions that are across multiple screens.

#### Lightning Web Components
Programming in the User interface layer is also possible. It allows to cover all cases that are not available via the higher level abstractions. In the concrete with Lightning Web Components Salesforce exposes a propriatry Javascript based component library to the Developers that are working with Salesforce.

Those components are used by Salesforce itself to create the existing User interfaces, so that there is generally a common look and feel independent if the component is created by Salesforce or by an external developer.

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

### Workflow & Automation

Turning away a little bit from the User interface part to a related area: Workflow & Automation. In order to create a solution that is more than just a structured way of storing data, it is necessary to create some kind of automation to take away the heavy lifting from the user.

In this area there are also multiple solutions on different levels of the abstraction pyramid.

#### Process Builder

One very powerful declarative development mechanism in Salesforce is called Process Builder. What it does is, that it allows to define automation workflows based on certain environmental circumstances. 

One abitrary example: When a Pet is created in the Petclinic example, in case the Pet is of type "Electric" a new Task is created for a Electric Vet specialist, to review the new Pet. Automatically after one week a reminder Email is send. Also a Visit for a regular checkup is scheduled after another week.

Defining this processes is part of declarative development as well. Here is a screenshot of the Process Builder Configuration User Interface:


<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/process-builder.png" />

It does not require to write source code. That being said it is possible to interact with source code. Oftentimes it will send notifications to the system that then executes a particular piece of source code. 

This intersection is quite common and has the potential to be a real enabler. It allows to let the pieces of the functionality be where they belong the best. Sometimes it is declarative development, sometimes it is source code.

Normally such functionality would be embodied by a BMPN workflow engine. Process builder is the propriatery equivalent of that tightly integrated onto other Salesforce capabilities.

### Logic & Database

The next section is all about business logic and the ability to store data. There is also a broad declarative as well as an imperative part to it.

#### Object Manager

The most obvious declarative mechanism to define data storage is the Object Manager. It allows to define Objects (= Database Tables / JPA Entities) and fields of that object. Fields can be either regular datatypes like text, numbers, checkboxes, picklist but also relationships to other objects.

It is also possible instead of creating custom Objects adjust existing Objects (to a certain degree). As said before Salesforce comes with certain business objects out-of-the-box like <code>Account, Product, Case</code> etc.

One example is the above shown <code>Pet</code> object. The corresponding Configuration User Interface looks like this:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/object-manager.png" />

#### Calculated Fields

A good example of logic is the concept of calculated / formular fields. When defining an Object in Salesforce it is possible to define fields of different types (numbers, text, checkboxes etc.). Those fields can either be editable by the user, or they can be calculated. Those fields can execute simple arithmetic operations like sum building, but they can also be defined as formulars.

Those Formulars are conceptually very similar to a function in an Excel cell like <code>=SUM(MIN(A1:A3), MAX(B5:B10))</code>. They have a dedicated language, that the user needs to get familar with, but conceptually requires the same level of understanding as for defining an Excel calculated cell. Similar to Excel the Salesforce Configuration User Interface supports with explanations and Click-and-Point tools.

The following configuration UI contains a definition of a calculated field for the Pet object. It calculates a reminder date that can be used for sending out Email reminders to the Vets about an upcoming Pets Birthday:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/formular.png" />

### Declarative Programming Possibilities in SF

### Salesforce for Developers



## 
