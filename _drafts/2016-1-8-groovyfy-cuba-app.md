---
layout: post
title: Groovify your CUBA App
description: "In this blog post i’d like to show you how to change the normal Java CUBA App to a Groovy CUBA App to increase the developer productivity even further"
modified: 2015-1-8
tags: [cuba, groovy, java]
image:
  feature: groovify-cuba-app/feature.jpg
  feature_source: https://pixabay.com/de/neujahr-silvester-silvester-2015-1090770/
---

Basically developing an application in [CUBA](https://www.cuba-platform.com/) is mostly about productivity. It has other advantages, but productivity is probably one of the most important reasons. To ramp up productivity on all stages of CUBA app development, we can invite Groovy to the party.


<!-- more -->

In this blog post i will give you a little guideline how to change the language of a CUBA App from Java to Groovy.
Why? Because in my opinion this productivity advantage of CUBA falls apart when going away from generating the UI or creating the domain model to the part of programming where you actually want to implement business logic. In this scenario you are back at a good old POJO model either in your Controller logic or in the services that are just Spring beans. This is because after striping everything down to efficiency except the raw business logic and probably some database access or ui related logic is the last part that remains.

Coming from a Groovy background and haven't developed in Java for quite some time, it feels a little bit clumsy to go back. This is why i think it's probably worth thinking about raising the productivity grain just a little bit by getting the different benefits of the Groovy language onto our CUBA application.

## The elevator pitch for Groovy


{% highlight java %}

public class Person {
	
	private String firstName
	privat String lastName

	private int age

	public String getFirstName() {
		return firstName;
	}
	public String getLastName() {
		return lastName;
	}
	public int getAge() {
		return age;
	}
}

{% endhighlight %}