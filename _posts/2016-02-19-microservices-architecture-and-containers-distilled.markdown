---
layout: post
title: Microservices Architecture and Containers distilled
date: 2016-02-19 21:05:13.000000000 +00:00
---
Microservices architectures have taken off dramatically over the recent years and newly emerging technologies such as CoreOS, Docker, Mesophere and Digital Ocean are making it easier than ever before to build highly available, fault tolerant and scalable enterprise ready applications on the fly.

This article is aimed at those who have heard the terms microservices and containers mentioned about the office but are overwhelmed by the vast amount of whitepapers out there by companies such as Netflix. I hope this article will serve as a digestible amount of theoretical information as well as some technical on how you can start building scalable applications at Facebook quality without the overhead, cost or infrastructure.

## What is a microservices architecture?

Simply put, an application that follows the microservices architecture pattern is simply a system that is logically split up into several smaller systems designed to communicate over standardised protocols such as REST or lightweight message queuing.

Here are the main points I feel explain what a microservices architecture is, and why it’s important:

* Applications designed as a suite of small services opposed to monolithic codebases

* Such services are easy to scale, distributed across data centres and geo-locations allowing for fault tolerance and high availability

* Microservices should be designed with failure in mind

* Easier to automate deployments allowing for continuous delivery

* Code and applications are independent and de-centralised, not tying developers or operations to a particular stack, framework or language

* All services are de-coupled and should implement where possible a standard communication protocol such as a RESTful interface or message queue for communication with other parts of the application

There are a huge amount of positives that come with a microservices architecture for developers and the dev-ops culture, easier deployments, the ability to throw pieces of an application out and replace it with a different stack, improved scalability and ability to achieve continuous delivery, however, with these positives come negatives too, and a microservices architecture isn’t a bed of roses.

It’s important to understand your technical resources on-hand before designing a microservices architecture, and this is where we would normally start talking about the dev-ops movement and the forever decreasing divide between developers and operations, but I’m going to leave that for another article.

However, this isn’t the full story, a more in-depth explanation on microservices by Martin Fowler can be found [here](http://martinfowler.com/articles/microservices.html).

## What is a container?

In a nutshell, a container is a way to package an application, move it around environments and deploy it anywhere a container engine is present!

Containers are an alternative to hypverised virtualisation environments and fit very well into most cloud deployment architectures.
Containers are:

* A lightweight environment for running applications

* Bundled applications running within isolated environments upon the same kernel in their own userspace

* Portable cross environments, for example a container can be pushed from several developers laptops through int, test and live and stay consistent throughout

* Isolated resources which do not interfere wither other running containers on the same kernel

Already we can see the advantages of using containers, they’re cheaper — spinning up several cloud instances to host multiple applications is expensive and time consuming, containers can be spun up in a single VM and run side-by-side with very little overhead. 

Environments are consistent and reproducible cross host environments. Resource utilisation can be limited and monitored with ease, allowing for better scalability.

As with everything, there are downsides to using containers, they are not full protection from security threats, a containers process and resources are isolated, however, you should continue to secure your applications as you would if they were deployed onto a bare metal OS. Many container engines do not follow a standard, it is hard to migrate from one container engine to another once your choice has been made.

There are a vast number of discussions around the web focusing on Containers opposed to Virtualisation, [here](http://blog.smartbear.com/web-monitoring/why-containers-instead-of-hypervisors/) is just one of many.

## Running a microservices architecture on containers

The philosophy of microservices emphasises on small, scalable applications that are reliable, easy to deploy, scale and maintain.

Virtualised infrastructures running automation tools such as Puppet (or even shell scripting) can be unreliable, in-consistent and expensive.

Container based solutions are a great choice for deploying microservices on, they can be clustered and many container deployment environments support automatic service discovery for the addition of new resources to a cluster as well as automatic failover.

Containers can also be deployed on a wide variety of environments, from Platform as a Service, to cloud server instances or even dedicated servers running traditional GNU/Linux distributions.

Visit my [website](https://www.jacob.uk.com), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
