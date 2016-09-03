---
layout: post
title: Building a continuous deployment pipeline for the cloud
---
The theory of continuous deployment in software development is simple, it is the art of having a release candidate at every stage of your development cycle ready to go into live.

This article is going to cover exactly what continuous deployment is at a conceptual level, we will move on to building a continuous integration pipeline using the cloud provider Digital Ocean and these following technologies:

* Ubuntu
* CloudInit
* .deb packages
* Jenkins
* GitHub

Continuous deployment doesn't have to be expensive or complicated. It can dramatically improve your product's stability and greatly increase your overall confidence in supporting a live product 24/7.

If we wanted to summarise continuous deployment with a diagram, it might look something like this:

![](/content/images/2016/03/ContinousDeployment.png)

Simple right? Maybe not, let's break this diagram down a little.
