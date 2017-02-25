---
layout: post
title: Inspecting GNU coreutils ‘whoami’
date: 2016-02-19 20:08:15.000000000 +00:00
---
## Introduction

Let’s talk about the GNU core utilities. 

You probably use them every day in your development activities. **cat**, **mv**, **yes** and **whoami**, just to name a few.

They’re vital in getting things done and being productive on the command line, but how do they work under the hood?

In this post I’m going to talk about **whoami**, infact, we’re going to dive right into the implementation, and talk about how it works, under the hood.

To start with, it might be worthwhile browsing the Git repository over on [GitHub](https://github.com/coreutils/coreutils/tree/master/src) to get a feel of all the programs that make up coreutils.

## whoami

`whoami`, a simple utility which displays the effective username of the person operating command. It’s worth noting whoami has been made obsolete by id, but for the purpose of this post, it’s useful, and a simple introduction on how coreutils work under the hood.

If we take a look at the [source](https://github.com/coreutils/coreutils/blob/master/src/whoami.c) for whoami, it comes in at 91 lines, 91 lines of code to figure out which user is invoking that command. However, a lot of these lines are simply setup functions and commands for the GNU system as a whole, we can actually remove these and see that the actual minimal code for this program comes in at a mere 9 lines of code:

```c
#include <stdio.h>
#include <unistd.h>
#include <pwd.h>

int main (){
 uid_t id = geteuid();
 struct passwd *pwd = getpwuid(id);
 puts (pwd->pw_name);
}
```

Go ahead, if you’re on a unix like system, compile this and run it, you should see your username printed to screen!

```shell
[jacobclark:~/Desktop]$ gcc whoami.c -o whoami
[jacobclark:~/Desktop]$ ./whoami
jacobclark
```

This isn’t very useful for us trying to understand what's happening under the hood, the code above is just doing several external functional calls.

Interestingly enough, whoami is simply just a wrapper around two other very important functions which are provided by **glibc**, or in other words, a c library. 

These c libraries essentially wrap system call functions (to the kernel) and provide access to utilities such as open, printf, etc.

So, GNU coreutils for whoami is simply a wrapper of wrappers, taking two very different functions, one for getting the current users id, and another for taking an arbitrary ID and returning the users name, making whoami one very useful program.

The majority of these wrapper functions are defined in the Open Group unix specification, infact, both wrapper functions we use **geteuid** and **getpwuid** are defined in this specification.

geteuid: [http://pubs.opengroup.org/onlinepubs/009695399/functions/geteuid.html](http://pubs.opengroup.org/onlinepubs/009695399/functions/geteuid.html)

getpwuid: [http://pubs.opengroup.org/onlinepubs/009695399/functions/getpwuid.html](http://pubs.opengroup.org/onlinepubs/009695399/functions/getpwuid.html)

Happy hacking.

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
