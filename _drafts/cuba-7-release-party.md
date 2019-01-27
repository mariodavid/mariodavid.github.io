---
layout: cuba-7-release-party
title: CUBA 7 Release Party
description: "In this blog post let's celebrate the long awaited big major release 7.0 of CUBA which brings great polished APIs and updates of the underlying libraries."
modified: 2017-04-15
tags: [cuba, business applications]
image:
  feature: cuba-7-release-party/feature.png
---
<img src="/images/cuba-7-release-party/star.png" />

In this blog post let's celebrate the long awaited big major release 7.0 of CUBA which brings great polished APIs and updates of the underlying libraries.

<!-- more -->


## Premium Addons are even cooler with Open Source

<img src="/images/cuba-7-release-party/star.png" />

One of the major things that comes alongside with CUBA 7 is that the previously premium addons (FTS, Reports, BPM etc.) are now open source. Actually the sources of the addons always had been so, that the sources were delivered (and adjustable). But with the new release the existing premium addons are now fully open - accessible on Github.

This is a great addition to the existing open source application components that can be found on Github as well: [#cuba-component](https://github.com/topics/cuba-component).

## IDEA is now the Center of the Universe

<img src="/images/cuba-7-release-party/star.png" />

Two major things happened on the IntelliJ IDEA front.

First one is that CUBA studio is now IntelliJ IDEA. Previously there were always this, at least for developers, odd world of having two development environments that want to deal with the changes you want to make to your application.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-7-release-party/studio-idea.png"><img src="{{ site.url }}/images/cuba-7-release-party/studio-idea.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-7-release-party/studio-idea.png" title="Studio support for Event subscriptions">Studio support for Event subscriptions</a></figcaption>
</figure>


Although the integration and sync between the two worlds were really good, and moreover pretty seamless, sometimes I felt the wish to have the refactoring capabilities / code editing features within studio. On the other side I sometimes wanted to have the understanding of the CUBA specifics inside my IDEA - which was lacking oftentimes.

This is now gone, because it is basically the same application. This is a huge shift and it will enable more higher efficiencies and integrations that previously I wasn't even able to think about. One example would be: now that IDEA gets knowledge about the CUBA specific domain model and DB connection - it could get possible to combine this with the native IDEA features like the DB browser or the JPQL execution environment.

That being said, integrating CUBA Studio into IDEA is a process and I would argue that it is not finished yet. 

I've used the Studio BETA 7 while playing around with BETA 7 of the platform. I have to say that although it has the above mentioned benefits, the previous features of Studio feel a little bit rough. Certain shortcuts within the entity designer are not there e.g.

The majority of screens are already native IDEA screens, which means that all the shortcuts work as expected. I expect this situation to be completely solved in the next minor release of Studio. 

## Interesting Changes in CUBA 7.0

<img src="/images/cuba-7-release-party/star.png" />

Although the tooling side is very important - what the main area of interest for me is the framework itself. What changes have been introduced into CUBA 7.0. There are multiple areas of change - I will highlight my personal favorites.

### New Screens APIs

On the UI layer - a lot has changed. Probably it is the area of the most changes. First, the underlying Vaadin version was bumped from 7 to 8. This causes a lot of internal changes within the platform code. Luckily (in this case), since CUBA UI abstractions hide away the details of the underlying technology, the migration process for the end-user should be close to zero.

But this is only true for the Vaadin changes. CUBA itself has now a lot of re-thought APIs on the UI layer.

#### Composition over Inheritance

The main change that can be observed is the shift away from inheritance to composition. Previously the way to go for a UI controller has been to extend one of the common superclasses like <code>AbstractEditor</code> or <code>AbstractLookup</code>. These classes (or other classes in the inheritance hierarchy) offered certain methods for notifications, screen opening etc.

Inheritance normally created a comparibily high coupling between the superclass and their subclasses. Sometimes this can be benefitial, but oftentimes it is also harmful - especially when it comes to code reuse.

In CUBA 7 this inheritance was mainly exchanged with composition. For notifications features e.g. there is a dedicated <code>Notifications</code> API as a bean. Likewise for screen management there are now the two APIs <code>Screens</code>, <code>ScreenBuilder</code> that can be injected where needed.

Where showing a notification on a Screen pre to CUBA 7 looked like this:

{% highlight java %}
public class CustomerBrowse extends AbstractLookup {

    @Inject
    protected GroupTable<Customer> customersTable;

    public void greetCustomer() {
        Customer customerToGreet = customersTable.getSingleSelected();
        showNotification("Howdy, " + customerToGreet.getInstanceName(), NotificationType.TRAY);
    }
}
{% endhighlight %}

The shift towards delegation changes the code in the following way:

{% highlight java %}

@UiController("cuba6to7changes_Customer.browse")
@UiDescriptor("customer-browse.xml")
@LookupComponent("customersTable")
@LoadDataBeforeShow
public class CustomerBrowse extends StandardLookup<Customer> {

    @Inject
    private Notifications notifications;

    @Inject
    private Messages messages;

    @Inject
    protected GroupTable<Customer> customersTable;

    ...

    public void greetCustomer() {
        Customer customerToGreet = customersTable.getSingleSelected();

        notifications.create(Notifications.NotificationType.TRAY)
                .withCaption(messages.formatMessage(this.getClass(), "greetMessage", ))
                .show();
    }

}
{% endhighlight %}

#### Builders everywhere

Furthermore within those APIs there is a dramatic shift towards the builder pattern. You'll see those all over the place. I personally really like this Fluent API style, because it is easy to read and to compose. It removes the interlectual burden of knowing which parameter index represents which parameter.

An example of this new API pattern can be found in the <code>ScreenBuilders</code> API:


{% highlight java %}
SomeCustomerEditor screen = screenBuilders.editor(Customer.class, this)
    .withScreen(SomeCustomerEditor.class)
    .withListComponent(customersTable)
    .editEntity(customersTable.getSingleSelected())
    .build();
{% endhighlight %}


#### Event based lifecycle interactions

CUBA 7 switches from programmatically registering event listener classes and screen lifecycle hook methods towards a more event based approach.

Methods can be registered to certain events - and the UI components as well as the screens have a lot of those.

Instead of having a hook method <code>init(Map<String, Object> params);</code> all lifecycle interactions are registered with <code>@Subscribe</code>.

CUBA Studio offeres support in order to create those event subscriptions that are available within a particular screen.

<figure class="center">
	<a href="{{ site.url }}/images/cuba-7-release-party/studio-subscribe-to-events.png"><img src="{{ site.url }}/images/cuba-7-release-party/studio-subscribe-to-events.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-7-release-party/studio-subscribe-to-events.png" title="Studio support for Event subscriptions">Studio support for Event subscriptions</a></figcaption>
</figure>

#### Controller Mixins

#### API switch towards Java's functional style




### Data containers replace Datasources

### Catch up with Java

#### Java 8 java.time package

#### Java 11 support


## How the app component ecosystem evoloves
<img src="/images/cuba-7-release-party/star.png" />
