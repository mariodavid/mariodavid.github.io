---
layout: sf-lens-red
title: Salesforce through the Lens of a Java Dev
subtitle: Part 2 - Declarative Technology Solutions
description: "This blog post series is about the learnings that I had during working in the domain of Salesforce Development. This blog post gives an overview of one side of the Technology coin: the declarative development solutions."
modified: 2020-02-25
tags: [cuba, Salesforce]
image:
  dir: salesforce-through-the-lens-of-a-java-dev-part-2
  feature: salesforce-through-the-lens-of-a-java-dev-part-2/feature.png
---
This blog post series is about the learnings that I had during working in the domain of Salesforce Development. This blog post gives an overview of one side of the technology coin: the declarative development solutions.

<!-- more -->

After elaborating on the mindset and landscape in the first part let's go a little bit deeper into technical details in this part.

## Declarative vs. Imperative Development

The first topic to talk about is the concept of declarative development. Declarative development mainly means to define parts of the solution by expressing the intention of what we would like to achieve instead of also express how we would like to get the solution to go there. 

In the land of Salesforce, this happens through a more or less user-friendly & accessible configuration user interface that sits directly next to the actual application that is built.

This concept is quite big in the Salesforce ecosystem and it is for good reasons. 

On the one hand, there is the target audience of tech-savvy business people and Administrators, which need and benefit most of this kind of development. 

On the other hand, Salesforce runs the solutions, the users' design on their behalf as part of the Software-as-a-Service offering. This means that they have to take care that the stuff that some builder/developer creates is not going wild when running their solution on their servers. 

One example of this would be that I, as a builder/developer cannot create something, that affects any other application that runs on Salesforce by another client.

Declarative development tools are much more controllable & restrictable from the SaaS runtime perspective of Salesforce than imperative development mechanisms are.

Besides those two concrete Salesforce related reasons for declarative development, there is an even more fundamental reason: it is oftentimes simply not necessary to use imperative development when the higher-level abstractions of declarative development are sufficient.

To make this abstract explanation about declarative vs. imperative development a little bit more concrete, let's take a look at various UI configuration examples that show how to do certain things are expressed through declarative development.


## Salesforce Declarative Development Technologies

Next, let's explore the declarative technology landscape of the Salesforce platform to get a better feeling of the overall space. As this topic is so huge to fill multiple books we will just scratch the surface level. But conceptually I will try to at least touch the main options available to give you a full overview of the landscape.

Broadly speaking, it is possible to put the available solutions & technologies into the following categories:

1. User Interface
2. Workflow & Automation
3. Business Logic & Database
4. Integrations & APIs

Let's go through those categories to unfold these somewhat general terms. But before we do that it is necessary to revisit the Abstraction Pyramid from the first blog post, because for each category there are different solutions even in the declarative development space that are representing different levels of abstraction.


### Abstraction Pyramid Revisited

In the next section we will take a look at concrete examples how Salesforce coveres the different categories with multiple solutions even within one category. The reason is that for the different tasks different abstraction levels or "super-powers" are necessary to fulfil the requirement.

In the first blog post I drawed a very high level and oversimplified abstraction pyramid and was putting Salesforce somewhere in there.

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/abstraction-pyramid-salesforce-cuba.png" />

Now we can zoom in a little further and concentrate only on the Salesforce side and try to compare it the CUBA / high-level Java category. 

What we will see then is that actually the closer we look, the more diverse the picture becomes Instead of clean cuts between the abstraction categories with straight lines (red line), you can almost thing of it like an overlap between those categories. 

Also the borders of the pyramid that indicate the transition from the flexible space (white area within the pyramid) to the inflexible space (dashed area outside the pyramid) are not straight lines. Instead it is more an somewhat undefined wavy line that shows a correlation between speed / abstraction and flexibility.

The points (1) - (7) are arbitrary solutions within Salesforce like "Process Builder" (some of them we will see in the examples below). The important point here is, that it is not inheriently dependent on the abstraction category like "in Salesforce everything is more high level and faster" in a black / white fashion. But instead the reality is more shades of grey. It might be that there are solutions in the CUBA world that have the same abstraction level as something in Salesforce (like the BPM add-on) or even higher.

### Example Project: Salesforce Petclinic

All the examples that you will see in this blog post series will be based on an example project: Salesforce Petclinic.

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/sf-petclinic-overview.png" />

This example originated from the Spring Petclinic and was ported extended later implemented with CUBA. The Petclinic application deals with the domain of a Pet clinic and the associated business workflows to manage a pet clinic.

{% include image-left.html image="sf-petclinic.png" width="100px" %}

I took this example domain to show certain concepts in Salesforce but did not implement it thoroughly.

A full running Petclinic example can be found here: [CUBA Petclinic Demo](https://demo10.cuba-platform.com/petclinic/).

### 1. User Interface

Starting with the first category of user interfaces. In Salesforce this can happen in various shapes and forms or better: at various abstraction levels.


#### Page Layouts
At the very top of the abstraction pyramid, let's look first at a detail screen of a Pet:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/pet-details-screen.png" />

This Screen is defined out of the box for every custom object, that is created within Salesforce. It is possible to configure the arrangement of elements within the screen, as well as fields that should be displayed.

This definition of the screen is expressed within a dedicated Setup section of the SF environment. 

The screen arrangement is called "Page Layout". It can be defined within the "Lightning App Builder":

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/pet-details-screen-page-layout.png" />

The way to configure requires a little bit of knowledge about the SF concepts, but besides that, it does not require any kind of coding capabilities.

The components on the left can be dragged on to the page. The list contains standard Salesforce components. Additionally, it is also possible to create Components via imperative development and source code. Then those components also appear in the list and can be leveraged by tech-savvy business people.

#### Flow
The next layer below Page Layouts is something called "Flow". Flow is the capability to define UI workflows/wizards with a brief understanding of programming concepts. 

It still does not require to write source code, but it at least is necessary to have some abstract understanding of the underlying concepts like conditions or variables.  

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/flow.png" />

Instead of defining particular screens/pages, flows main focus is around guided user interactions that are across multiple screens. That being said, it is also possible to create ad-hoc screens that are part of the overall flow directly from here (similar to the way the above-seen Page Builder).

#### Lightning Web Components
Going down further in the abstraction pyramid more into the imperative development with the source code side of things there are also multiple options available.

It allows covering all cases that are not available via the higher-level abstractions. 

Salesforce offered various technological choices that emerged over the years. The newest incarnation is something called "Lightning Web Components". It is a Javascript-based component library available to the developers that are working with Salesforce.

Those components are used by Salesforce itself in a lot of parts to create the out-of-the-box user interfaces. The goal is that there is a common look and feel independent if the component was created by Salesforce or by an external developer.

{%highlight html%}
<lightning-datatable
        key-field="id"
        data={data}
        columns={columns}
        onrowaction={handleRowAction}>
</lightning-datatable>
{% endhighlight %}

This definition (together with some Javascript functionality to load data) will result in the following UI component:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/lightning-web-component-datatable-petclinic.png" />

### 2. Workflow & Automation

Let's turn away a little bit from the user interface part to a related area: Workflow & Automation. 

To create a solution that is more than just a structured way of entering and storing data, it is necessary to create some kind of automation to take away the heavy lifting from the user.

In this area, there are also multiple solutions on different levels of the abstraction pyramid.

#### Process Builder

One very powerful declarative development mechanism in Salesforce is called "Process Builder". What it does is, that it allows defining automation workflows based on a certain environmental event that occurs. 

One arbitrary example: When a Pet is created in the Petclinic application and the Pet is of type "Electric" a new Task is created for an Electric Vet specialist, to review the new Pet. This Task appears in the Vets Task Management with a reference to the Pet.

Automatically after one week, a reminder email is sent. Also, a visit for a regular checkup is scheduled after another week.

Defining these processes is part of the declarative development spectrum as well. Here is a screenshot of the Process Builder configuration user interface:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/process-builder.png" />

It does not require to write source code. That being said it is possible to interact with source code. Oftentimes it will send notifications to the system that then executes a particular piece of source code. 

This intersection is quite common and has the potential to be a real enabler. It allows to let the pieces of the functionality be where they belong the best. Sometimes it is declarative development, sometimes it is imperative development via source code.

Normally such functionality would be embodied in a BMPN workflow engine. Process builder is the proprietary equivalent of that concept, tightly integrated onto other Salesforce capabilities.

### 3. Logic & Database

The next section is all about business logic and the ability to store data. There is also a broad declarative part to it as well as an imperative part.

#### Object Manager

The most obvious declarative mechanism to define data storage is the Object Manager. It allows defining objects (= Database Tables / JPA Entities) and fields of that object. Fields can be either regular data types like text, numbers, checkboxes, picklist but also relationships to other objects.

It is also possible instead of creating custom Objects to adjust existing Objects (to a certain degree). As said before Salesforce comes with certain business objects out-of-the-box like <code>Account</code>, <code>Product</code>, <code>Case</code> etc.

One example of a custom object is the above shown <code>Pet</code> object. To give you a sneak-peak of corresponding Configuration user interface as part of the Object Manager, here is a screenshot of the definition of the fields that are configured for the Pet object:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/object-manager.png" />

#### Calculated Fields

A good example of logic is the concept of calculated / formula fields. When defining an Object in Salesforce it is possible to define fields of different types (numbers, text, checkboxes, etc.). Those fields can either be editable by the user, or they can be calculated. Those fields can execute simple arithmetic operations like sum building, but they can also be defined as complex formulas.

Those Formulars are conceptually very similar to a function in an Excel cell, like <code>=SUM(MIN(A1:A3), MAX(B5:B10))</code>.  Similar to Excel the Salesforce Configuration User Interface supports the user with explanations and a Click-and-Point tool.

The following configuration UI contains a definition of a calculated field for the Pet object. It calculates a reminder date that can be used for sending out email reminders to the Vets about an upcoming Pets Birthday:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/formular.png" />

Defining those formulas requires to have an understanding of this dedicated formula language from the user. But conceptually it only requires the same level of understanding as for defining an Excel calculated cell. As seen in the Screenshot Salesforce tries to help the builder/developer here via inline help e.g.

The concept of formulas is used in different parts of Salesforce. Calculated Fields are just one example. Also, validation rules for object fields are defined similarly.

### 4. Integrations & APIs

In the remaining last big category, it is all about manual and automatic integrations between the Salesforce application and other applications. 

In the space of integration, it starts from something very lightweight like manual / automatic file import. Next, there are various out-of-the-box integrations with external Services like Outlook 365, CTI offerings like Amazon Connect e.g. All those integrations are all still on the declarative side of things. 

Here is one simple example of integration. In this screen a CSV import was started to import new Visits from another system manually:

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/csv-import.png" />

Then in the intersection between declarative development & imperative development, there are integrations with arbitrary Webservices via SOAP / REST.

Besides the Integration side, where the Salesforce application sits more on the consuming side of things it is also possible to let Salesforce be the Provider of data. Manually this happens via various export & reporting functionalities that are part of the standard Salesforce offering. 

When it comes to automated integration it goes into the direction of APIs. Salesforce provides out-of-the-box APIs via SOAP Webservices and further allows them to imperatively develop any kind of APIs to act as the Provider of automated system-to-system integrations.

Through dedicated integration services like Zapier, it is possible to extend that list to thousands of Services.

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-2/zapier-salesforce-to-google-calender.png" />

In this example, an integration is configured through Zapier to create a new Google Calendar Event in case a new Record (e.g. Pet) is created in the Salesforce System.

## It is all about declarative development?

After looking over the declarative technology landscape in Salesforce you could get the impression that this is all only Click-and-Point. 

In fact, although Salesforce covers the main use-cases all with declarative development this is quite far from reality. Instead for almost all use-cases, it is possible to alternatively imperatively develop the solutions through source code.

> The Choice between declarative vs. imperative development boils down to the abstraction level that is necessary to fulfill the business requirement at hand

Declarative development in these configuration UIs has the following characteristics:

* it increases the abstraction level
* it reduces the flexibility
* it speeds up initial development due to higher-level abstractions
* it opens up the specifics to a broader range of builders/developers
* it is much harder to test in an automated fashion

Looking back at our Venn Diagram of the last blog post, for the most part, we have covered the area (1), (2) and a small portion of (3).

<img src="/images/salesforce-through-the-lens-of-a-java-dev-part-1/config-vs-code-ratio-sf.png" class="drawing" />

##### Legend: SF declarative vs. imperative development overlap

1. creating a solution only by configuration
2. creating a solution normally by configuration, code variant possible
3. creating a solution normally by code, sometimes configuration is possible
4. creating a solution only by codes


## Summary

In this blog post, we started with an exploration of the differences between declarative vs. imperative development styles.

After this classification, we took a deeper look into the first class: declarative solutions, that are provided to the builder/developer within the Salesforce ecosystem. Those solutions can be grouped into different areas, namely "User Interface", "Workflow & Automation", "Business Logic & Database" and "Integrations & APIs".

For each of those groups, Salesforce provides different solutions to sub-problems in this area. Those solutions differ mainly in the main target audience, the required technical know-how as well as the abstraction level and inherently therefore also with the flexibility of the solution.

In the next blog post, we will go down the abstraction level further (or you could also say: "go up the flexibility level further") by looking closer in the imperative style of development. So expect some source code in there.