---
layout: cuba-7-release-party
title: CUBA 7 Release Party
description: "In this blog post let's celebrate the long awaited big major release 7.0 of CUBA which brings great polished APIs and updates of the underlying libraries."
modified: 2019-01-31
tags: [cuba, cuba 7]
image:
  feature: cuba-7-release-party/feature.png
---
<img src="/images/cuba-7-release-party/star.png" />

In this blog post let's celebrate the long awaited big major release 7.0 of CUBA which brings great polished APIs and updates of the underlying libraries.

<!-- more -->

CUBA 7 offers a lot of new APIs and also some new concepts. In this blog post, I will guide you through my favorite changes.


## CUBA 7 Release Party Example Application

<img src="/images/cuba-7-release-party/star.png" />

To see the new concepts live in action, you can also find an example application on Github, that deals with the domain of party management:

{% include github-example.html repository="cuba-7-release-party" %}


<figure class="center">
	<a href="{{ site.url }}/images/cuba-7-release-party/overview-cuba-7-release-party-app.gif"><img src="{{ site.url }}/images/cuba-7-release-party/overview-cuba-7-release-party-app.gif" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-7-release-party/overview-cuba-7-release-party-app.gif" title="CUBA 7 Relese Party example application">CUBA 7 Relese Party example application</a></figcaption>
</figure>


The application is built in CUBA 6 and CUBA 7 simultaneously. The sources are in two different git branches. This way, you can switch between the branches in the same file in the Github UI directly and see the differences between the two major versions.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-7-release-party/switch-cuba-versions-in-github.gif"><img src="{{ site.url }}/images/cuba-7-release-party/switch-cuba-versions-in-github.gif" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-7-release-party/switch-cuba-versions-in-github.gif" title="Switch example code between CUBA 6 and CUBA 7 in Github">Switch example code between CUBA 6 and CUBA 7 in Github</a></figcaption>
</figure>


## Premium Addons for the Masses

<img src="/images/cuba-7-release-party/star.png" />

One of the major things that comes alongside with CUBA 7 is that the  Premium Addons (FTS, Reports, BPM etc.) are now open source. Actually, the sources of the addons have always been available (and adjustable). But with the new release the existing premium addons are now fully open - accessible on Github. And as a nice side effect: they are free of charge.

This is a great addition to the existing open source application components that can be found on Github as well: [#cuba-component](https://github.com/topics/cuba-component).

## IDEA is now the Center of the Universe

<img src="/images/cuba-7-release-party/star.png" />

Two major things happened on the IntelliJ IDEA front.

First one is that CUBA Studio *is* now IntelliJ IDEA. Previously there was always this, at least for developers, odd world of having two development environments that want to deal with the changes you want to make to your application.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-7-release-party/studio-idea.png"><img src="{{ site.url }}/images/cuba-7-release-party/studio-idea.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-7-release-party/studio-idea.png" title="CUBA Studio based on the IntelliJ IDEA platform">CUBA Studio based on the IntelliJ IDEA platform</a></figcaption>
</figure>

Although the Integration and Sync between the two worlds were really good and pretty seamless, sometimes I had the wish to have the refactoring capabilities / code editing features within Studio. On the other side, I sometimes wanted to have the understanding of the CUBA specifics inside my IDEA - which was lacking oftentimes.

This is now gone, because it is basically the same application. This is a huge shift, and it will enable much higher efficiencies and integrations that previously I wasn't even able to think about. One example would be: now that IDEA gets knowledge about the CUBA specific domain model and DB connection - it could get possible to combine this with the native IDEA features like the DB browser or the JPQL execution environment.

That being said, integrating CUBA Studio into IDEA is a process, and I would argue that it is not finished yet.

I've used the Studio BETA 7 while playing around with BETA 7 of the platform for quite some time. I have to say that although it has the above mentioned benefits, the previously existing features of Studio feel a little bit rough. Certain shortcuts within the entity designer are not there e.g.

A lot of screens are already native IDEA screens, which means that all the shortcuts work as expected. I expect this situation to be completely solved in the next minor release of Studio.

## Interesting Changes in CUBA 7.0

<img src="/images/cuba-7-release-party/star.png" />

Although the tooling side is very important - what the main area of interest for me is the framework itself. What changes have been introduced into CUBA 7.0. There are multiple areas of change - I will highlight my personal favorites.

### URLs become First Class Citizens

URLs and CUBA up until now always seemed to always be a love-hate relationship. CUBA actually offered support for URL handling at least for deep links. But URL changes have never been a thing. With CUBA 7 and the underlying Vaadin 8, this now became possible.

CUBA 7 offers a programmatic API to reflect changes in the UI state also in the URL in the browser address bar.

This is one of my personal favorites, because it brings some of the main features of the Web to CUBA land.

### New Screens APIs

On the UI layer - a lot has changed. Probably it is the area of the biggest changes. First, the underlying Vaadin version was bumped from 7 to 8. This causes a lot of internal changes within the platform code. Luckily (in this case), since CUBA UI abstractions hide away the details of the underlying technology, the migration process for the end-user should be close to zero.

But this is only true for the Vaadin changes. CUBA itself has now a lot of re-thought APIs on the UI layer.

#### Composition over Inheritance

The main change that can be observed is the shift away from inheritance to composition. Previously the way to go for a UI controller has been to extend one of the common superclasses like <code>AbstractEditor</code> or <code>AbstractLookup</code>. These classes (or other classes in the inheritance hierarchy) offered certain methods for notifications, screen opening etc.

Inheritance normally created a comparatively high coupling between the superclass and their subclasses. Sometimes this can be beneficial, but oftentimes it is also harmful - especially when it comes to code reuse.

In CUBA 7 this inheritance was mainly exchanged with composition. For notifications features e.g. there is a dedicated <code>Notifications</code> API as a bean. Likewise for screen management there are now the two APIs <code>Screens</code>, <code>ScreenBuilder</code> that can be injected where needed.

Where showing a notification on a Screen pre to CUBA 7 looked like this:

{% highlight java %}
public class PartyBrowse extends AbstractLookup {

  @Inject
  protected GroupTable<Party> partiesTable;

  public void startParty() {

    Party partyToStart = partiesTable.getSingleSelected();
    String themeName = messages.getMessage(partyToStart.getTheme());
    String startPartyMessage = formatMessage("startPartyMessage", partyToStart.getTitle(),
        themeName);

    showNotification(startPartyMessage, NotificationType.WARNING);
  }
}
{% endhighlight %}

The shift towards delegation changes the code in the following way:

{% highlight java %}

@UiController("c7rp_Party.browse")
@UiDescriptor("party-browse.xml")
@LookupComponent("partiesTable")
@LoadDataBeforeShow
public class PartyBrowse extends StandardLookup<Party> {

    @Inject
    private Notifications notifications;

    @Inject
    private Messages messages;

    @Inject
    protected MessageBundle messageBundle;
    
    @Inject
    protected GroupTable<Party> partiesTable;


    public void startParty() {
        Party partyToStart = partiesTable.getSingleSelected();
        String themeName = messages.getMessage(partyToStart.getTheme());

        String startPartyMessage = messageBundle.formatMessage("startPartyMessage", partyToStart.getTitle(), themeName);

        notifications.create(Notifications.NotificationType.WARNING)
                .withCaption(startPartyMessage)
                .show();
    }

}
{% endhighlight %}

#### Builders Everywhere

Furthermore, within those APIs there is a dramatic shift towards the builder pattern. You'll see those all over the place. I personally really like this Fluent API style, because it is easy to read and to compose. It furthermore removes the intellectual burden of knowing which parameter index represents which parameter.

An example of this new API pattern can be found in the <code>ScreenBuilders</code> API:


{% highlight java %}
SomePartyEditor screen = screenBuilders.editor(Party.class, this)
    .withScreen(SomePartyEditor.class)
    .withListComponent(partiesTable)
    .editEntity(partiesTable.getSingleSelected())
    .build();
{% endhighlight %}


#### Event Based Lifecycle Interactions

CUBA 7 switches from programmatically registering event listener classes and screen lifecycle hook methods towards a more annotation-based approach. Methods can be registered to certain events - and the UI components, as well as the screens, have a lot of those. The two main Annotations are <code>@Subscribe</code> and <code>@Install</code>

One example would be: Instead of having a hook method <code>init(Map<String, Object> params);</code> you register a method via subscribe <code>@Subscribe</code> like this:


{% highlight java %}
public class PartyEdit extends StandardEditor<Party> {

    @Subscribe
    protected void onInitEntity(InitEntityEvent<Party> event) {
        event.getEntity().setTheme(Theme.STAR_WARS);
    }

}
{% endhighlight %}


CUBA Studio offeres support in order to create those event subscriptions that are available within a particular screen.

<figure class="center">
	<a href="{{ site.url }}/images/cuba-7-release-party/studio-subscribe-to-events.png"><img src="{{ site.url }}/images/cuba-7-release-party/studio-subscribe-to-events.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-7-release-party/studio-subscribe-to-events.png" title="Studio support for Event subscriptions">Studio support for Event subscriptions</a></figcaption>
</figure>

#### Controller Mixins

Another very interesting feature is the possibility to use Interfaces as Mixins for controller functionality. With the Release of Java 8 and the ability for interfaces to have default method implementations, the way was open to allow the Mixin pattern in the Java world as well. Mixins are a concept available in multiple languages like Ruby.

It is basically a mechanism which allows injecting functionality from different places into a class, without having to inherit from another class. Therefore it basically allows the same functionality what multiple inheritances provide but without the downsides.

CUBA 7 adopted this pattern to use it in Controllers. The usage of this mechanism would look like this:

{% highlight java %}
public class PartyEdit extends StandardEditor<Party> implements Commentable, Attachable {

    @Subscribe
    protected void onInitEntity(InitEntityEvent<Party> event) {

        // call your code from Commentable or Attachable Interfaces here
    }

}
{% endhighlight %}

<code>Commentable</code>, as well as <code>Attachable</code>, are both interfaces that provide certain generic logic, which can be used within the Party editor. Also, it might be possible that those interfaces automatically extend the UI screen with certain behavior like having a UI for managing comments to this Party.

Previously it was possible to use the [declarative-controllers](https://github.com/balvi/cuba-component-declarative-controllers) application component for this kind of behavior. There the implementation was done via Annotations, which has certain limitations. With CUBA 7 it is now possible to get a very similar kind of functionality from the framework directly.

I really like that interface based-approach, because it is more type save and adds various compile time checks to the table.

#### API switch Towards Java's Functional Style

Even before CUBA 7 APIs alternatives were introduces which more and more go into the style of leveraging Java 8 functional style. There are a lot of usages of the <code>java.util.function</code> package within the CUBA APIs. This seems to be a logical way. CUBA 7 continues this way and deprecates certain alternatives that previously existed.


### Data Containers replace Datasources

Pre CUBA 7 there was this concept of a Datasource, which is a connection layer between the UI components and the data fetching / storage on the middleware. As from what I have heard is that the implementation had some implications on the borders of what is possible to achieve with datasources. Examples of those limitations were:

* in a two nested composition case, it was required to create a datasource definition in XML for the second level in the root element
* it was not possible to have more than two nested levels of composition
* it was not possible to filter in nested datasources or use the Filter component

There might be a lot of additional limitations.

CUBA 7 reimplemented the concept of the datasources and called them Data containers. Fundamentally they solve the same problem, but as what I have seen, just in a better way. The above mentioned example limitations are solved with the new Implementation. Actually what previously was all under the umbrella of "datasource" is now split into two concepts: "data containers" and "data loaders".


You can find an example of the declarative definition of the data loaders in the [party-browse.xml](https://github.com/mariodavid/cuba-7-release-party/blob/master/com/rtcab/c7rp/web/party/party-browse.xml)

{% highlight java %}
<data readOnly="true">
    <collection id="partiesDc"
                class="com.rtcab.c7rp.entity.Party"
                view="party-view">
        <loader id="partiesDl">
            <query>
                <![CDATA[select e from c7rp_Party e]]>
            </query>
        </loader>
    </collection>
</data>
{% endhighlight %}


### Catch Up with Java

In general, it seems that the CUBA team used the major release in order to catch up with the latest Java changes. For some changes (like the <code>java.time</code> support) I have waited for ages, for others I'm really surprised.

#### Java 11 Support

The most important one is that CUBA catches up with new Java releases. This has probably become an additional burden for all Frameworks since the Java release cyclus changed from "up to 10 years" (;)) to a fixed six month period.

CUBA 7 now supports every Java version up until 11. It will be interesting how they will keep up with the new pace of Java. I hope (and assume) their plan is to also support new Java versions in minor releases, like having Java 12 support in CUBA 7.1. But this obviously heavily depends on how the underlying frameworks / libraries (Gradle, Eclipselink etc.) have support for latest Java versions.

#### Java 8 java.time Package

One thing I really appreciate in particular is the support for the non-crazy Date types that were introduced in Java 8, which are located in the <code>java.time</code> package. Dealing with dates (and times and timezones) in Java has been a mess for quite some time - for historical reasons. In Java 8 the situation improved dramatically, mainly because they mirrored what turned out to work great in the [Joda time library](https://www.joda.org/joda-time/).

CUBA was pretty late in the game with support for those datatypes. But with CUBA 7 and the corresponding Studio release, there is now full support for:

* <code>LocalDate</code>
* <code>LocalTime</code>
* <code>LocalDateTime</code>
* <code>OffsetTime</code>
* <code>OffsetDateTime</code>

## How the Ecosystem Evolves

<img src="/images/cuba-7-release-party/star.png" />

With a major version change of a framework, there always comes up the question of how fast the version is adopted. CUBA is no difference here. Oftentimes with major version updates heavy changes in the APIs go along with it.

The reason is that for the framework authors with changing a major version is one of the few possibilities to introduce new concepts and get rid of the old ones. They have the chance of putting everything in that new release, conceptually and API-wise, that has previously been a flaw.

The less frequent those possibilities occur for the framework authors, the more likely it is that the changes will be bigger.

### Application Component Adoption

When it comes to adopting the new version, it boils down not only to the framework but the complete ecosystem. In particular, this means the public open source application components have to support the new CUBA version as well.

This normally means that the practical adoption of new framework versions for applications is dependent on how fast the ecosystem can keep up.

Speaking of my existing open source application components: the way I will probably go is that I will publish versions of them, that will only have very basic compatibility with CUBA 7. This has the benefit that the users at least are not blocked by the app component in order to update to CUBA 7. Support for the new concepts of CUBA 7 in the application components will be published in another version.
If you want to help me out here with that, I would really like to see that. You can create issues or even PRs in the [Githhub repositories](https://github.com/mariodavid).

### Required Changes for Updating

The CUBA team did a pretty good job in not introducing heavy breaking changes. It is more that there are more options on the table. They used the <code>@Deprecation</code> hint to indicate that certain concepts / APIs are not future-proof anymore.

However, most of the stuff keeps working as it is. E.g. you can still use the <code>AbstractEditor</code> & <code>AbstractLookup</code> without interacting with the new Screen APIs at all. The same is true for the datasources.


## Let's Celebrate the New Release

<img src="/images/cuba-7-release-party/star.png" />

Overall, I really like the new release. It is packed with evolutionary ideas that bring CUBA into the Java world of 2019.

The CUBA team did a great job on not forcing the complete ecosystem to rewrite their apps from scratch. This is always one of the most important points, and a lot of other frameworks (teams) fail heavily in that regard.

The release has taken quite some time. CUBA 6.10 was released somewhere in the summer of 2018. But the development of CUBA 7 was already ongoing back then.

I would like to take the chance and congratulate the whole Haulmont team for the Release of CUBA 7. It most likely was a huge undertaking to get it shipped. I think you can really be proud of what you did!
