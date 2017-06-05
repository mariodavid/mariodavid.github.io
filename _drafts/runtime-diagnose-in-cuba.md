---
layout: post
title: Runtime diagnose in CUBA
description: ""
modified: 2017-05-19
tags: [cuba, debugging, runtime, diagnose]
image:
  feature: runtime-diagnose-in-cuba/feature.jpg
---

Doing diagnosis on running applications is oftentimes a very tedious thing. <code>grep</code> through log files is mostly the preferred way of doing it. Since this is not the most efficient way, there are other options. One of those is to execute diagnosis code at runtime. Let's look how this can be done in CUBA.


<!-- more -->

## Investigating on a running system is hard

When you try to resolve an issue in a running system it can be hard sometimes to effectivly getting the information you need to understand the problem. First of all you need to understand the not always so clear bug descriptions you might have gotten from your customers. Sometimes it is reproducible in your test system, other times it is not.

In these cases you have to somehow get further information from the running system. In case you run you software on your own you have to get access to the corresponding log files or application to see the error in real life or at least see the impact it has on the software. When doing boxed software the situation is even worse, because the barrier to get to the required information just is higher and the process is more tedious.

If you already live in this fancy microservices world where you have full control over your running software. In this case you probably have automated environments in place that will raise the visibility of insights into the system.

If not you are in this weired place where trying to identify the actual problem feels like trying hit a target with a bullet in a totally dark room. Probably the majority of software developers have more or less been in this situation from time to time.

If you can identify with this situation or with the one below in the gif, you might be interested in seeing another tool in the runtime-debuggers' toolset:

<figure class="center">
<img src="{{ site.url }}/images/runtime-diagnose-in-cuba/fixing-bug-in-production.gif" alt="">
	<figcaption>fixing a bug in production is like...</figcaption>
</figure>


## Runtime diagnoses for CUBA to the recuse

When i was working in [grails](https://grails.org/) land, there was (and still is) a really cool plugin called the [Grails console](http://plugins.grails.org/plugin/sheehan/console). What that is basically, is just like an interactive [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) for a running application. It has a web UI, so it is possible to write code in this UI and execute it on the fly. This can be used for totally different purposes:

* manually adjust the Log4j settings
* insert data into the running system
* executing business logic you wrote in services e.g.
* shutdown the tomcat server

While some of these features are pretty cool, others are really scary (from a security point of view). So this plugin has to be used with caution and properly configured from a security point of view. Or like Spider-Man likes to phrase it:

> With Great Power Comes Great Responsibility

### The grails console for CUBA

I recently created a application component that took the ideas from the grails console and transfered it to CUBA island. Actually this was just the starting point, because i took the chance to look a little bit deeper in the topic of runtime dianose and came up with three different areas of features that are useful for doing analysis on problems in a running system.

### 1. Groovy console for ad-hoc debugging & diagnosis

<figure class="center">
	<img src="https://github.com/mariodavid/cuba-component-runtime-diagnose/raw/master/img/groovy-console-screenshot.png" alt="">
	<figcaption>Groovy console in CUBA</figcaption>
</figure>

The groovy console allows you to interactivly inspect the running application. You enter a groovy script and execute it in an ad-hoc fashion. This obvisouly requires access to the running system.

The code get's executed in the core module of the application. Therefore all CUBA beans are at your fingertips. Here are some examples of what you can do with it:

* Insert / update data through CUBAs DataManager
* execute middleware beans to get dianose information or trigger certain business logic
* correct data that have been introduced through a bug in your application
* check configurations
* ...

#### Execution results
There are different results of a groovy script (displayed in the different tabs). The actual result of the script (meaning the return value of the last statement) is displayed in the first tab. The stacktrace tab displayes the stacktrace of a possible exception that occurs during script execution. The tab executed script shows the actual executed script. For more information take a look at the [docs](https://github.com/mariodavid/cuba-component-runtime-diagnose/#execution-results)


These results can be downloaded as a zip file that will represent the tabs with the corresponding information.

### 2. SQL console for ad-hoc debugging & diagnosis

<figure class="center">
	<img src="https://github.com/mariodavid/cuba-component-runtime-diagnose/raw/master/img/sql-console-screenshot.png" alt="">
	<figcaption>SQL console in CUBA</figcaption>
</figure>

The SQL console allows you to interactivly interact with the database using raw SQL statements. You enter a SQL script and execute it in an ad-hoc fashion.

<div class="information">NOTE: for normal data diagnosis the <a href="https://doc.cuba-platform.com/manual-6.4/entity_inspector.html">Entity inspector</a> is oftentimes more user friendly, even for debugging purposes. Usage of the SQL-console is to be preferable to the entity inspector if you want to access data across tables using joins e.g.</div>

Results of a SQL statement are displayed in a table in the result tab. The result can be downloaded using the Excel button in the Results tab.

The execution if different SQL statements can be restricted. Normally only SELECT statements are allowed. For more information see the corresponding [docs](https://github.com/mariodavid/cuba-component-runtime-diagnose/#security-of-the-sql-console).


### 3. Diagnose wizard for non-interactive use cases

<figure class="center">
	<img src="https://github.com/mariodavid/cuba-component-runtime-diagnose/raw/master/img/diagnose-wizard-screenshot.png" alt="">
	<figcaption>SQL console in CUBA</figcaption>
</figure>

The last part is the diagnose wizard. This option is relevant if you as a developer or customer support person don't have direct access to the running application, because of security reasons or it is boxed software that is running out of your control. You could send your counterpart on customer side a text file which the the user should execute in the Groovy / SQL console, but this process is fairly insecure as well as error prone.

In these cases you can send the person a zip file (as a black box) and tell them to upload this file in the diagnose wizard. The person will be guided through the different steps, executed the scripts and gets back the execution results (as a zip file) that should be handed back to you.

#### Tighten the bug-report feedback loop
Let's look at the alternative we have in these scenarios. When the customer calls you that something strage happened in the application, you'll ask them a handful of questions in order to get a rough idea what category of error this is, if it is a bug at all or more a misunderstanding of the application etc.

After that you might ask for the log files to take a closer look. Depending on how fast this information can be provided, you are waiting for some time. After you got the log files and you already prayed to the Debugging-God that you have good logging messages in places as well as logging configured properly. If you are lucky you see some null pointer exception with a stacktrace that you can start your investigation form. Other times you don't see much. So you get back to your customer, ask them to adjust the logging settings and try to reproduce the error. After another round trip you might have more luck. Every step involves some email or phone calls with a lot of context switching on both sides.

I think i don't have to describe the process any further. The bottom line is, this is a situation that occurs sometimes, but nobody really wants to be in this error prone and tedious situation.


To tighten this feedback loop a little you can use the workflow by the diagnose wizard since it gives you more direct feedback. The best situation in this regard would be the ad hoc groovy / sql console, because the developer feedback is immediate. But as this is not always possible so the Diagnose wizard allows you to find a middle course in this scenario.


## Summary
