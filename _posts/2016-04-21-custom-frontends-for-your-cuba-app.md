---
layout: post
title: Custom front-ends for your CUBA app
description: In this blog post i want to show you how you can create really custom front-ends for your CUBA based application. If you want to have customer facing UI as part of your software, here's how you can create them.
modified: 2016-04-21
tags: [cuba, portal, bootstrap, Spring MVC]
image:
  feature: custom-ui-portal/feature.jpg
  feature_source: https://pixabay.com/en/imac-ipad-computer-calendar-year-965325/
---

In this blog post i want to show you how you can create really custom front-ends for your CUBA based application. If you want to have customer facing UI as part of your software, here's how you can create them.

<!-- more -->

When you look through the CUBA [docs](https://doc.cuba-platform.com/manual-6.1), you'll find a part which is often mentioned but not described in great detail: [the portal module](https://doc.cuba-platform.com/manual-6.1/portal.html). This is why i want to cover the topic in this blog post.

## What is the portal module?
The elevator pitch for the portal module is that you can create custom user interfaces which are not directly part of the CUBA application. Instead it can be deployed independently and communicate with the [platform middleware](https://doc.cuba-platform.com/manual-6.1/app_tiers.html). 

Further it does not use the generic user interface of CUBA. Instead the platform gives you the possibility to go down one abstraction level. The portal module allows you to create plain Spring MVC based applications. With this you have total control over the "plain" HTTP based communication. With Spring MVC you could either create HTTP based APIs (if you don't want to use the [generic REST API](https://doc.cuba-platform.com/manual-6.1/rest_api.html)) or HTML based user interfaces. For the latter one [FreeMaker](http://freemarker.org/) is included into the portal module as the template engine of choice.

## cuba-ordermanagement's public product catalog
As a basis for this topic we'll use [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement).
The addition that we want to achieve is the following:

* list all available products
* show details of a product in a detail view
* let the user login on the front-end 

This will be achieved by creating a [Boostrap](http://getbootstrap.com) based user interface. Here is the result of the UI. This is to give you a feeling about what is possible and where we want to go in the next 5 minutes:

<figure class="center"><img src="{{ site.url }}/images/custom-ui-portal/portal-product-detail.gif" alt="All products listed in a custom UI with detail information">
    <figcaption>All products listed in a custom UI with detail information</figcaption>
</figure>


### Activate the portal module

First of all, the portal module has to be created in the project. To do this, there is a link in studio within the *Project Properties* section called *Create portal module*. Studio will create the module and a little bit of sample code so that you have a solid starting point. In your project files you'll notice the new module under <code>modules/portal</code>. Looking at the generated code, we see that the <code><a href="https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/portal/src/com/company/ordermanagement/portal/controllers/PortalController.java">PortalController</a></code>  that handles the root URL <code>/app-portal/</code>. When you have a look at it it will show something like this:
{% highlight java %}

@Controller
public class PortalController {

    @Inject
    protected DataService dataService;

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public String index(Model model) {
        if (PortalSessionProvider.getUserSession().isAuthenticated()) {
            LoadContext l = new LoadContext(User.class);
            l.setQueryString("select u from sec$User u");
            model.addAttribute("users", dataService.loadList(l));
        }
        return "index";
    }
}

{% endhighlight %}

What we see here is a mix of CUBA logic together with Spring MVC glue code. First, there is the <code>RequestMapping</code> Annotation which indicates the above mentioned route to the root URL.

After this, there is an example on how to get information about the login status of the current user. In case of a authenticated user, all users from the database are added to the model (which will be passed to the view layer). 

After this brief overview, let's start with our first requirement from above: *List all products in the view*.

### Create a public product catalog

To create product catalog that is publicly accessible, we need to slightly change the example. As always, you'll find the running example on [Github](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/portal/src/com/company/ordermanagement/portal/controllers/PortalController.java).

{% highlight java %}

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public String index(Model model) {

        LoadContext l = LoadContext.create(Product.class).setView("product-view");
        l.setQueryString("select p from om$Product p");
        model.addAttribute("products", dataService.loadList(l));

        if (!PortalSessionProvider.getUserSession().isAuthenticated()) {
            final LoginUserCommand loginUserCommand = new LoginUserCommand();
            model.addAttribute(loginUserCommand);
        }

        return "index";
    }

{% endhighlight %}

All products will be fetched from the database (like above). In this case we want the <code>product-view</code> View to be taken for retrieval. This is not directly neccessary to get our job done, but in case we want to show the product category e.g. it might be useful.

In case the user in not logged in, the <code>LoginUserCommand</code> will be additionally passed to the view.

To work with the passed view information we have to take a look at the <code><a href="https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/portal/web/WEB-INF/templates/index.ftl">index.ftl</a></code> FreeMaker template:

{% highlight html %}

<h2>All Products</h2>

<div class="list-group">
    <#list products as product>
        <a class="list-group-item product">${product.name}</a>
    </#list>
</div>

{% endhighlight %}


FreeMaker is a template language that allows you to combine HTML with different elements of normal programming languages like loops, variables, conditional statements and so on. The expressions of the template are parsed and executed. The output of this is a plain HTML structure.

What happens here is basically iterating over the list of <code>products</code> (passed from the controller) and for each <code>product</code> a link is created with the name of the product. This is packed into a bootstrap <code>list-group</code>, so that we have a fairly lovely UI.

<figure class="center half"><img src="{{ site.url }}/images/custom-ui-portal/portal-product-list.png" alt="">
    <figcaption>All products listed in the view</figcaption>
</figure>

### Show details of a product

The next step is to improve the UI so that the user can show details of products. This step is more around Bootstrap and a little bit of JQuery.

There are many possible solutions to achieve the dynamism from the overview image above. It would be possible to load the data asynchronously via Ajax and update the DOM. I choosed a solution where all the data is already on the client at inital delivery of the HTML file. With HTML5 we have the possibility to add any metadata to HTML tags through <code>data-*</code> [attributes](http://www.w3schools.com/tags/att_global_data.asp). 

We'll extend the existing view to store the data inside the product list like this:


{% highlight html %}

<!-- product list -->
<div class="col-md-4">
    <h2>All Products</h2>

    <div class="list-group">
        <#list products as product>
            <#if product.description??>
                <a class="list-group-item product"
                        data-title="${product.name}"
                        data-content="${product.description}">
                ${product.name}
                </a>
            <#else>
                <a class="list-group-item product"
                   data-title="${product.name}"
                    data-content="">
                    ${product.name}
                </a>
            </#if>
        </#list>
    </div>
</div>

<!-- product details -->
<div class="col-sm-8" id="product-detail-column"  style="display: none">
    <h2>Productinformation</h2>
    <div class="panel panel-default" id="product-details">
        <div class="panel-heading"></div>
        <div class="panel-body"></div>
    </div>
</div>

{% endhighlight %}

Product title and description are stored in the <code>data-*</code> attributes. There is one gotcha which i stumbled upon: In case there is no <code>description</code> value set for a product, the FreeMaker template will screw up with an exception. To solve this hurdle, there is an explicit null check in-place.

I personally find that a little daunting because null could easily translated to an emtpy string. But as i'm not at all a FreeMaker expert i assume there are resons behind this behavior.

The second part of the markup (starting with <code>col-sm-8</code>) is just an empty template for the product details. After getting the markup inplace let's have a look at the required Javascript that brings the content to life.


{% highlight javascript %}

// ...

$(".product").click(function() {
    var productElement = this;
    setProductActiveState(productElement);
    $("#product-detail-column").fadeOut("fast", function() {
        var product = readProductDetailsFromElement(productElement);
        changeProductDetails(product);
        $("#product-detail-column").fadeIn("fast");
    });
});

//...

{% endhighlight %}

All elements with the class <code>product</code> (which are the product links with the data attributes from the product list) are selected and a click event is registered.

In case this event occurs, different things have to happen. Firstly the clicked element has to be marked as active (<code>setProductActiveState(productElement)</code>). Next, the product detail column will fade out. 

After finishing this animation, the product details will be read from the data-* attributes and stored into the variable <code>product</code>. This will be passed to <code>changeProductDetails</code> to update the hidden product detail column. The last this is fade the column back in.

The complete code of the view is available [here](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/portal/web/WEB-INF/templates/index.ftl).

### Activate front-end login

The last thing that i want to show in this example portal module is the ability to login on this front-end. The generated code already contains all ne neccessary bits go do a full login. So in this case, we will just shuffle it a little bit around so that i fits our UI needs a little better.

First of all, as we want to login directly from the main page, there is no LoginController needed to give us the login form. Instead, we just copy the <code>LoginUserCommand</code> in case of a non existing authentication into the <code>PortalController</code> (you already saw that code above).

In the view i created a bootstrap form in the navbar template, which uses the <code>LoginUserCommand</code> like this (<a href="https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/portal/web/WEB-INF/templates/common/navbar.ftl">navbar.ftl</a>):



{% highlight html %}

<div id="navbar" class="navbar-collapse collapse">
<#if userSession?? && userSession.authenticated>
    <ul class="nav navbar-nav navbar-right">
        <li class="active"><a href="#">${userSession.user.login}</a></li>
        <li><a href="logout">Logout</a></li>
    </ul>

<#else>
    <form 
    	class="navbar-form navbar-right" 
    	method="POST" 
    	action="<@spring.url "/login"/>">
        <div class="form-group">
            <@spring.formInput
                path="loginUserCommand.login"
                fieldType="text"
                attributes="class='form-control' placeholder='Username'"/>
        </div>
        <div class="form-group">
            <@spring.formInput
                path="loginUserCommand.password"
                fieldType="password"
                attributes="class='form-control' placeholder='Password'"/>
        </div>
        <button type="submit" class="btn btn-success">Sign in</button>
    </form>
</#if>

{% endhighlight %}

The navigation menu shows a login to the unauthenticated users. This form does a <code>POST</code> to <code>/login</code> with the fields <code>login</code> and <code>password</code>. The actual authentication is handled by *Spring Security*. Luckily the generated code already includes an [authentication provider](https://github.com/mariodavid/cuba-ordermanagement/blob/master/modules/portal/src/com/company/ordermanagement/portal/auth/PortalAuthenticationProvider.java) that handles the authentication agains the middle tier of the CUBA application.

Next we could probably do different stuff for people that are logged in like for example change the product description or display internal information about the product. But i leave it at that for now. It is up to you to come up with something of value for the authenticated users.

## The resonable borders of this approach

Although the sky is the limit here, when i starting fiddling around doing this custom UI and thinking about possible use cases for the authenticated users, i had to take a step back and revisited the purpose of this custom UI. 

Most things i had in mind, like showing all orders for this product, was just stuff i would normally develop in the actual CUBA app. So this portal module just makes sense if you want to reach out to people that are mostly not users of the system or if they have particular requirements. 

One thing that we got out of this approach is the possibility to *easily* get all product information on their mobile devices because bootstrap made it mobile ready.

Another possibility is to create a JSON API which can be consumed by JS frameworks like AngularJS or by native apps. I haven't touched this idea for now, but that might be an interesting topic for a future blog post.

I hope you enjoyed this little overview of what you can do with the portal module. If you have any feedback i would love to hear from you via [Twitter](https://twitter.com/mariomddavid), [Mail](mailto:mario@die-davids.de) or leave a comment below.