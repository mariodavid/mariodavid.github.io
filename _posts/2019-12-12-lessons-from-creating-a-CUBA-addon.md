---
layout: post
title: Lessons from creating a CUBA addon
description: "This blog post we will walk through the process of creating an application component for CUBA. It is based on the recently released default-values addon, that provides the ability to configure default values for entity attributes at runtime. You will learn something about the default-values internal implementation as well as some internals of the CUBA."
modified: 2019-12-12
tags: [cuba, default values]
image:
  feature: lessons-learned-from-creating-a-cuba-addon/feature.png
---

This blog post we will walk through the process of creating an application component for CUBA. It is based on the recently released default-values addon, that provides the ability to configure default values for entity attributes at runtime.

You will learn something about the default-values internal implementation as well as some internals of the CUBA framework.

<!-- more -->

### Identify a Pattern

<img src="/images/lessons-learned-from-creating-a-cuba-addon/pencils/green-180.png" style="height:30px;" />

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

The solution pattern was always very similar - because the problem at hand was very similar. Normally as this is logic expressed as source code I also implemented unit tests for this piece of code.

One realization that I had when implementing the problem space of default value population over and over again is that actually the programmatic approach was cumbersome and it almost felt that there was some missing declarative abstraction.

Furthermore sometimes there were requirements that actually business users would want to change this population of default values in the software development process or as a user of the application there was a need to configure those values based on the use-case / context the current user is in.

Also - and this is probably very crucial when going through the process of creating a application component or any kind of abstraction: I was bored by doing that same-looking task over and over again and the pain of doing that boring task brought me to the point of thinking about a proper abstraction.

### A Mental Model over the Common Problem

<img src="/images/lessons-learned-from-creating-a-cuba-addon/pencils/brown-180.png" style="height:30px;" />

So I gave it a try to crystallize the idea about having a dedicated application component that deals with default value population.

What I wanted to achieve is that it should be possible to configure default values via a administrative UI that allows tech-savy business users / administrators / developers to define what values should be set by default for a particular entity / screen.

#### The Value of CUBA Abstraction Layers

Besides the administrative UI for setting up the default values, the main part of the application component was to actually populate the values.

Initially I was thinking about a UI annotation based approach as I did previously in various application components (think of injecting a field in the screen controller and add a additional annotation on it to declare that it has a certain value by default). Then it would at least be a little more declarative, but still only accessible to developers.

But this was not the main problem. The main problem was that when restricting the solution to the UI layer, you would have a inconsistent solution in your overall application, because there are multiple other ways to create an entity instance then just the specific UI controller with the correct annotation:

* creating an instance via the REST API
* using CUBAs entity inspector
* leveraging the data-import application component to create entities
* having a react based frontend for entity creation
* using any kind of service business logic to create instances

Luckily it became clear that CUBA with its general tedency to create thin layers of abstractions on "everything" was helping our here very much.

The way you create an entity instance in CUBA is not by calling the constructor of the entity like this: <code>Customer customer = new Customer();</code> but instead by using a factory method:<code>Customer customer = metadata.create(Customer.class);</code>.

The difference is only very small for the calling code, but it enables a whole lot functinality that could otherwise not be put under this abstraction. E.g. the following use-cases are executed during the entity creation process of <code> metadata.create()</code>:

* assign a unique ID value
* finding the actual class that should be instanciated through entity inheritance
* executing <code>@PostConstruct</code> methods on the instance

There are several more possibilities that could be executed there, e.g. if it is possible in this user context to actually create instances of this entity.

One additional use-case that can be executed in there: <i>Default Value population</i>.

#### Extending CUBA by Inheritance

Having that abstraction in place allows to easily inject this new behavior into the system without changing any of the calling code. It would have just not been possible to create this cross cutting functionality into the system, if this abstraction would not be there.

But with the existence of the <code>Metadata</code> abstraction, is should be possible to inject my desired functionality and not only - not breaking all other parts of the system, but also to automatically include the other parts with my new functionality.

When overriding the behavior of the <code>Metadata</code> API, I would immediately have the default value behavior also in the REST API e.g.

So I went ahead and created a new class that extends the default API implementation <code>MetadataImpl</code>:

{%highlight java%}
public class MetadataWithDefaultValuesSupport extends MetadataImpl {

    @Override
    protected <T> T create(Class<T> entityClass) {
        T entityInstance = super.create(entityClass);

        // here comes the default value logic
        initDefaultValues((Class<Entity>) entityClass, (Entity) entityInstance);

        return entityInstance;
    }
}
{% endhighlight %}

#### Replacing CUBA Default Behavior with Dependency Injection

Overriding the behavior of the Metadata API has to be elaborated a litte bit here, because it is the omnipresent underpinning of almost all CUBAs APIs.

CUBA is based on the Spring framework. Spring is a at its core a Dependency Injection framework. Dependency Injection (DI) is a the process of Spring taking over the instanciation of abitraty objects in the application (just like <code>metadata.create()</code> itself for Entity instances).

Instead of the calling class doing a <code>Metadata metadata = new MetadataImpl();</code> the user simply declares a Field (or a constructur parameter) and annotates it with <code>@Inject</code>. This leads Spring to instanciate an instance on behalf of this class and _injects_ the instance into the target class.

This is where I got lucky once again. In case every class (within CUBA itself or within any CUBA application) that would have written the instanciation of the Metadata API like this: <code>Metadata metadata = new MetadataImpl();</code> - I would not have been able to somehow replace <code>MetadataImpl</code> with my subclass <code>MetadataWithDefaultValuesSupport</code>.

Since CUBA is a Spring application, I can rely on the fact that the Metadata API is injected into the calling code.

Since the class instanciation is delegated to a Spring mechanism in a dedicated place, Spring provides the ability to influence the creation of the <code>Metadata</code> instance.

The way it works is that there is a declarative definition of which class should be used to provide this _Bean_ (this is a Spring Term and basically means a Class / Interface that is registered in the Spring application context, that can be injected). This definition for a CUBA application is normally in the <code>spring.xml</code> / <code>web-spring.xml</code>. What I had to do is that defined under the bean name a differernt class (my class) like this:

{%highlight xml%}
<bean
    id="cuba_Metadata"    
    class="de.diedavids.cuba.defaultvalues.MetadataWithDefaultValuesSupport"
/>
{% endhighlight %}

With that single line now all classes that require an instance of the <code>Metadata</code> Interface via Injection will receive my Implementation - Luckily CUBA uses Spring...

Also I was able to define that overriding rule for the <code>Metadata</code> instantication only in the application component. Applications that now use my application component, will automatically inherit this configuration without any further action.

### Creating a Management UI

<img src="/images/lessons-learned-from-creating-a-cuba-addon/pencils/orange-180.png" style="height:30px;" />

With the default values population injection point in place, the remaining question was - how to configure the values. In particular in the UI there was one interesting part. The input fields where the default value is configure needs to be of the correct type based on the MetaProperty of the Entity attribute.

For a <code>Order.orderDate</code> attribute, it needs to be a Date Picker, where the Customer's name should be of type String and the Customer Type needs to show a LookupField with all of the possible <code>CustomerType</code> Enum options.

CUBA 7.1 released a functionality that helped out here heavily. With the introduction of the <code>InputDialog</code> through the <code>Dialogs</code> API, it is possible to create a dialog window, that depending on the provided parameters renders the correct input fields.

This is a very good enhancement, because it is very easy to create an on-the-fly input form programmatically without creating a dedicated and static screen.

#### Split mixed-up usages and API

In the scenario of the default values, instead of having the different datatypes, I had the <code>MetaProperty</code> of the entity attribute at hand.

Taking that idea a little further I noticed that it would actually be a good standalone enhancement to create a input parameter out of a <code>MetaProperty</code>. Currently this is not possible with the <code>InputDialog</code> functionality.

With that realization I refactored the code that provided the dialog window to set the entity attribute default value. Initially there was a mixed-up implementation where the input dialog creation, datatype determination and usage was all combined.

This turned into one dedicated API I called <code>EntityDialogs</code> that only has one purpose: create an <code>InputDialog</code> with a <code>MetaProperty</code> as the parameter.


### Extend CUBA functionality by Delegation

<img src="/images/lessons-learned-from-creating-a-cuba-addon/pencils/blue-180.png" style="height:30px;" />

During the creation of this particular API two worth mentioning points came up.

The first one was the question on how to extend the existing <code>Dialogs</code> API of CUBA. Unfortunately this time it was not as straight forward as with the <code>Metadata</code> API from above. Instead of injecting new functionality into the same API I needed to do some API extensions.

So I needed to created a new interface that would extend or at least mirror the interface of the existing <code>Dialogs</code> API.

With extension of the interface there were some problems, because of how internally the dependency injection mechanism works in the web module of a CUBA application.

One learning that I had here is that although CUBA creates an almost perfect imagination that Spring is taking care of the Depenency Injection on the _web_ module as it does in the _core_ module, this is not entirely true.

At least for the Controller classes CUBA itself has taken over the DI mechanism. Form the API layer it is still the same - you just declare a dependency via <code>@Inject</code> and magically the correct insance appears in the controller.

Ultimately the <code>EntityDialogs</code> API ended up like this:


{%highlight java%}
public interface EntityDialogs {

    <E extends Entity> EntityInputDialogBuilder<E> createEntityInputDialog(FrameOwner frameOwner, Class<E> entityClass);

    interface EntityInputDialogBuilder<E extends Entity> {

        EntityInputDialogBuilder withParameter(EntityAttributeInputParameter inputParameter);

        EntityInputDialogBuilder withParameters(EntityAttributeInputParameter... inputParameters);

        EntityInputDialogBuilder withEntity(E entityInstance);

        // ...

        InputDialog show();

        InputDialog build();

    }
}
{% endhighlight %}

Internally this API basically acts as a _remote-control_ facility of the Dialogs API with some specific tailored usage in mind. It is using the Dialogs API and the <code>InputDialogBuilder</code> to create a specific use case. A lot of methods like <code>withValidator(Function<InputDialog.ValidationContext, ValidationErrors> validator)</code> are directly forwarded to the underlying input dialog builder.

To sum up this piece of extensions: compared to the API extension from above, this time I used delegation instead of inheritance. This allowed for more freedom in the implementation, since the two classes / interfaces (Dialogs and EntityDialogs) are more loosly coupled. On the other hand it required a little more code duplication compared to an inheritance based approach.

In the end the <code>EntityDialogs</code> API alongside with a couple of other classes ended up not directly in the default-values application component. Instead I decided that since this functionality is quite common and I (and others) could re-use it in different application components.

This is why I created another application component that hosts this functionality: [metadata-extensions](https://github.com/mariodavid/cuba-component-metadata-extensions).



### The Metadata Underpinning

<img src="/images/lessons-learned-from-creating-a-cuba-addon/pencils/purpule-180.png" style="height:30px;" />

As you already saw in the EntityDialogs example - I leveraged the Metadata Subsystem of CUBA. It is one of the less common known / more advanced capabilities of the framework.

During the development of the default-values application component I took a deep look into it and found that there is a lot to uncover.

The Metadata Subsystem (besides the fact that it has a scary name right there) - is basically nothing more then a programmatic interface, that allows to introspect the data model of the application.

One example is the way to show a String input Field in case the Entity attribute is of type String.

I will not go into any more detail about this right here, but I can only encourage you that in case you have the need to either get some information about the entity model in a programmatic fashion of needs to execute different logic based on the types of the entity at hand - the Metadata Subsystem will got your back.


### Wrap Up
<img src="/images/lessons-learned-from-creating-a-cuba-addon/pencils/red-180.png" style="height:30px;" />

To wrap up this piece of experience write down: I explained a couple of ways how to extend CUBA by Spring bean replacement or Delegation. Also we learned about some capabilities of CUBA that are not super common in the everyday use but still very powerful.

The journey for this particular application component does not end here, as there are some other parts that are not yet explored by me. E.g. the ability to cache the default value configurations somehow so that loading the default values do not impact the performance of the overall application that much.

You can check out both application components here:

* [default-values](https://github.com/mariodavid/cuba-component-default-values)
* [metadata-extensions](https://github.com/mariodavid/cuba-component-metadata-extensions)



<style type="text/css">
  article.hentry {
    background-color:#a7d6dc;
  }

  .entry-content, .read-more, section#disqus_thread {

    background-color:#e8f3f5 !important;
    color:#000;
    border: 0px solid #333;

  }
  h1, h2, h3, h4 {
    color: #74979b;
  }

    .entry-content h1 {
      margin-bottom:-20px;
    }

      .entry-content h2 {
        margin-bottom:-20px;
      }

        .entry-content h3 {
          margin-bottom:-20px;
        }

  .entry-content a {
    color: #74979b;
  }

  .read-more h3 > a, .read-more h4 > a, .read-more  a {
    color: #74979b;
  }

.entry-content a:hover, .entry-content a:active {
    color: #74979b;

  }

  code {
    color:#000;
  }

</style>
