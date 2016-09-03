---
layout: post
title: Building a Telegram bot with Node.js and Docker
---
[Telegram](https://telegram.org/) is a messaging client which focuses on speed and security, it has clients for the web, desktop (PC + Mac) and most mobile operating systems (Android, iOS and Windows Phone).

It's a highly extensible platform, it has several fully featured API, entirely cloud based, free to use, and open (even their [encryption schema](https://core.telegram.org/schema) is published and open).

A relatively new Telegram feature is the ability to create bots, a Telegram user, operated by software, which can respond to messages and other arbitrary events and send messages back to users who message or subscribe to the chat.

I'm not going to go into any further details about what a bot is and how they work in this post, I'm going to assume you already know and just want to get stuck in building one! If you are interested in learning more, the [Telegram documentation for the bot platform](https://telegram.org/blog/bot-revolution) is a fantastic resource.

##Creating your bot

In order to create your bot, you need to start a conversation with 'The BotFather'. He will get you up and running with your bot in no time, he is your gateway to generating an API token and inputting extra metadata about what your bot does.

In order to start your app, head over to https://telegram.me/botfather and begin a conversation with The BotFather.

Upon opening your conversation you'll be greeted with a list of commands.

![](/content/images/2016/02/botfather-welcome.png)

Commands are used to allow us to instruct a bot to do something, they're provided by the bots developer and brought together really well inside the Telegram interface automatically. 

Lets get started and issue the `/newbot` command:

![](/content/images/2016/02/newbot-name.png)

At this stage, The BotFather just wants a name for our bot, it's what the user will see when they're in a chat with our bot. In this article we're going to build a bot which will notify subscribers when a Reddit [/r/programming](https://www.reddit.com/r/programming) top story changes, so let's call this bot `r/programming latest`.

![](/content/images/2016/02/programming-latest.png)

Now we need to specify a username for our bot, this is how users will search for our bot, it has to be unique and end in `Bot`, I'm going to go for `RedditProgrammingBot`, you won't be able to register this name, if you're following along, choose a different one.

![](/content/images/2016/02/created-bot.png)

You'll notice at this stage we've been provided with an API token, our bot setup is complete! 

Click the `telegram.me` link in the success message to open a chat with your bot, once in the chat, hit the `Start` button.

![](/content/images/2016/02/bot-chat.png)

##Interacting with the API

Telegram bots are interacted with over a REST HTTP API, we're going to be using a npm module to interact with this API, but I think it's worthwhile understanding how things work 'under the hood', so we're going to take a whistle stop tour looking at the basics of the API.

First things first, the base url for the bot API is https://api.telegram.org/, if you attempt to access a resource that doesn't exist or you're not authenticated - you'll be re-directed back to the bot documentation. 

In order to authenticate with the API, you'll need to specify your API token in the request, this is done simply as a HTTP parameter, you'll need to remember to also place the keyword 'bot' at the front of the token, your URL should then look something like https://api.telegram.org/bot11:zz/ where `11:zz` is your API token.

Finally, in order to have a valid API call, we need to specify a valid API method. The API provides a way to get information about a bot, it's the `getMe` command, putting all of the URL scheme together, we get https://api.telegram.org/bot11:zz/getMe.

Try hitting this endpoint in a browser with a real API token, you should see a JSON document returned similar to:

```json
{
  ok: true,
  result: {
    id: 111111,
    first_name: "r/programming latest",
    username: "RedditProgrammingBot"
  }
}
``` 

If something went wrong, or you were re-directed back to the bot API documentation, consult the [Making Requests](https://core.telegram.org/bots/api#making-requests) section of the bot documentation and see if you can spot what's gone wrong. My first suggestion would be ensuring your API token and the URL schema is correct.

Now we are able to communicate with the HTTP API, we can do some really useful stuff, like reading messages sent to our bot! 

Send a message to your bot to start with, nothing fancy, a `Hello World` will suffice. You'll get no response and your bot will sit there, silent.

We now need to turn our attention to the `getUpdates` method within the API. This endpoint will simply return you new updates that have happened since the last time we acknowledged a series of updates.

Updates are stored on the Telegram servers for 24 hours, if your backend doesn't pick up an update within those 24 hours, it'll be lost.

Go ahead and hit the endpoint https://api.telegram.org/bot11:zz/getUpdates, again replacing the API key with yours.

You should see a JSON document returned similar to the following:

```json
{
    ok: true,
    result: [
        {
        update_id: 578313191,
        message: {
            message_id: 2,
            from: {
                id: 1111,
                first_name: "Jacob",
                last_name: "Clark",
                username: "imjacobclark"
            },
            chat: {
                id: 1111,
                first_name: "Jacob",
                last_name: "Clark",
                username: "imjacobclark",
                type: "private"
            },
            date: 1456273350,
            text: "Hello World!"
            }
        }
    ]
}
```

Lovely, a payload of information about the message we just sent to our bot!

Refresh the endpoint, you'll notice the same response is returned, interesting, so how does the API know when we've acknowledged the update? 

Simple, we simply pass a query parameter of `offset` to the `getUpdates` method with the ID of the update we want to acknowledge, incremented by 1.

The ID of the particular message we want to acknowledge above is `578313191`, which means we would need to send a parameter with an offset ID of `578313192` in order to remove it from the getUpdates response.

https://api.telegram.org/bot11:zz/getUpdates?offset=578313192.

As there are no other messages waiting to be acknowledged, you should be returned a similar response to:

```json
{
    "ok": true,
    "result": []
}
```

If you don't, something has gone wrong, check over your URL and if in doubt, consult the [Get  Updates](https://core.telegram.org/bots/api#getupdates) documentation. 

For the scope of this blog post, I'm not going to suggest we interact with the API endpoints directly, instead I suggest we the [https://github.com/yagop/node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api) npm module, it's actively maintained and the source code is very easily to follow and understand, so a great starting point to our application.

##Bringing the bot online
