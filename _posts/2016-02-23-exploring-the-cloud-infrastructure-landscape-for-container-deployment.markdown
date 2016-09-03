---
layout: post
title: Exploring the cloud infrastructure landscape for container deployment
date: 2016-02-23 14:31:34.000000000 +00:00
---
The landscape of application deployment is changing constantly, deploying applications into highly available infrastructures has got easier over the past few years, first came the introduction of public cloud development platforms as a service such as Heroku, ElasticBeanstalk and OpenShift.

Cloud provider tools also emerged such as CloudFormation by Amazon Web Services for cloud infrastructure automation. This can be tightly coupled to your application deployment and bootstrapping process with features such as logging, autoscaling and application deployment.

The above examples have high amounts of vendor lock-in, which can be an issue for some development teams, company policies and even data protection. So now we find ourselves asking the question, how do I architect my own cloud infrastructure?

The past few years have seen the relentless rise of container engines from [Docker](https://www.docker.com/), [lmctfy](https://github.com/google/lmctfy) and [LXC](https://linuxcontainers.org/), if you're unfamiliar with what a container is, my recent article [Microservices and Containers distilled](https://blog.jacob.uk.com/microservices-architecture-and-containers-distilled/) will be a useful read at this stage, the article also covers Microservices, an architecture pattern for building fine-grained application components which also fit directly into the context of this article.

Containers allow us to build our applications into consistently portable environments, cross operating systems, data centres and even container engines themselves. This article will focus on exploring how to achieving a highly available cloud infrastructure leveraging container technology.

At this stage it's important we evaluate our requirements against that of platforms such as Heroku and CloudFormation:

* Environment provisioning - the ability to bring nodes and applications up rapidly.
* Portability, avoiding vendor lock-in so you may switch platforms at any time.
* Clustering - the ability to cluster nodes and launch services onto them. These nodes should be cloud provider independent without lock-in to allow for high availability cross multiple data centres.
* Service Discovery - the ability to detect when services are unavailable or unresponsive and take action accordingly.

## Environment provisioning and portability

Containers allow us to remove the need for automation and bootstrapping tools such as [Ansible](https://www.ansible.com/), [Puppet](https://puppetlabs.com/) and [Chef](https://www.chef.io/chef/) which you would normally expect to find on bare metal cloud infrastructures. Containers allow us to build once and deploy everywhere.

Container images can be created as templates for applications to be deployed into, bootstrapping and automation of such images is generally done once per each type of application, be it Node.js or Java. In comparison to OpenShift and Heroku who achieve similar functionality through 'Cartridges' and 'Buildpacks', both of which focus on the bootstrapping of an environment and deployment of an application through standard shell scripting.

Containers are portable by design, meaning you avoid cloud provider and operating system level lock-in, of-course, as with anything, badly designed application container implementations can be lock you in to particular container engines, however, with the correct type of implementation, your application shouldn't be bound to engines such as Docker, lmctfy or LXC and you should at any time be able to de-containerise your application and deploy anywhere you like.

The [Docker](https://www.docker.com/) platform has received huge amounts of traction recently and as a result official container images from many software vendors now exist for Docker, [nginx](https://www.nginx.com/resources/wiki/) for example is available on the [Docker Hub Registry](https://hub.docker.com/).

## Clustering and high availability

In any highly available infrastructure the need for clustering of nodes and the ability to deploy applications cross infrastructure is considered an absolute necessity, but this requirement isn't particularly easy.

There are many routes we may go down, again, automation tools such as Puppet allow us to build manifests and template configurations to automate the deployment of our applications across a cluster, we could in theory use automation software to pull and distribute our containers, however this method still requires huge amounts of abstraction and dependencies to operate and so isn't ideal in a massive server deployment.

At the moment the container clustering landscape is evolving everyday, much of the current focus is on tools that allow us to manage containers as a single unit allowing developers to focus on building applications rather than their infrastructure, some very promising tools are emerging, including:

[Swarm](https://blog.docker.com/2015/02/scaling-docker-with-swarm/) - A solution currently being built for the Docker platform to allow for clustering of nodes for container scheduling. Swarm is almost 100% transparent to the Docker API, meaning if you already know how to launch a Docker container, you know how to launch a Docker container into a cluster.

[Fleet](https://coreos.com/using-coreos/clustering/) - A distributed init system for systemd, allowing service files to be loaded, started and monitored across clusters. Fleet achieves this by tying into Etcd, a service discovery and distributed key value store.

[Kubernetes](http://kubernetes.io/) - Cluster management tool for loading containers onto many nodes within a cluster and managing them as once instance, specifically built for Docker containers.

[Mesosphere](https://mesosphere.com/) - A completely different contender in the infrastructure landscape, built as a kernel and designed for datacenter scale deployments, whilst keeping developers at it's focus to create one unified cluster interface.

At the moment it's not clear on an obvious winner, all four solutions above are still evolving and no clear winner may ever emerge. However it is clear that each of the four above are working to an open operating model of 'Batteries included, but removable' essentially meaning you can change the working parts of each system to fit you infrastructure needs.

## Service discovery

We could integrate into monitoring tools such as [Nagios](https://www.nagios.org/), [DataDog](https://www.datadoghq.com/) or [OpsViewfor](https://www.opsview.com/) service monitoring and availability events, but this introduces further dependencies into our infrastructure, it doesn't allow us to tightly couple our service discovery with running applications, so the ability to take action on services being unavailable would require even further dependent applications or scripts.

Whilst we require monitoring and availability checking in any stack to ensure the 'lights are kept on', this isn't one of our four main concrete requirements for building a highly available architecture, it's generally considered good practise to use such tools, but it's not a core enabler at the moment.

Service Discovery is essentially a global persistence store accessible across an entire cluster which can be subscribed to at an application level. Services running on a cluster will make their existence know by registering themselves onto the persistence store, typically by their IP and Port Number, other applications such as load balancers will then subscribe to this information and take action if it changes. In an nginx setup for a load balanced JSON API, we may subscribe the load balancer pool so when new instances of the JSON API are available on the cluster, the load balancer will know it has additional resources to send requests too, improving availability.

Here are some emerging service discovery solutions currently available to use:

[Etcd](https://coreos.com/etcd/) & [confd](http://www.confd.io/) - A distributed key-value store and cluster discovery service, developed as a fundamental component of the CoreOS ecosystem, combined with confd etcd becomes a powerful service discovery tool for dynamically reconfiguring services such as load balancers to ensure availability. Etcd also provides a HTTP interface for communication.

[Consul](https://www.consul.io/) & confd -Similar to Etcd, Consul however provides a DNS query interface for service discovery.

[Mesosphere](https://docs.mesosphere.com/administration/service-discovery-with-marathon-lb/service-discovery/) & [Marathon](https://github.com/mesosphere/marathon) - mesosphere has service discovery built in, when combined with Marathon mesosphere is able to deploy, manage and sync a HAProxy service based on current service availability within a cluster.

## Summary

It's clear we are overwhelmed with choice right now for building production ready infrastructures for container deployments, if I had to put my bets on now, I'd bet on [CoreOS](https://coreos.com/) and [Mesosphere](https://mesosphere.com/).

CoreOS and mesosphere both offer complete solutions to application deployment through containers, commercial support and some heavyweight cloud supplier backing.

Visit my [website](https://www.jacob.uk.com), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
