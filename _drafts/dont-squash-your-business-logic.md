---
layout: post
title: Don't squash your business logics
description: "In this blog post let's try to understand where business logic in a CUBA app can and should go - and why you shouldn't squash it. But wait: What is actually business logic? And does it at all have something to do with CUBA?"
modified: 2017-04-15
tags: [cuba, business applications]
image:
  dir: dont-squash-your-business-logic
  feature: cuba-security-subsystem-distilled/feature-2.jpg
---

In this blog post let's try to understand where business logic in a CUBA app can and should go - and why you shouldn't squash it. But wait: What is actually business logic? And does it at all have something to do with CUBA?

<!-- more -->

## What is business logic

A CUBA application, just like any other custom application, has different logic implemented that represents:

* the policies of the domain
* the understanding of the business domain that the application is in
* the rules that are the automation piece

Especially the automation parts are oftentimes the very basis why the application was developed in the first place.

Normally this pieces of functionality are called _business logic_. That term is not always very precise, because oftentimes it has not very much to do with the business itself, but rather with the solution domain of the application - let's call it _solution domain business logic_ for now.

A CUBA application has oftentimes less of _solution domain business logic_ compared to other applications because of the features the platform itself provides. But regarding the real _business logic_ a CUBA application is not different by any means compared to any other application.


### An example of _real_ business logic

To better understand what _real business logic_ means and where it differs from _solution domain business logic_ the following examples should give an option to identify the differences.

{% highlight java %}
public VisitPrice calculateVisitPrice(PetType petType, LocalDate visitStart, LocalDate visitEnd) {

    PriceUnit priceUnit = determineUnitPrice(petType);

    VisitDuration visitDuration = calculateDuration(visitStart, visitEnd);

    VisitPrice visitPrice;

    if (visitDuration.isOneDay()) {
        visitPrice = priceUnit * 1 + FIXED_VISIT_PRICE
    }
    else if (visitDuration.isLessThenAWeek()) {
        visitPrice = priceUnit * visitDuration.getDurationDays() + FIXED_VISIT_PRICE / 2
    }
    else if (visitDuration.isMoreThenAWeek() {
        visitPrice = priceUnit * visitDuration.getDurationDays()
    }

    return visitPrice;
}
{% endhighlight %}

The method `calculateVisitPrice` is an example of _real business logic_. It contains rules that are very much related to the business. Moreover it does not really contain any code that deals with the solution space and constructs of that.

### An example of _solution domain_ business logic

The next example will show a piece of code which in contrast falls more under the category of _solution domain business logic_:


{% highlight java %}
    @Inject
    protected DataManager dataManager;

    @Inject
    protected EmailerAPI emailerAPI;

    @Override
    public int warnAboutDisease(PetType petType, String disease, String city) {

        List<Pet> petsInDiseaseCity = dataManager.load(Pet.class)
                .query("select e from petclinic$Pet e where e.owner.city = :ownerCity and e.type.id = :petType")
                .parameter("ownerCity", city)
                .parameter("petType", petType)
                .view("pet-with-owner-and-type")
                .list();

        petsInDiseaseCity.forEach(pet -> {

            String emailSubject = "Warning about " + disease + " in the Area of " + city;

            Map<String, Serializable> templateParameters = getTemplateParams(disease, city, pet);

            EmailInfo email = new EmailInfo(
                    pet.getOwner().getEmail(),
                    emailSubject,
                    null,
                    "com/haulmont/sample/petclinic/templates/disease-warning-mailing.txt",
                    templateParameters
            );

            emailerAPI.sendEmailAsync(email);
        });

        return petsInDiseaseCity.size();
    }
{% endhighlight %}

This example sends out Disease warning Emails for Owners of pets in particular cities. It contains some _real_ business logic. But is also contains parts that are
dealing with the solution space like the Database or the Email sending mechanisms. Those two examples show what different kinds of business logic are common and where the differences are.

With that differentiation between _real business logic_ and _solution domain business logic_ in mind, this guide will explore in which ways _real_ business logic can be expressed in a CUBA application and what the pros and cons of those options are.


## How business logic can be represented

A CUBA application is a _regular_ Java application. Therefore all options to represent logic in Java are available in a CUBA application as well. Furthermore since a CUBA application uses Spring as a Framework for Dependency Injection, the patterns that Spring suggests are available as well. Additionally CUBA has certain preferences on how to represent business logic as specific artifacts.

What oftentimes happens is that we try to embrace those options in descending order:

1. use CUBAs artifact patterns where ever we can
2. if not possible try to fiddle around with Spring
3. if everything does not work, come back to Java mechanisms

We normally tend to prefer the solution that is closest to the framework we use. The reason is, that when staying close to the framework (like CUBA) we can leverage the most of it.

For _real business logic_ I would propose to turn this way upside down. Start with the Java mechanisms wherever possible, use spring where needed and switch to CUBA artifacts only where it makes sense. Let's get into the why:

## POJOs for real business logic

The easiest and most powerful thing is to put business logic into a class. Just a normal Java class. No Frameworks included. No libraries used.

Let's adjust the example from above to express the business logic in a regular Java class:


{% highlight java %}
class VisitPriceCalculator {
  public VisitPrice calculateVisitPrice(PetType petType, LocalDate visitStart, LocalDate visitEnd) {

      PriceUnit priceUnit = determineUnitPrice(petType);

      VisitDuration visitDuration = calculateDuration(visitStart, visitEnd);

      VisitPrice visitPrice;

      if (visitDuration.isOneDay()) {
          visitPrice = priceUnit * 1 + FIXED_VISIT_PRICE
      }
      else if (visitDuration.isLessThenAWeek()) {
          visitPrice = priceUnit * visitDuration.getDurationDays() + FIXED_VISIT_PRICE / 2
      }
      else if (visitDuration.isMoreThenAWeek() {
          visitPrice = priceUnit * visitDuration.getDurationDays()
      }

      return visitPrice;
  }

}
{% endhighlight %}

As you see: there is nothing to change. How the class is used within the application is a different story, but in general: this is a totally resonable place to put your business logic.

This calculator class can be used anywhere in the application. In a CUBA project setup it is a little more complicated since CUBA uses a multi module project structure. Depending in which module the class was created, it can be used only in this module. But generally it can be used just like this:

{% highlight java %}
VisitPriceCalculator calculator = new VisitPriceCalculator()

VisitPrice visitPrice = calculator.calculateVisitPrice(catType, today, tomorrow)

// do stuff with the calculated visit price

{% endhighlight %}

In the light of everyone is using frameworks these days (just as I am), this option feels a little bit _inferior_, a little _strange_ and _non intuitive_ - at least to me. But this is actually not true. There is nothing that is wrong with this idea.

So where does this fuzzy feeling of not-state-of-the-art comes from?

## Integration points lead to mixing kinds of business logic

Let's look at the integration points. Take a look at the private method <code>determineUnitPrice</code> that is called within this class. When we imagine what this method has to do, we will very quickly end up at the database. But how does it access the database? This is where the transition between the _real_ business logic and the bespoken _solution domain_ business logic comes into play. If we mix this two kinds of business logic the resulting implementation of the method would look like this:

{% highlight java %}
class VisitPriceCalculator {

  @Inject DataManager dataManager

  public VisitPrice calculateVisitPrice(PetType petType, LocalDate visitStart, LocalDate visitEnd) {

      PriceUnit priceUnit = determineUnitPrice(petType);

      // ...
  }

  private PriceUnit determineUnitPrice(PetType petType) {
    dataManager.load(PriceUnit.class)
            .query("select e from petclinic$PriceUnit e where e.type.id = :petType")
            .parameter("petType", petType)
            .one();
  }

}
{% endhighlight %}

In order to do the database lookup, this class needs access to the DataManager facility from CUBA. But the code as it is written down there will not work. The reason is that this magic line <code>@Inject DataManager dataManager</code> will not work as expected. Instead it will not work at all. Not working in this case means: the instance variable <code>dataManager</code> in <code>dataManager.load(PriceUnit.class)</code> will be not initialized and instead be <code>null</code>. This results in a <code>NullPointerException</code> when executing the code.

The reason for that is, that the mechanism called dependency injection done by Spring is not working with this regular Java class (POJO).

To make it work, the most straight forward solution to the problem is to somehow integrate the class with Spring. So what oftentimes happens then is that the class is registered in the Spring context (and we get to that in a second) and with that make it a _Spring bean_ (which is basically a POJO that is registered and managed by Spring). In fact it is a pragmatic approach and it will do the job.

The resulting code looks like this:

{% highlight java %}
import org.springframework.stereotype.Component;
import javax.inject.Inject;
import com.haulmont.cuba.core.global.DataManager;

@Component("myApp_visitPriceCalculator")
class VisitPriceCalculator {

  @Inject DataManager dataManager

  public VisitPrice calculateVisitPrice(PetType petType, LocalDate visitStart, LocalDate visitEnd) {

      PriceUnit priceUnit = determineUnitPrice(petType);

      // ...
  }

  private PriceUnit determineUnitPrice(PetType petType) {
    dataManager.load(PriceUnit.class)
            .query("select e from petclinic$PriceUnit e where e.type.id = :petType")
            .parameter("petType", petType)
            .one();
  }

}
{% endhighlight %}

Pretty similar, right? Right. To point you at the differences, I added the import statements to it. With Spring configured in a way that it will do something called "component scanning", Spring will pick up the class and register it as a _Spring bean_ and with that enable the dependency injection functionality.

## Mixing business logic has disadvantages

But what is oftentimes overlooked is that this decision comes with a cost associated to it. Let's recap what this decision also includes:

* we introduced a compile-time dependency between the VisitPriceCalculator class and a CUBA specific interface called <code>DataManager</code>
* we introduced a logical & compile-time dependency between the VisitPriceCalculator class and the Spring dependency injection framework
* we merged _business logic_ with _solution domain business logic_

Let's go through them one by one and unpack what the problems associated with those are:

### Compile-time dependency to CUBA interfaces

The first thing would be that we introduce a dependency between the POJO <code>VisitPriceCalculator</code> and the CUBA interface <code>DataManager</code>.  This means, that in order to _compile_ and with that _use_ the class <code>VisitPriceCalculator</code> it is always required to also ship the code CUBA platform framework in the code as well. Without this dependency it is not possible to execute the source code any more.

Let's once again think through what <code>VisitPriceCalculator</code> should do: it should calculate prices for visits. But calculating prices for visits in itself does have nothing to do with any technology. It is basically just an expression of business rules defined in a programming language.


When we look at a specific part of software development, this _dependency problem_ becomes very visible: testing. When you want to unit test the business rules defined by <code>VisitPriceCalculator</code>, you also have to make sure the <code>DataManager</code> object has to be initiated properly. In a unit test, this is hardly possible. There are two solutions to this problem:

1. start up a integration test context
2. mock the <code>DataManager</code> interface to emulate its behavior

But when you "just" want to test your business rules, why do you have to spin up a integration test context? Why do you need to spin up the database? But this is oftentimes the next logic step, that leads down to a highly coupled software which cannot live without its framework anymore.

### Logic dependency to Spring

With introducing the <code>javax.inject.Inject</code> annotation we basically introducted a compile-time dependency to an injection mechanism. It logically means that there is a dependency to Spring (more or less). This kind of dependency is basically the same as the above with the CUBA interfaces.

Compared to the CUBA interfaces, this dependency is a little more robust (especially through using java standard annotation instead of the Spring specific ones like <code>@Autowired</code>), but still it is there.

The dependency is weaker than the <code>DataManager</code> dependency, especially because it is just an annotation. It means, that e.g. in a unit test we still can manually create an instance of the POJO and set all the dependencies as constructor parameters. The dependency injection will just be ignored in this case.

But still: it means, that your code can only be compiled with the Spring framework is in place (through the annotation: <code>org.springframework.stereotype.Component</code>).

The same question applies here: does the _real_ business logic of calculating prices has anything to do with the Spring framework? No, it doesn't.

### Squashing of _real_ and _solution domain_ business logic

The underlying problem of those dependencies is that we allowed ourselves to merge the two concepts of _real_ and _solution domain_ business logic.

This tangling of the two concepts leads to the situation where you cannot differentiate between those two concepts clearly anymore. When doing that in a decent sized application, it feels like _the framework is eating your application_.

The problem is that it is so easy to do it. Therefore it is oftentimes the default choice. Also: when we look at the example from above - one could ask: _now what? - who cares_? From a pragmatic point of view this is legit.

The whole discussion deals with the questions of what _software architecture_ is all about. One of the main parts of software architecture is _dependency management_. Managing dependencies in general means keeping track of dependencies between services / modules / classes in the application and deal with them in a way that allows good maintainability over time.

In the concrete this means, that there should be as little dependencies between the parts as possible. Furthermore it means that there should not be arrows from every service / module / class to every other service / module / class in a corresponding dependency diagram.

## Separate instead of squash

Let's try to organize the class and its dependencies in a way, that keeps the _real_ business logic different from _solution space_ domain logic. When we look at the dependency to the <code>DataManager</code> class, why is it there? It is there, because the <code>VisitPriceCalculator</code> also tries to load the data from a datasource. We can turn that around, because as the name of the class already states: it should calculate the price, not load the data and calculate.

This in fact is a violation of the single responsibility. So let's get rid of it. Instead we will pass in the data into the method:


The resulting code looks like this:

{% highlight java %}
package com.rtcab.cuba.my_app.real_business_logic;

class VisitPriceCalculator {
  public VisitPrice calculateVisitPrice(PriceUnit priceUnit, LocalDate visitStart, LocalDate visitEnd) {

      VisitDuration visitDuration = calculateDuration(visitStart, visitEnd);

      VisitPrice visitPrice;

      if (visitDuration.isOneDay()) {
          visitPrice = priceUnit * 1 + FIXED_VISIT_PRICE
      }
      else if (visitDuration.isLessThenAWeek()) {
          visitPrice = priceUnit * visitDuration.getDurationDays() + FIXED_VISIT_PRICE / 2
      }
      else if (visitDuration.isMoreThenAWeek() {
          visitPrice = priceUnit * visitDuration.getDurationDays()
      }

      return visitPrice;
  }
}
{% endhighlight %}

This also requires to have a code snippet that will still interact with the database and load the data. We will extract that method into its own class that is onlys responsibility is to load the data. This class is part of the solution space business logic.


{% highlight java %}
package com.rtcab.cuba.my_app.solution_domain_business_logic;

import org.springframework.stereotype.Component;
import javax.inject.Inject;
import com.haulmont.cuba.core.global.DataManager;

@Component("myApp_visitPriceUnitFetcher")
class VisitPriceUnitFetcher {

  @Inject DataManager dataManager

  public PriceUnit determineUnitPrice(PetType petType) {
    dataManager.load(PriceUnit.class)
            .query("select e from petclinic$PriceUnit e where e.type.id = :petType")
            .parameter("petType", petType)
            .one();
  }
}
{% endhighlight %}

As you see: we now have separated the solution domain business logic from the real business logic. The last step is that we need to have a class that combines the two worlds. This class in fact will need to have one foot in the solution domain, because it needs to interact with the <code>VisitPriceUnitFetcher</code> which is a spring bean and so on. It could look like this:

{% highlight java %}
package com.rtcab.cuba.my_app.integration_of_the_two_worlds;

import org.springframework.stereotype.Component;
import javax.inject.Inject;
import com.haulmont.cuba.core.global.DataManager;

@Component("myApp_visitPriceOrchestrator")
class VisitPriceOrchestrator {

  @Inject VisitPriceUnitFetcher visitPriceUnitFetcher

  public VisitPrice calculateVisitPrice(PetType petType, LocalDate visitStart, LocalDate visitEnd) {
      PriceUnit priceUnit = visitPriceUnitFetcher.determineUnitPrice(petType);
      VisitPriceCalculator priceCalculator = new VisitPriceCalculator();

      return priceCalculator.calculateVisitPrice(priceUnit, visitStart, visitEnd);
  }
}
{% endhighlight %}

With this change we have accomplished the following aspects:

* the _real_ business logic is encapsulated from all dependencies to frameworks
* the _solution space_ business logic is still the doing its thing, but independent of the business rules
* the *thin* orchestration layer combines the two worlds
* the _real_ business logic can be tested in isolation without mocking
* the _real_ business logic can be tested in isolation without integration testing
* the _solution space_ business logic can still be tested in an integration style

There are also other possibilities to do the orchestration layer or don't even have one at all and instead directly call the real business logic from the solution domain business logic. There is a whole field of ideas around it that have been there for quite some time. One very sophisticated example of it is "Clean architecture" from Uncle bob.

## What about the domain entities?

If you looked carefully over the example, you will notice that there is a dependency between the two worlds, that actually sits in the _solution space_ business logic and is required from the _real_ business logic to do its job. It is the complete entity layer. Why is this the case?

Well, this actually is a hard trade off. Above I said it is mainly about dependency inversion. There should be no dependency from the _real_ business logic to the _solution space_ business logic. If this dependency is there, what is the whole point of not squashing? Because in order to compile the real business logic, it needs the entity layer, which is baked into the _solution space_ business logic.

This is right. In order to drive that out what we would need to do is to try to carve out the entity layer from the _solution space_ business logic. But when you look at CUBA - this is literally impossible. CUBA heavily relies on the entity layer as everything is built on top of it. The DB tables are generated from it, the UI is created based on it and so on.

Even if this is accomplishable - it would remove the whole point of CUBA all together. To be fair, this is also true for a lot of other "Full stack frameworks".

So what alternatives are there? There is one. It is based on the idea to create an entity-interface layer for your entities. This interface layer lives in the _real_ business logic. In the _real_ business logic code you will only interact with those interfaces.

In the _solution space_, where the real entities live, they now implement those interfaces. This way, once again, we have achieved an inversion of the dependencies.

The UML representation of this change would look like this:


{% include hover-image.html image="entity-interface-wrong.png" class="overview" description="Dependencies between classes before the dependency inversion" %}


{% include hover-image.html image="entity-interface-right.png" class="overview" description="Dependencies between classes after the dependency inversion" %}


Note, that this architectural changes does not come for free. It adds additional burden, especially if there are a lot of entities. Therefore it is not a silver bullet. 

### Summary 

The above mentioned solution for the entity dependency problem  a very good reminder that there are no easy choices when it comes to architecture decisions. Architecture is a set of trade-offs that need to be taken into consideration.

Generally, actively thinking about architecture, dependencies between classes, modules and so on is the real value here. Only because with CUBA you are in a full stack framework that offers a lot out of the box does not mean that we cannot emancipate from the framework. Applying proper software architecture gives us a way out of a way to _the framework eats my application_ and protects our most important business logic and treats is like a real asset that is worth carving out properly.