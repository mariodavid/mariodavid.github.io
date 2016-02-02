---
layout: post-dark
title: CUBA Security Subsystem Distilled
description: "..."
modified: 2015-11-02
tags: [cuba, security]
image:
  feature: cuba-security-subsystem-distilled/feature.jpg

---


Resuming my blog post series about an overview on the [CUBA Platform](https://www.cuba-platform.com) features, i will go through the possibilities of the so called "Security subsystem". This time with a particular theme from a well known TV series. I hope you enjoy it.

<!-- more -->

### From Albuquerque to Havanna
Walter, Jesse, Saul and Mike. After screwing up their last "Company" in Albuquerque, they started over again and try their Luck in Cuba. This time, with some serious trading business in place. The company is called *CUBa SeCurity Inc.* To get that going, they selected [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) as the basis for their ERP system. But unfortunately they had some requirements regarding security, that we will go through to see what the [CUBA Platform](https://www.cuba-platform.com) is capable of to accomplish their needs.

### The wide ocean called security

*Security* is a pretty bloated term. Words like Encryption, Integrity, ACL, TLS, Authentication, Cerfiticats, OAuth2, SQL Injection, AES, LDAP, PGP, Role based Access, Malware, Safety, 2-Factor authentication, Confidentiality and so on are subtopics, implementations or related topics to *security* and stuff that came out of the first requirements meetings with *CUBa SeCurity Inc.*

It can mean so much different things to different people, that it is necessary to talk about what i mean when talking about security. The subpart of security, that i want to cover is the typical business software scenario that include more or less something like this:

* User Authentication (often based on a LDAP directory like Microsoft Active Directory)
* User & User Groups based Acces Control to different parts of the software as well as Data (Authorization)
* Audit of user actions, who editied what data when and possibly why?


### What will not be covered in this article

This seems to be the *functional part* of the generally *non functional requirements*. Then we have the real *non non functional requirements*, like that the software should have a basic implementation againts application security attacks. A resilience against *SQL Injection* or *cross side scripting* and so on. 

This kind of security will not be part of this article firstly because of the sheer size of this problemspace and secondly because *CUBa SeCurity Inc.* is not interessted in these kinds of things, because they are planning to implement their ERP system as an offline application. 

Besides these application security topics, i won't cover the layers below the application layer as well, like [DNS Spoofing](http://www.windowsecurity.com/articles-tutorials/authentication_and_encryption/Understanding-Man-in-the-Middle-Attacks-ARP-Part2.html), [TCP SYN flooding](http://www.cisco.com/web/about/ac123/ac147/archived_issues/ipj_9-4/syn_flooding_attacks.html) or [ARP attacks](https://en.wikipedia.org/wiki/ARP_spoofing).

So after setting the scene, let's get back to the "functional non functional security requirements".

## CUBA's security subsystem

The possibilities that CUBA's security subsystem implements in this whole area are versitale. But mainly it solves different problems i described above as "functional non-functional security requirements". 

The concept behind it from a 10,000 feet overview is basically a *role-based access control* model. The baseline of the implementation has three different parts: *User*, *Role* and *Permission*. A User has different roles that grand or deny certain permissions. Permissions can be set on Entities, Entity attributes, UI elements like menus, buttons or screens. Additionally Users can be grouped in access groups to constrain data to a certain sub-part of the system.

On the administration layer, CUBA comes with UI that allow the administrators to manage users, their roles and permission. Alternatively user management can be achieved via an LDAP integration.

To get a better understanding of this pretty generic description, we will go through different scenarios that have been extracted from different consulting meetings with *CUBa SeCurity Inc.*


<img style="float:right" src="{{site.url}}/images/cuba-security-subsystem-distilled/cuba-login-dark.png">

### >> Only allow registered users access to the app

This one should be pretty straightforward. To achieve this, we've just to start up the app, because this feature is enabled by default. In the menu <code>Administration > Users</code> you get an extended CRUD mask for users. Besides the obvious ones like setting a password and personal information, you have the possibility to assign roles and *substituted users* to this user. 

User Substitution is a way to allow users to act on other users behalf. An example of this is a holiday replacement.

<img src="{{site.url}}/images/cuba-security-subsystem-distilled/create-new-user-dark.png">

Additionally, you have to select a *Group* where the user is placed in. We'll take a deeper Look into Groups a little later. For now, let's get to our next Requirement.



<img style="float:right; width:200px; padding: 10px; margin-right:-50px;" src="{{site.url}}/images/cuba-security-subsystem-distilled/jesse-pinkman.png">

### >> Let Jesse only see products and categories

This is Jesse. Jesse is responsible for the master data of the products and their categories. Since Jesse's manners aren't always the best, Mike, the manager, has decided to keep Jesse away from direct customer contact. Due to this, Jesse is not allowed to see and manage anything but the products and their categories.

To ensure this kind of requirement, we have to create a Role called <code>Master Data Manager</code>.


<figure class="center">
	<a href="{{ site.url }}/images/cuba-security-subsystem-distilled/master-data-manager-role.png"><img src="{{ site.url }}/images/cuba-security-subsystem-distilled/master-data-manager-role.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/cuba-security-subsystem-distilled/master-data-manager-role.png" title="Entity persmission of the Master Data Manager role">Entity persmission of the Master Data Manager role</a></figcaption>
</figure>