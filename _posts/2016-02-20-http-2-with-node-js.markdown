---
layout: post
title: HTTP/2 with Node.js
date: 2016-02-20 18:54:49.000000000 +00:00
---
HTTP/2 with Node.js Let’s take a crash course on how we can build HTTP/2 ready applications in Node.js.

Firstly, many clients and browsers are not planning on implementing the ability to use HTTP/2 over plain-text, even though this is in the protocols specification. This notion comes as a wider community effort to move towards a more secure and encrypted internet by default.

For us however, this means if we intend on using HTTP/2 in production, we must supply a valid TLS certificate!

If you wish to use a valid TLS certificate which is signed by an authority and not self signed, I recommend [Let’s Encrypt](https://letsencrypt.org/), a free, automated and open certificate authority!

For the purpose of this guide, we’re going to be using self-signed certificates, feel free to replace these steps with a certificate you have acquired from Let’s Encrypt (or another certificate authority).

This is the real nitty gritty stuff of openssl, let’s quickly zip past it and get ourselves a certificate.

## Generating a TLS certificate 

Create a new project directory and assuming you have openssh installed, lets generate a 2048 bit private key for our server, note that we specify the key passphrase as `x` using the flag `-passout pass:x`:

```shell
$ mkdir node-http && cd node-http
$ openssl genrsa -des3 -passout pass:x -out server.pass.key 2048
```

We now need to remove the passphrase from this key so it can be loaded into our HTTP server, we then remove the original key:

```shell
$ openssl rsa -passin pass:x -in server.pass.key -out server.key
$ rm server.pass.key
```

Now we need to generate the certificate signing request in order to validate who we are (even though we are self signing the certificate, we must present a CSR), you’ll be prompted for information such as your country whereabouts, name, organisation details and common name, enter these as appropriate:

```shell
$ openssl req -new -key server.key -out server.csr
```

Finally, we can (self) sign our certificate:

```shell
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

You now should have three extra files within the node-http2 directory:

* server.crt — Your new TLS certificate
* server.key — Your TLS certificate private key
* server.csr — Your TLS certificate signing request

Now we have our certificates ready, we can move on to writing a basic HTTP/2 node server.

We’re going to be using the mature node library [node-spdy](https://github.com/indutny/node-spdy) in this article.

It’s worth noting, Express 5.0 will eventually have HTTP/2 support out the box, although there is uncertainty within the community as to when Express 5.0 will be released, Koa also has HTTP/2 support from additional middleware.

Whilst spdy is being deprecated, node-spdy has a fantastic API and full support for HTTP/2 and fallback to HTTPS, so there is currently no reason not to use it in production.

Building our HTTP/2 node server Let’s start by installing the npm module:

```shell
$ npm install spdy
```

Create an app.js file and require spdy and fs:

```javascript
'use strict';

let spdy = require('spdy'),
    fs = require('fs');
```

We will need to provide some basic configuration to spdy, to start with we will simply specify the locations of the certificate and private key we just created:

```javascript
let options = {
    key: fs.readFileSync(__dirname + '/server.key'),
    cert: fs.readFileSync(__dirname + '/server.crt')
};
```

Finally we can create our basic HTTP/2 server:

```javascript
spdy.createServer(options, function(req, res) {
    res.writeHead(200);
    res.end('Hello world over HTTP/2');
}).listen(3000);
```

Putting all this together:

```javascript
'use strict';

let spdy = require('spdy'),
    fs = require('fs');
    
let options = {
    key: fs.readFileSync(__dirname + '/server.key'),
    cert: fs.readFileSync(__dirname + '/server.crt')
};

spdy.createServer(options, function(req, res) {
    res.writeHead(200);
    res.end('Hello world over HTTP/2');
}).listen(3000);
```

Here we have created HTTP/2 server which responds with ‘Hello world over HTTP/2’ in the streams ‘body’ frame.

Head over to https://localhost:3000 (note https), you should be presented with a certificate warning error stating it is not trusted, this is because we self-signed our certificate and your browser is warning you that the certificate is not signed by a trusted authority, accept the warning and continue anyway.

If all is well, you should see:

![](/images/hello-world.png)

If we inspect the Chrome network tab, we can see the protocol has been listed as H2, confirming we are infact viewing the webpage over HTTP/2:

![](/images/localhost-h2.png)

## HTTP/2 Server Push 

Server-Push is the ability for HTTP/2 enabled web servers to push content back to the client in a single request without the browser requesting it first.

Clearly this functionality reduces the amount of overhead for the browser in having to request content back and forth, instead we specify it in our server configuration and forget about it.

node-spdy allows us to programmatically specify pushed content as follows:

```javascript
let spdy = require('spdy'),
    fs = require('fs');

let options = {
    key: fs.readFileSync(__dirname + '/server.key'),
    cert:  fs.readFileSync(__dirname + '/server.crt')
};

spdy.createServer(options, function(req, res) {
    let stream = res
        .push('/main.js', {
            request: {
                accept: ‘*/\*’
            },
            response: {
                'content-type': 'application/javascript'
            }
        })
        .end('console.log("Hello World");');
    
    res.writeHead(200);
    res.end('<script src="/main.js"></script>');
}).listen(3000);
```

Start the application again, this time inspecting the JavaScript console in your browser, you should see:

![](/images/helloworld-console.png)

We can verify this was pushed by opening up Chrome internals for HTTP/2: chrome://net-internals/#http2, locating localhost:3000 and observing the active streams and unclaimed pushes columns at 1. You may also drill down into this information further and inspect exactly what happened during the request/response cycle by clicking the ID URL.

![](/images/http2-sessions.png)

HTTP/2 operates on a concept of requests, streams and frames, every request that HTTP/2 responds too opens a series of streams for each message that needs to be sent to the client, each stream is split up into separate frames, there are several frame types including: data, headers, priority, push_promise and several others.

The frame we define above is a push_promise, this informs the client of some data it will receive at some point in the future, when the client will receive this data is calculated by many HTTP/2 prioritisation factors which I’m not covering in this guide.

We specify a filename for the stream, request and response attributes including content type and accept policies and we end the stream with the actual body, this could be a file from the file system, for simplicity it’s just an inline console.log in this example, this console.log will be delivered and executed on the client

We also updated our servers response body to reference the script we are pushing from the server so the browser knows to execute it.

There we have it, the foundations of a HTTP/2 node application.

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
