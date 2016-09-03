---
layout: post
title: HTTP/2 and content delivery
date: 2016-02-19 21:34:17.000000000 +00:00
---
Web servers supporting HTTP/2 can take advantage of a couple new features introduced as part of the new specification designed to improve how we deliver content over HTTP, and as a result, significantly improve the end user experience of our applications!

This article is based of [H2O](https://h2o.examp1e.net/), a modern and fast HTTP/2 & HTTP/1 web server.

The main two features I’m going to cover here are Prioritisation and Cache-Aware Server-Push.

HTTP/2 is very different in implementation and design to that of HTTP/1.

HTTP/2 is a binary protocol which uses streams and frames in order to exchange data, every request made to a HTTP/2 server will open up a new stream for each message sent back to the client, these streams are split up into multiple frames which have attributes associated with them. Attributes such as: parent streams, data, headers, weight, push_promise and more, check out the official [specification](https://http2.github.io/http2-spec/#StreamsLayer) for more in depth detail on streams and frames.

## Prioritisation 

When writing web applications, we attempt to resolve our assets in a particular order: HTML (index.html) and stylesheets first, then JavaScript and images.

We do this in order to achieve what is perceived to be a fast loading web application, delivering content and styles first makes our applications appear to load quickly whilst the browser continues to fetch all other assets such as JavaScript and images under the hood.

Browsers are able to do this because of the positioning of script tags and external resources within our HTML hierarchy, allowing us to achieve a form of prioritisation for asset loading.

Today, HTTP/2 is able to prioritise responses based on the ‘weighting’ of a frame within a stream (dependency weighting), which can be specified in the form of a HTTP request header. Meaning it is possible for us to specify the priority manually on each HTTP request no matter where the markup is located on the page.

Many browsers are able to dynamically calculate which resources should be prioritised when performing HTTP/2 requests and block those which are less important, meaning it is rare in general usage to pass the weight header in manually, unless you’re developing back-end services.

Many modern web servers supporting HTTP/2 will fall back to server-side prioritisation based on MIME type when no other prioritisation in the form of a header (manual or browser generated) has been offered in the request.

[Benchmark of download timings over HTTP/2 via H2O](https://h2o.examp1e.net/benchmarks.html#download-timings)

## Server Push 

HTTP/2 can prioritise the order in which requests are served, but it can also serve (push) responses to the client without the client requesting that resource within that particular stream (bidirectional exchange between the client and the server over HTTP/2). This mechanism is called server-push.

Modern web applications can make a vast amount of HTTP calls to gather all the resources the app needs to run, with HTTP/2, we can configure a web server to push all the content in a single particular stream without the browser needing to request the resources individually.

For example, a client requests an index.html file being served by a web server supporting HTTP/2, our server is configured to serve back the index.html file, and several other resources in this one particular request, including CSS and JavaScript files, all individually weighted for prioritisation.

This means the browser no longer needs to calculate and request resources back and forth to the web server through multiple streams, a single stream is sufficient, a stream is bound to 256 requests, once this cap is hit, a new stream must be opened.

Clearly though, this isn’t always optimal, as web browsers are quite capable of caching static content that does not change. HTTP/2 handles this through CASPer (cache-aware server-push), which essentially means the web server holds some form of snapshot of a web browser’s cache to detect whether or not it needs to push a resource or hold off as the browser already has it cached, reducing the amount of network talk time within an application!

## Facts and figures 

Currently H2O can support an astonishing 256 max concurrent requests per a single HTTP connection, that’s a phenomenal number! Clearly indicating we can now deliver all our content and resources to the browser in one single HTTP request.

H2O can re-prioritise and block on certain assets at the web browser’s request, currently Firefox supports such feature and is able to request H2O blocks requests for images until all CSS and JavaScript assets have been served to the browser in order to deliver a good perceived user experience.

HTTP/2 is now supported in most modern web browsers with the emphasis of TLS by default, with web servers able to fall back to HTTP/1 support for older browsers, now is the time to begin thinking about upgrading and reaping the performance benefits of HTTP/2.

Need a TLS certificate? [Let’s Encrypt](https://letsencrypt.org/) has you covered.

[H2O](https://h2o.examp1e.net/) is a blazingly fast, modern HTTP/1 & HTTP/2 web server written in C.

Visit my [website](https://www.jacob.uk.com), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
