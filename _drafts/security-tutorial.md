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

The possibilities that CUBA...





