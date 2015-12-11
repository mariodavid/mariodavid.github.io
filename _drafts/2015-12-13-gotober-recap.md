---
layout: post
title: GOTO Berlin 2015 recap 
description: "In this blog post i’d like to show you how to change the normal Java CUBA App to a Groovy CUBA App to increase the developer productivity even further"
modified: 2015-12-15
tags: [GOTOber, conference]
---

I want to make a little stopover on my road to CUBA in Berlin. On the beginning of december i had the chance to attend to [GOTO Berlin 2015](http://gotocon.com/berlin-2015) and i just wanted to recap my thoughts on the conference. So if you're interessted because you were not able to attend and fly over to berlin, here's a summary.

The conference took place from 2nd of december until 4th of december, where the first day was a workshop day i didn't attend. On Thursday morning the conf started for me with a keynote and a free and super-delicios cappuccino from [The Barn](http://barn.bigcartel.com/), which had a stand in the foyer. 

<img style="float:right; padding: 10px;" src="{{site.url}}/images/2015-12-15-gotober-recap/rocket.png">

### To the moon - Russ Olsen 

The keynote was called "To the moon" and [Russ Olsen](https://twitter.com/russolsen) was the guy presenting it. Unfortunately i watched a previous held talk at GOTO Aarhus 2014 on [Youtube](https://www.youtube.com/watch?v=Z0MbpkYPgM8) a while ago, but luckily i couldn't remeber all the bits and pieces of it. It wasn't a real talk as you might expect it from a tech conference, but more of a story about the moon landing and the people behind the story. BTW: I absolutely encourage you to look at the talk on Youtube!

As Russ is a amazing story teller, this story absolutly absorbed me. Since the moon landing generation is a little before my time, i wasn't really aware of the details of the landing and due to this it was really amazing to get these insights.

With his analogy drawings between rocket science and software development he took all of us back down to earth and the nitty gritty problems that the John Doe developer has as a sotware developer. But it was the right mindset to get onto the next talks about different technologies, software practices and experience reports.

### Thursday: The first conference day
The conference was split into six parallel [tracks](http://gotocon.com/berlin-2015/schedule/thursday.jsp). I was personally interessted in Docker, Cloud and so on, i started off with a talk called "[Build, Ship and Run the Docker Way](http://gotocon.com/dl/goto-berlin-2015/slides/MaximeHeckel_BuildShipAndRunTheDockerWay.pdf)" from [Maxime Heckel](https://twitter.com/MaximeHeckel). He showed how [Tumtum](https://www.tutum.co), an orchestration enginge with a slick UI and plenty of features that let you step up from the docker command line to something more high-level. It is meant to be used for running docker in production.
In my opinion the title promised a slightly different topic but it was ok for me, since the ecosystem around docker is at least equally interessting as the tool itself.

Next up, i went to the track "Smart things for a connected world" with a talk from [Michael Fait](https://twitter.com/MFait) and [Martin Comfort](http://gotocon.com/berlin-2015/speaker/Martin+Comfort) called "[50000 Lines Of Code to Brew a Coffee](http://gotocon.com/berlin-2015/presentation/50000%20Lines%20Of%20Code%20to%20Brew%20a%20Coffee)". They talked about a project they worked on at Thoughtorks, where they had to create the "smart" part of a smart coffee machine. As it was a project with a hardware development, it seems to be a whole different story in terms of cycle times and testing e.g. They came up with different interessting approaches to tackle these problems when you compare it to a normal web development projects and were thus able to get their common practices like CI, TDD in place.


<img style="float:left; padding: 10px;" src="{{site.url}}/images/2015-12-15-gotober-recap/spotify.png">

After this we had a pretty amazing lunch break. Due to this i was in a good mood to hear about "[Microservices @ Spotify](http://gotocon.com/dl/goto-berlin-2015/slides/KevinGoldsmith_MicroservicesSpotify.pdf)". [Kevin Goldsmith](https://twitter.com/KevinGoldsmith) showed how Spotify stays prodictive and embrace change while growing that fast. Basically he talks about the need for *autonomous* feature teams as their way out of this from an organisational point of view and microservices from a technical point of view.

These kind of talks from Internet scale companies like Twitter, Netflix, Amazon or like in this case Spotify are pretty impressive just from the numbers. When hearing about ~800 active running microservices developed by currently 600+ developers and 90+ teams - this is where you try to get your head around this. Especially when thinking about, that these guys are probably 10x faster and able to change their business that the normal enterprise is. Very interessting, although there was "nothing new", because it was just an experience report.

### DevOps: Next
For me there were two more talks about Containers and then just before the party started there was this Keynote from [Nicole Forsgren](https://twitter.com/nicolefv) called "[DevOps: Next](http://gotocon.com/dl/goto-berlin-2015/slides/NicoleForsgren_PartyKeynoteDevOpsNext.pdf)". It was a very interessting talk. She referenced the [Puppet Labs 2015 State of DevOps Report](https://puppetlabs.com/2015-devops-report), an annually survey about the implementation state of DevOps in organisations. 

With different numbers she showed that DevOps together with Lean practices enables not only IT performance but also organizational performance. Changing IT from a cost center to something that lives in the heard of the organization as a fundamental profit center. Very cool talk, although as a John Doe developer it is not something that can be taken home to be implemented, but more as a general advice.

Right after this the conference party took place. I think i don't have to write much about it expect it was very cool... :)

### The second day of the conference

<img style="float:right; padding-left: 10px; width:200px;" src="{{site.url}}/images/2015-12-15-gotober-recap/challenger_explosion.jpg">
On friday, the first keynote was another space talk, held from [Stephen Carver](https://twitter.com/CarverStephen) called "[Space Shuttle](http://gotocon.com/dl/goto-berlin-2015/slides/StephenCarver_MorningKeynoteSpaceShuttle.pdf)". I think it was even cooler than the Keynote from Day 1. 

It was about the space shuttle program and the accidents of Challenger and Columbia that occur. He digged down into the details why these disasters happend from a technical point of view and that they could have been avoided if only NASA would have act like a giantic team and they did with all this politics and pressure behind it. With this, he builded a pretty impressive bridge between this topic and this whole agile movement. Absolutly great talk.

Right after this talk, there was [Nicola Paolucci](https://twitter.com/durdn) from Atlassian talking about "[The age of orchestration](http://gotocon.com/dl/goto-berlin-2015/slides/NicolaPaolucci_TheAgeOfOrchestrationFromDockerBasicsToClusterManagement.pdf)", where he introduced the different orchestration techniques that are avaliable in the docker ecosystem. Going from basic stuff like docker compose and docker machine to a little more advanced topics like docker swarm. Since i already played with it, it was not that much of new information, but creating a production ready confluence setup stays still impressive.

Then i went to the talk from [Jez Humble](https://twitter.com/jezhumble) called "[Why Scaling Agile Doesn't Work](http://gotocon.com/dl/goto-berlin-2015/slides/JezHumble_WhyScalingAgileDoesntWork.pdf)". Jez, at least for me, is a pretty amazing speaker. I already saw him last year at the conference and so unfortunately both talks overlapped a little bit. Nevertheless he introduced a phrase i wasn't aware of: "Water-Scrum-fall". he explained that even if you optimize your dev department with stuff like agile practices and so on, it is just relevant to a very little degree if the projects that you work on are planned a year before that. And the reactions of the users to this projects will never influce the decision to even start with the project, because they see it that later in time. He promoted the ideas behind continous delivery and lean. Also a very good talk.

<img style="float:left; padding-right: 10px; width:100px;" src="{{site.url}}/images/2015-12-15-gotober-recap/kubernetes.png">
The last talk i attended to was from [Brian Dorsey](https://twitter.com/briandorsey) from Google. He introduced [Kubernetes](http://kubernetes.io/), created by Google. Kubernetes is another possibility to orchestrate containers. Since i had already had heard about it but had the time looking into it, i was really looking forward to hearing about it. It was a very cool talk. Firing up VIM in a talk is always a good idea :)

### Conclusion

To sum it up, GOTO Conference was pretty amazing. Most of the talks were really enlightening. Especially the keynotes this year were super cool. 
<a href="https://www.youtube.com/user/GotoConferences">
<img style="float:right; padding: 10px;" src="{{site.url}}/images/2015-12-15-gotober-recap/youtube.png"></a>
If you're interessted in looking at some of the talks, i think the recordings are being published at their [Youtube Channel](https://www.youtube.com/user/GotoConferences). I'm really looking forward to next year.
