---
layout: post-dark
title: CUBA Security Subsystem Distilled
description: "..."
modified: 2015-11-02
tags: [cuba, security]
image:
  feature: cuba-security-subsystem-distilled/feature-2.jpg

---


Resuming my blog post series about an overview on the [CUBA Platform](https://www.cuba-platform.com) features, i will go through the possibilities of the so called "Security subsystem". This time with a particular theme from a well known TV series. I hope you enjoy it.

<!-- more -->




<img style="float: left; margin-left:-280px; margin-top:-450px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-7.jpg">



### From Albuquerque to Havanna


<img style="float: right; padding: 10px; width: 300px; margin-right:-10px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/landkarte-dark.jpg">

The story begins with Walter, Jesse, Saul and Mike. They lived in Albuquerque quite happily for sometime. On day, they screw up their "Company" in here, and so they decided to started over again and try their Luck in Cuba. This time they want to make a better job with some serious trading business in place. 

The company they started is called *CUBa SeCurity Inc.*, which is selling security goods and widgets all around the US and south america.



To get that going, they selected [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) as the basis for their ERP system. In addition what the software already provides, they had some requirements regarding security that we will go through to see what the [CUBA Platform](https://www.cuba-platform.com) is capable of to accomplish their needs.

<img style="float: right; margin-right:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-6.jpg">

### The wide ocean called security

*Security* is a pretty bloated term. Words like encryption, integrity, ACL, TLS, authentication, cerfiticats, OAuth2, SQL Injection, AES, LDAP, PGP, role based access, malware, safety, 2-Factor authentication, Confidentiality and so on are subtopics, implementations or related topics to *security* and stuff that came out of the first requirements meetings with *CUBa SeCurity Inc.*

<figure class="center">
	<img src="{{site.url}}/images/cuba-security-subsystem-distilled/meetings-2.jpg" width="300">
</figure>


The meetings revealed that everyone has different things in mind when talking about *Security*. Due to this it is necessary to talk about what i mean when writing about security in the article. The subpart of security, that i want to cover is the typical business software scenario that include more or less something like this:

* User Authentication (often based on a LDAP directory like Microsoft Active Directory)
* User & User Groups based Acces Control to different parts of the software as well as Data (Authorization)
* Audit of user actions, who editied what data when and possibly why?


### What will not be covered in this article

<img style="float: left; margin-left:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-8.jpg">

This seems to be the *functional part* of the generally *non-functional requirement: Security*. Then we have the real *non non-functional requirements*. The software should have implementations to protected itself against application security attacks, like a resilience against *SQL Injection* or *cross side scripting* and so on. 

This kind of security will not be part of this article firstly because of the sheer size of this problemspace and secondly because *CUBa SeCurity Inc.* is not interessted in these kinds of things. They tols use, that they are planning to implement their ERP system as an offline application. 

Besides these application security topics, i won't cover the layers below the application layer, like [DNS Spoofing](http://www.windowsecurity.com/articles-tutorials/authentication_and_encryption/Understanding-Man-in-the-Middle-Attacks-ARP-Part2.html), [TCP SYN flooding](http://www.cisco.com/web/about/ac123/ac147/archived_issues/ipj_9-4/syn_flooding_attacks.html) or [ARP attacks](https://en.wikipedia.org/wiki/ARP_spoofing) either. Although it's a very interessting topic, generally it is not something a "normal business software developer" has any impact on. Oftentimes its outside of the scope of the application developer.

So after setting the scene, let's get back to the "functional non-functional security requirements".

<h2>CU<img src="{{site.url}}/images/cuba-security-subsystem-distilled/Ba.png" style="width: 2em;" alt="BA">'s security subsystem</h2>

The possibilities that CUBA's security subsystem implements in this whole area are versitale. But mainly it solves different problems i described above as "functional non-functional security requirements". 

<img style="float: right; margin-right:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-5.jpg">

The concept behind it from a 10,000 feet overview is basically a *role-based access control* model. The baseline of the implementation has three different parts: *User*, *Role* and *Permission*. A User has different roles that grand or deny certain permissions. Permissions can be set on entities, entity attributes, UI elements like menus, buttons or screens. Additionally Users can be grouped in access groups to constrain data to a certain sub-part of the system.

On the administration side of things, CUBA comes with UI that allows the administrator to manage users, their roles and permission. Alternatively user management can be achieved via an LDAP integration.

To get a better understanding of this pretty generic description, we will go through different scenarios that have been extracted from consulting meetings with *CUBa SeCurity Inc.*

<figure class="center">
	<img src="{{site.url}}/images/cuba-security-subsystem-distilled/requirements-meeting.jpg" width="300">
</figure>



### >> Only allow those four buddies access to the app

<img style="float: right; padding: 10px; width: 150px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/cuba-login-dark.png">


This one should be pretty straightforward. To achieve it, we've just to start up the app because this feature is enabled by default. 

In the menu <code>Administration > Users</code> you get an extended CRUD mask for users. Besides the obvious ones like setting a password and personal information, you have the possibility to assign roles and *substituted users* to this user. Below, you'll see the management UI for creating a user with the described attributes.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-security-subsystem-distilled/create-new-user.png"><img src="{{ site.url }}/images/cuba-security-subsystem-distilled/create-new-user-dark.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-security-subsystem-distilled/create-new-user.png" title="Creation of a new User">Creation of a new User</a></figcaption>
</figure>


Additionally, you have to select a *Group* where the user is placed in. We'll take a deeper Look into Groups a little later. For now, let's get to our next Requirement.



<img style="float: left; margin-left:-177px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-2.jpg">



<img style="float:right; width:200px; padding: 10px; margin-right:-50px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/jesse-pinkman.jpg">

### >> Let Jesse only see products and categories

This is Jesse. Jesse is responsible for the master data of the products and their categories. Since Jesse's manners aren't always the best, Mike, the manager, has decided to avoid direct customer contact for him. Due to this, he isn't allowed to see or manage anything but the products and their categories.

To ensure this kind of requirement, we have to create a Role (<code>Administration > Roles</code>) called *Master Data Manager*.

In the entities tab, you are able to search for certain entites that you want to give permissions on. Just uncheck the "Assigned only" flag and click "Apply", which will show all entities. When you select a certain entity, you can grand or deny permissions for this selection. For attribute based security it works exactly the same in the "Attributes" tab. 

Since the *Master Data Manager* role has only rights on the two entities *Product* and *Product Category* we'll only enable them for management. On the screen tab, we'll only allow the screen of these two to be shown.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-security-subsystem-distilled/role-master-data-manager.png"><img src="{{ site.url }}/images/cuba-security-subsystem-distilled/role-master-data-manager-dark.jpg" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-security-subsystem-distilled/role-master-data-manager.png" title="Entity persmission of the Master Data Manager role">Entity persmission of the Master Data Manager role</a></figcaption>
</figure>


<img style="float: left; margin-left:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel.jpg">

To assign users to this new role, you can either edit the user and add the role, or you can use the "Assign to Users" Button on the Roles browser. After doing that Jesse will only get the corresponding views and rights to edit *Products* and *Product Categories*.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-security-subsystem-distilled/jesse-application-view.png"><img src="{{ site.url }}/images/cuba-security-subsystem-distilled/jesse-application-view-dark.jpg" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-security-subsystem-distilled/jesse-application-view.png" title="Jesse can only see Products and Categories">Jesse can only see Products and Categories</a></figcaption>
</figure>


<h3> » Walter is responsible for the Northeast of the <img src="{{site.url}}/images/cuba-security-subsystem-distilled/U.png" style="width: 2em;" alt="U"> S</h3>


<img style="float:right; margin-right: -30px; width:150px; padding: 10px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/walter-white.jpg">

From earlier connections to New Hampshire, Walter - the sales guy, is selected to be responsible for the Northeast region of the US market. 

Due to this, the scope of the data that Walter needs to act upon is basically the customers that are based here. 
To achieve this kind of requirement, we'll have to take a look at the already mentioned *User Groups* and the constraints that can created for them. 

After creating a User account for Walt and a Role called *Sales* which has full permissions on Customers, Orders and read permissions on the Products and Categories, we will create a Access Group (<code>Administration > Access Groups</code>) called "Northeast". We assign Walter to this group. 


<img style="float: right; margin-right:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-3.jpg">


Next, we will create two contraints that will only allow access to data of the customers and the orders, where the customers *state* attribute is in the northeast of the US. 


<figure class="center">
	<a href="{{ site.url }}/images/cuba-security-subsystem-distilled/access-group-northeast.png"><img src="{{ site.url }}/images/cuba-security-subsystem-distilled/access-group-northeast-dark.jpg" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-security-subsystem-distilled/access-group-northeast.png" title="Constraints for Access Group 'Northeast'">Constraints for Access Group "Northeast"</a></figcaption>
</figure>

As shown in the image above, a constraint can have a *Where* condition, which will reduce the data corresponding to its contraints. For the Customer entity it is something like: <code>{E}.state = 'NH' or {E}.state = 'MA'</code> where <code>{E}</code> always references the selected entity (*Customer* in this case).

With this setting in place, Walt will only be able to see the orders and customers in the selected area.

<h3> » S<img src="{{site.url}}/images/cuba-security-subsystem-distilled/AU.png" style="width: 1.5em;" alt="AU">l can take on business for Walter</h3>

Walter had some problems with its physical condition in the last years. This is why Mike wants Saul, the Head of Sales to take on business for Walter in certain situations. Normally, Saul sales region is the Midwest of the United States, especially Nebraska because *CUBa SeCurity Inc.* has a branch office in Omaha. 


<img style="float: left; margin-left:-280px" src="{{site.url}}/images/cuba-security-subsystem-distilled/nebel-7.jpg">


<figure class="center">
<img style="width: 400px; padding: 10px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/saul-and-walter.jpg">
</figure>

Instead of giving Saul rights on data for the midwest as well as northeast, we will use *User Substitudion* to achieve this goal.
A User for Saul is created, Sales Role will be assigned and an addtional Access Group under "US" called "Midwest" next to the "Northeast". 

<img style="float: right; width: 300px; padding: 10px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/saul-user-substitution-walter.jpg">

Next up, we'll go into the User details of Saul and create a *User Substition*. The substituted User is Walter in this case. Optionally a substitution can be granted for a given time period (not necessary in this case).

After doing so, Saul can login and see a little select box in the top right corner of the menu where he can change the user to Walt. 

This user switch will lead to the following situation: Before, Saul is able to see all orders and customers from the midwest. After changing the user to Walt, Saul can see the ones in the northeast and actually acting as Walt with all the permissions that are granted to him.

The described example with all the required data can be found at the [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) example app.