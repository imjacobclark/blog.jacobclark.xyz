---
layout: post
title: Native vs Web Applications
date: 2016-02-19 20:46:21.000000000 +00:00
---
I recently came across the article [Web vs. native: let’s concede defeat](http://www.quirksmode.org/blog/archives/2015/05/web_vs_native_l.html) and it got me thinking.

Here is my opinion on the Native vs Web argument.

Building applications cross-platforms is hard. Not only from a UX perspective, but from a technical and business one too. Each and every project stakeholder in a digital applications lifecycle have concerns ranging from usability, the cost of development to production, platform reach and even device constraints.

The landscape in which we consume content is forever changing too. From phablets to e-readers, we need better, more efficient and cost effective ways to distribute our content across the globe to potentially hundreds of different devices and millions of consumers.

Choosing the right technology to develop your next application in can be difficult, do you go with a fully native implementation, or do you wrap a web application inside a native shell? What are the benefits of either? Why does it even matter? I’m going to try and answer these questions.

## What are the benefits of native? 

Data is expensive, so are page load times, not only to your infrastructure running costs but to your consumer too. Native applications allow us to deliver working applications to the client in one download, generally in an easy to find location such as the Google Play Store or Apple App Store.

Native applications work well with the underlying ecosystem they’ve been developed to run within, generally speaking, native applications tend not to stray away from the look, feel and usability of their ecosystem, whether that’s Android or iOS, a Samsung Smart TV, Google Cast or even Android Wear.

So what are the benefits of going native?

1. A great experience is king, building a consistent look and feel to the environment you’re running in not only entices users to use your application, but it keeps them coming back for more, making them feel like they’re getting more out of the ecosystem they have so heavily invested in.

2. Responsiveness, and no, I don’t mean the kind that’s mobile first. Generally speaking, web applications are downloaded every time the user visits them: JavaScript, Images, Stylesheets, you name it. Native applications are a single package, downloaded only once (unless the developer chooses to update, and the user accepts), the user doesn’t have to wait for page loads, the data is already there, waiting.

3. Less infrastructure, because your application is delivered directly to the client, generally using a third-party service such as the Google Play Store, your requirement for scalability now only applies to the data layer, meaning cheaper running costs and less maintenance over time.

4. Native applications support all device functionality, from accelerometers to NFC readers, they’re easier to debug and tune for device and battery performance and generally have a much smaller footprint on the device.

There are many reasons why one may not write an application using native technologies, including: cost, the requirement for universal platform reach on a limited budget, the requirement to deliver a consistent user experience

## What are the benefits of the web? 

The web is changing. The way we develop websites has changed, in fact, we’re no longer developing websites anymore at all. We’re developing fully fledged complex applications with baked in business logic, this change has lead to new development patterns emerging, from MV* to Isomorphic JavaScript, on premise hosting to infrastructure as a service on the cloud. A lot has changed over the recent years.

But, what are the benefits of developing your next application using web technologies. In my opinion there are several, I could list the following reasons:

1. Usability, cross-platform UX makes a better user experience for your particular application.

2. There’s a vibrant ecosystem out there. It’s huge. From mailing lists to [StackOverflow](http://stackoverflow.com/), new technologies and better ways to develop applications emerge daily, the web is a hot topic and an exciting place to be.

3. Companies are constantly pushing web standards further, from the famous [Apple and Adobe Flash controversy](https://en.wikipedia.org/wiki/Apple_and_Adobe_Flash_controversy) pushing open HTML5 standards forward away from proprietary technologies to new W3C standards such as [web components](http://webcomponents.org/) to which allow us to create custom DOM elements.

4. Shipping applications to potential audiences is a breeze, tools such as [Heroku](https://www.heroku.com/), [Docker](https://www.docker.com/) and [Phonegap](http://phonegap.com/) make it easy to deploy scalable, cheap, cross-platform applications and infrastructure quickly.

5. Vast amounts of platforms, libraries, frameworks and tools exist to make your developers lives easier, argued to be the problem with the web in the article [Tools don’t solve the web’s problems, they ARE the problem](http://www.quirksmode.org/blog/archives/2015/05/tools_dont_solv.html).

6. Design standards are quickly emerging for building standard interfaces such as [Material Desgin](http://www.google.com/design/spec/material-design/introduction.html) by Google.

As with native applications, there are many reasons why one may not write an application using web technologies, including: inexperienced developers, specific device implementation requirements such as iBeacon for iOS, performance where lack of may affect the user’s overall experience and even costs where a large user base is expected.

## Why should I even care? 

To be frank, maybe you shouldn’t. These arguments all depend on your situation as a stakeholder in your project, do you care about development time over cost? If you need an application fast and cheap, maybe a web implementation is the one for you. On the other hand if you need something fast and reliable and you’re concerned around page load times and data exchange costs, maybe native is the right path for you.

You should approach these considerations with care, making the wrong choice could have a detrimental effect on your overall applications perception from your end user, choosing the wrong technology could quickly set your application up to fail. Gather information, research how your competitors have built their applications, and make an informed decision over the technology stack you require.

That’s my pennies worth on Native vs Web, I hope it was insightful!

Visit my [website](https://www.jacob.uk.com), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
