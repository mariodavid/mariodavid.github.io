---
layout: post
title: Groovify CUBA - Integrate Groovy with CUBA
description: "In this blog post i’d like to show you how to change the normal Java CUBA App to a Groovy CUBA App to increase the developer productivity even further"
modified: 2015-1-20
tags: [cuba, groovy, java]
image:
  feature: groovify-cuba-app/feature.jpg
  feature_source: https://pixabay.com/de/neujahr-silvester-silvester-2015-1090770/
---

In this second blog post about groovify your [CUBA](https://www.cuba-platform.com/) app, after introducing Groovy it's all about integrating Groovy with CUBA in different shapes.


<!-- more -->

## Give it to my CUBA project

To activate Groovy in the project is pretty [straight forward](https://www.cuba-platform.com/support/topic/using-groovy-to-implement-a-service-interface) when you know a little about Gradle. 

In the <code>build.gradle</code> you have to add the groovy plugin in the section of configure <code>coreModule</code>. Then you have to tell groovy where to find the <code>src</code> and the <code>test</code> directories.
{% highlight groovy %}
configure([globalModule, coreModule, guiModule, webModule]) {
    
    // ...

    apply(plugin: 'groovy')
    sourceSets {
        main {
            groovy {
                srcDirs = ['src']
            }
        }
        test {
            groovy {
                srcDirs = ['test']
            }
        }
    }
{% endhighlight %}

With this setting inplace, you are able to create a groovy file inside of your IDE in the core module.
