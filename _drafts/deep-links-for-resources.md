---
layout: post
title: Deep Links for Resources
description: "A little hidden feature in CUBA platform is deep linking to resources"
modified: 2016-02-18
tags: [cuba]
image:
  feature: deep-link/feature.jpg
  feature_source: https://pixabay.com/de/treffpunkt-treffen-pfeile-zetrum-755795/
  background: deep-link/feature-bottom4.jpg
---

Here's a thing i stumbled upon lately in the CUBA Platform docs, that seems not to be very common but nevertheless very valuable: Deep Links to resources.

<!-- more -->

When opening up a CUBA application in your browser, you'll end up with a static URL like <code>http://localhost:8080/app/#!</code>. It is *one* endpoint. In fact, it is *literally* and *end-point*, because it's kind of the end of the web. There is no possibility to get down further into the application via a link (at least, it is not shown in the URL of the browser). This is kind of sad. This is due to Vaadins architecture, but also very common nowadays when looking into the whole *single page application land*.

CUBA Platform currently does not support URL changes via *HTML5 History API* out of the box, meaning that it will change the shown URL when navigating through the application. 


<img style="float:right; padding: 10px; margin-right:-50px;" src="{{site.url}}/images/deep-link/labyrinth.jpg">

## CUBA Platform understands deep links

Although CUBA is not updating the address bar, it is in fact capabale of dealing with deep links. What i mean by that is, when you want to directly link to a certain customer entry, CUBA is able to handle that request and display e.g. the editor screen of the customer.

The feature is called [Screen Links](https://docs.cuba-platform.com/cuba/6.0/manual/en/html-single/manual.html#link_to_screen). With Screen Links you can address parts of the application, in particular a certain screen. An example of this would be:

<code>http://localhost:8080/app/open?screen=om$Customer.edit&item=om$Customer-*UUID*</code>

The parameters *screen* defines the screen to show and the parameter *item* qualifies the desired customer entity. The docs have some other examples like additional parameters that will be passed into the <code>init()</code> method or additional endpoints that can be registered instead of <code>open</code>.


## Generate links via the UI
Since hand-crafting these URLs seems not to be the best approach for the users, we can create a mechanism in the application which at least generates these URLs for us. Obviously it would be better if we could just copy the URL from the address bar, but since this is currently not possible, we can try to achieve something similar.

For this i created a little Service in the [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) app called [DeepLinkService](https://github.com/mariodavid/cuba-ordermanagement) that will build the corrent URL for us.

{% highlight groovy %}

@Service(DeepLinkService.NAME)
class DeepLinkServiceBean implements DeepLinkService {

    @Inject
    private GlobalConfig globalConfig

    @Override
    String generateDeepLinkForEntity(Instance entityInstance) {

        String webAppUrl = globalConfig.webAppUrl

        String className = entityInstance.getMetaClass().name
        String screen = "${className}.edit"
        String item = "${className}-${entityInstance.uuid}"
        String deepLink = "${webAppUrl}/open?screen=${screen}&item=${item}"

        return deepLink

    }
}

{% endhighlight %}

The <code>globalConfig</code> bean, as i learned [recently](https://www.cuba-platform.com/support/topic/how-to-get-the-server-url-with-context-path), holds the information about the current web app url. Next up, we invoke the service in the controller of the customer / [order browser](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/order/OrderBrowse.groovy):

{% highlight groovy %}

public class OrderBrowse extends AbstractLookup {

	//...

    @Inject
    private Datasource<Order> ordersDs;

    @Inject
    private DeepLinkService deepLinkService;

    //...

    void showDeepLink(Component component) {

        Order order = ordersDs.item
        String deepLink = deepLinkService.generateDeepLinkForEntity(order)
        String title = "Deep Link: " + order.getInstanceName();

        showMessageDialog(title, deepLink, Frame.MessageType.CONFIRMATION);

    }

}

{% endhighlight %}

and register a button on the [view](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/gui/src/com/company/ordermanagement/gui/order/order-browse.xml) to invoke the controller action:

{% highlight xml %}

<table id="ordersTable">
    <actions>
    	<!-- ... --->
        <action id="showDeepLink" invoke="showDeepLink" caption="msg://showDeepLink"/>
    </actions>

    <!-- ... --->

    <buttonsPanel id="buttonsPanel">
        <!-- ... --->
        <button action="ordersTable.showDeepLink"/>
    </buttonsPanel>
</table>


{% endhighlight %}

After setting this up, the application is capable of creating deep links for the user like you see in the following picture.

<figure class="center">
	<a href="{{ site.url }}/images/deep-link/deep-link-dialog.png"><img src="{{ site.url }}/images/deep-link/deep-link-dialog.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/deep-link/deep-link-dialog.png" title="Create a deep link via the UI for a selected entity instance">Create a deep link via the UI for a selected entity instance</a></figcaption>
</figure>

With this little generator in-place, we can easily create links that the user can copy and paste to reference a specific resource with when talking with a co-worker. As always, you'll find the full example in the [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) demo application.

In this sense: Happy linking!
