---
layout: post
title: Web optimisation in 2016
date: 2016-03-02 17:40:41.000000000 +00:00
---
Web optimisation is important; it's how we make our websites fast - which in turn keeps our users happy and helps us to maintain a good SEO profile.

The way in which we build and optimise websites is driven by the foundation of data communication across the web: HTTP - the *Hyper Text Transfer Protocol*. 

The first documented version of HTTP was HTTP/0.9 dated back to 1991. We're now 25 years on and HTTP/2 is here. Nearly [all major browsers support it](http://caniuse.com/#feat=http2) and have done for some time. 

In this article let's talk about the state of web optimisation, the limitations which drive today's web development workflow and how HTTP/2 can help remove those limitations and simplify our toolchain.

At the time of me writing this article the current most widely used HTTP standard implemented and used by web servers is [RFC 2616](https://tools.ietf.org/html/rfc2616), or to you and me: HTTP/1.1, which is rapidly being replaced by HTTP/2. [HTTP/2 statistics from keyCDN](https://www.keycdn.com/blog/http2-statistics/). 

### HTTP/1.x

HTTP/1.1 was initially released in 1999 way before we started building websites at the size and complexity we do today with help from frameworks such as [Angular.js](https://angular.io/), [Backbone](http://backbonejs.org/) and [Bootstrap](http://getbootstrap.com/). These frameworks turn what used to be a single page website with a few hyperlinks into a fully fledged web application with large amounts of DOM manipulation, two way data bindings and data consumption across many resources and domains.

Our websites have become more sassy, responsive and complex - evolving into web applications. As a result of this increase in complexity the overhead of delivering these webpages from a server has grown exponentially. 

This overhead comes in the way of TCP connections to multiple origin servers in order to gather all the resources that a particular web application needs to function.

All these connections deliver a magnitude of different resource types, including; JavaScript, JSON, XML, HTML, images and CSS. All of which have an impact on webpage load time, download costs from mobile or ISP providers and just general annoyance whilst your end users sit there - waiting for all these connections to come through sequentially and the webpage to be ready. 

Sequentially is an important term here and is something I'm going to expand on further in this article.

Let's take [theguardian.com](http://www.theguardian.com/uk) (Desktop, Google Chrome) for example, a content and image heavy webpage. In an Incognito session on the initial page load  `theguardian.com` makes 218 requests — weighing in at 2.9mb — taking a grand total of 38 seconds to finish loading.

![](/images/webpage-load.png)

Now let's assume that all 218 assets on `theguardian.com` are all served from the same domain over HTTP/1.1. 

The HTTP/1.1 protocol specifies that if the 'Keep-Alive' header is present, which it is by default on almost every web server, the connection can be kept open and all requests to `theguardian.com` can take place through that one single TCP connection. This differs greatly from HTTP/1.0 where a new TCP connection would have to be opened for each request, so clearly a large performance benefit came from HTTP/1.0 to HTTP/1.1.

HTTP/1.1 reduced overhead when requesting resources from the same domain in terms of open TCP connections. It did not however specify any different mechanism to how resources are transferred over the wire. In the HTTP/1.1 protocol same domain resources are transferred sequentially one after the other.

The limitations of pre-HTTP/2 have driven the way we build and develop our web applications today. 

If you have been developing for the modern web over the past few years you'll be no doubt familiar with; concatenation, minification, spiriting and lazy loading of resources. You probably do all this through a magnitude of technologies all wired together in a build step facilitated by tools such as [Grunt](http://gruntjs.com/) or [Gulp](http://gulpjs.com/).  

This process is not only slow but time consuming too. It's additional overhead that we could all do without and the whole reason why we have this additional overhead comes back to the foundations of HTTP and it's limitations. 

We do several things to help overcome these limitations such as concatenating resources where possible - this reduces the amount of sequentially transferred files in a single TCP connection resulting in resources delivered faster to the client. 

We also began distributing static content such as HTML resources over different domains/origin servers to our assets such as JavaScript and CSS in an attempt to overcome the sequential limitation of HTTP/1.1, giving us extra TCP connections and avoiding head-of-line blocking.

### HTTP/2

Sites such as Facebook, Twitter and Google have recently swapped HTTP/1.1 out for HTTP/2, many CDNs such as Akamai and CloudFlare are also defaulting to HTTP/2. 

HTTP/2 is fundamentally different in it's design and implementation to that of HTTP/1.1. The most important change that this article is built around is this: 

**HTTP/2 can prioritise and deliver multiple resources in parallel through way of TCP multiplexing.**

So what does this mean for web optimisation and what are the best practices today for optimising the delivery of your web applications? 

#### Stop concatenating assets.

HTTP/2 uses TCP multiplexing to allow parallel downloading of multiple resources over a single TCP connection. 

It does this well for many small resources which means concatenating your entire web application into one single resource per type is now a bad practice and you'll begin seeing negative results on download times.

Instead of concatenating assets you should now be focusing more on your caching policies for each individual resource in your application. 

Previously cache invalidations were rife every time you changed a single line of JavaScript or CSS in your application. This forced browsers to re-download megabytes of data even if only a single line of JavaScript was changed. 

Delivering non-concatenated resources means you only invalidate portions of your application when you update or change something decreasing load on your servers and page load times to your end users.

#### Stop inlining assets.

[webpack](https://webpack.github.io/) is a tool that has been gaining traction over the past few months or so, it's a build tool which concatenates and minifies your JavaScript and CSS, whilst webpack is not the only tool that can achieve this it's certainly one of the most popular today.

webpack has a very important feature - after concatenating and minifying your JavaScript and CSS it then inlines it into your HTML page, a big win for HTTP/1.1 but a major fail for HTTP/2.

HTTP/2 works on a prioritisation basis when serving resources. You can specify weights and priorities on how your web server should respond to requests and which resources should have a higher priority when being served to the client.

HTML should always be your web servers main priority when serving a web application. Supporting progressive enhancement CSS and JavaScript should come second and not prevent valuable, core content from being rendered. This ensures your webpage feels snappy and responsive to your end user and has a positive impact on SEO. 

Inlining CSS and JavaScript into a HTML page over HTTP/2 essentially ranks your additional page enhancements the same priority as HTML which should almost never be the case. 

Instead you should be leveraging [server push](https://tools.ietf.org/html/rfc7540#section-8.2), a HTTP/2 feature which essentially works the same as in-lining of resources just from the servers end rather than the clients pre-emptively delivering resources to the client before the client requests it. 

For example: when a client requests 'index.html', the web server is configured to say: *"Hey, you want index.html? Okay, but you're going to need app.js too!"*. 

Server push however doesn't break the prioritisation features of HTTP/2. I talk more about sever push in my article [HTTP/2 and content delivery](https://blog.jacobclark.xyz/http-2-and-content-delivery/).

#### Stop serving assets from multiple domains.

This one is relatively simple. The more domains you serve your resources from the more TCP connections your end user has to open which in turn increases download overhead and breaks prioritisation of resource delivery. Along with TCP connections you also have DNS query overhead, TLS/SSL handshakes/negotiations and more points of failure which you'll have to handle in your web application.

Deliver your content from one single domain, whether that's a CDN such as [CloudFlare](https://cloudflare.com), your own web server or an [Amazon S3 bucket](https://aws.amazon.com/s3). Keep those TCP connections to a minimum.

### Summary

Clearly HTTP/2 is a technology you should be implementing today where possible on every new web application you create. 

HTTP/2 will help you simplify your build process, give you more control over how your resources are served to the client and it'll even please your users by allowing for a more responsive and snappier experience across the board.

HTTP/2 however is not a silver bullet to simplify your build chain, continue to minify your resources where possible - the smaller the file the smaller the download. Continue to leverage browser caching, reduce DNS lookups and always make sure you use a CDN for static resources that change infrequently.
 
If you'd like to go further and find out more about the inner workings of HTTP/2, you can do so in my blog post: [HTTP/2 and content delivery](https://blog.jacobclark.xyz/http-2-and-content-delivery/), if you'd then like to implement a HTTP/2 web server, I outlined how to go about that using Node.js in my article [HTTP/2 with Node.js](https://blog.jacobclark.xyz/http-2-with-node-js/)

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
