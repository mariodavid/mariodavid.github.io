---
layout: post
title: Security Tutorial
description: "..."
modified: 2015-11-02
tags: [cuba, security]
image:

---


Resuming my blog post series about an overview on the CUBA features, i will go through the possibilities of the so called "Security subsystem" of the CUBA Platform.

<!-- more -->

Starting with this fairly wide topic, i will lay out different possible requirements concerning security that might come up with the fictional [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) example app, i introduced in earilier blog posts.

### The wide ocean called security

*Security* is a pretty bloated term. Words like Encryption, Integrity, ACL, TLS, Authentication, Cerfiticats, OAuth2, SQL Injection, AES, LDAP, PGP, Role based Access, Malware, Safety, 2-Factor authentication, Confidentiality and so on are subtopics, implementations or related topics to *security*.
It can mean so much different things to different people, that it is necessary to talk about what i mean when talking about security.

The subpart of security, that i want to cover is the typical business software scenario that include more or less something like this:

* User Authentication (often based on a LDAP directory like Microsoft Active Directory)
* User & User Groups based Acces Control to different parts of the software as well as Data (Authorization)
* Audit of user actions, who editied what data when and possibly why?


### What will not be covered in this article

This seems to be the *functional part* of the generally *non functional requirements*. Then we have the real *non non functional requirements*, like that the software should have a basic implementation againts application security attacks. A resilience against *SQL Injection* or *cross side scripting* and so on. But this kind of security will not be part of this article because of the sheer size of this problemspace. Besides these application security topics, i won't cover the layers below the application layer as well, like [DNS Spoofing](http://www.windowsecurity.com/articles-tutorials/authentication_and_encryption/Understanding-Man-in-the-Middle-Attacks-ARP-Part2.html), [TCP SYN flooding](http://www.cisco.com/web/about/ac123/ac147/archived_issues/ipj_9-4/syn_flooding_attacks.html) or [ARP attacks](https://en.wikipedia.org/wiki/ARP_spoofing).

So after setting the scene, let's get back to the "functional non functional security requirements".

## CUBA's security subsystem

The possibilities that CUBA's security subsystem implements in this whole area are versitale. But mainly it solves different problems i described above as "functional non-functional security requirements". The concept behind it from a 10,000 feet overview is basically a *role-based access control* model. The baseline of the implementation has three different parts: *User*, *Role* and *Permission*. A User has different roles that grand or deny certain permissions. Permissions can be set on Entities, Entity attributes, UI elements like menus or buttons. Additionally Users can be grouped in access groups to constrain data to a certain sub-part of the system.

On the administration layer, CUBA comes with management ui's that allow the administrators to manage users, their roles and permission. Alternatively user management can be achieved via an LDAP integration.

To get a better understanding of this pretty generic description, we will go through different scenarios that the [cuba-ordermanagement](https://github.com/mariodavid/cuba-ordermanagement) app and it's users might have.


<img style="float:right" src="{{site.url}}/images/security-tutorial/cuba-login.png">

### Only allow registered users access to the app
This one should be pretty straightforward. To achieve this, we've just to start up the app, because this feature is enabled by default. In the menu <code>Administration > Users</code> you get an extended CRUD mask for users. Besides the obvious ones like setting a password and personal information, you have the possibility to assign roles and *substituted users* to this user. User Substitution is a way to allow users to act on other users behalf. An example of this is a holiday replacement.

<img src="{{site.url}}/images/security-tutorial/create-new-user.png">

Additionally, you have to select a *Group* where the user is placed in. We'll take a deeper Look into Groups a little later. For now, let's get to our next Requirement.

### Let Saul can only see products and categories

This is Saul. Saul is responsible for the master data of the products and their categories. Since Saul had some legal stuff going on, the bosses want to restrict the access of the software, so that Saul is only allowed to see the products and theier categories.


<img src="{{site.url}}/images/security-tutorial/saul-goodman.png">