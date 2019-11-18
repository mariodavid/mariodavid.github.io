---
layout: cuba-7-release-party
title: Help your users with Default Values
description: "In this blog post let's go through the beauty of pre-populated values for data entering, what options are available and how the default-values application component takes away the heavy lifting..."
modified: 2019-11-18
tags: [cuba, default values]
image:
  feature: cuba-7-release-party/feature.png
---

In this blog post let's go through the beauty of pre-populated values for data entering, what options are available and how the default-values application component takes away the heavy lifting...

<!-- more -->

### Going from plain CRUD to "smart apps"

### Default Value Definition is a common problem

Normally I used to develop default values in a CUBA application either directly in the UI controller code or in a <code>@PostConstruct</code> annotated method within the Entity class itself.

It normally looked like this for an Entity:

{%highlight java%}
public class Visit extends StandardEntity {

    @PostConstruct
    private void initVisitDate() {
        if (visitDate == null) {
            setVisitDate(today());
        }
    }
}
{% endhighlight %}

Or on the UI layer more like this:

{%highlight java%}
public class RegularCheckup extends StandardEditor<Visit> {

    @Subscribe
    protected void initRegularCheckupVisit(
        InitEntityEvent<Visit> event
    ) {
        Visit visit = event.getEntity();
        visit.setPaid(false);
    }
}
{% endhighlight %}

But the pattern was always very similar - because the problem at hand was very similar.

Normally as this is logic expressed as source code I also implemented unit tests for this piece of code.

One realization that I had when implementing the problem space of default value population over and over again is that actually the programmatic approach was cumbersome and it almost felt that there was some missing declarative abstraction.

Furthermore sometimes there were requirements that actually business users would want to change this population of default values in the software development process or as a user of the application there was a need to configure those values based on the use-case / context the current user is in.

### Introducing the default-values application component

Lately I gave it a try to cristalize the idea about having a dedicated application component that deals with default value population.

What I wanted to achieve is that it should be possible to configure default values via a administrative UI that allows tech-savy business users to define what values should be set by default for a particular entity / screen.  For developers / administrators of the system configuring this behavior would be a little easier and faster instead of doing the coding of this very same fact for the regular case.

#### The Value of CUBA abstraction layers

Besides the administrative UI for setting up the default values, the main part of the application component was to actually populate the values.

Initially I was thinking about a UI annotation based approach as I did previously in various application components. But the problem was that when restricting the solution to the UI layer, you would have a inconsistent solution in your overall application, because there are multiple other ways to create an entity instance then just the specific UI controller with the correct annotation:

* creating an instance via the REST API
* using CUBAs entity inspector
* leveraging the data-import application component to create entities
* having a react based frontend for entity creation
* using any kind of service business logic to create instances

Luckily it bacame clear that CUBA with its general tedancy to create thin layers of abstractions on everything was helping our here very much.

The way you create an entity instance in CUBA is not by calling the constructor of the entity like this: <code>Customer customer = new Customer();</code> but instead by using a factory method:<code>Customer customer = metadata.create(Customer.class);</code>.

The difference is only very small for the calling code, but it enables a whole lot functinality that could otherwise not be put under this abstraction. E.g. the following use-cases are executed during the entity creation process of <code> metadata.create();</code>:

* assign a unique ID value
* finding the actual class that should be instanciated through entity inheritance
* executing <code>@PostConstruct</code> methods on the instance

There are several more possibilities that could be executed there, e.g. if it is possible in this user context to actually create instances of this entity.

One additional use-case that can be executed in there: <i>Default Value population</i>.

Having that abstraction in place allows to easily inject this new behavior into the system without changing any of the calling code. It would have just not been possible to create this cross cutting functionality into the system, if this abstraction would not be there.

But with the existence of the <code>Metadata</code> abstraction, is should be possible to inject my desired functionality and not only - not breaking all other parts of the system, but also to automatically include the other parts with my new functionality. When overriding the behavior of the <code>Metadata</code> API, I would immediately have the default value behavior also in the REST API e.g. This is the power of Abstractions. 
