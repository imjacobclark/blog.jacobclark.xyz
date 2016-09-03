---
layout: post
title: Efficient command line navigation with directory stacks
date: 2016-02-19 21:02:16.000000000 +00:00
---
Moving directories is easy, on any command line flavour, it’s hardly the most complex command. The real trick is remembering where that all important directory is you need to get back to when you’ve moved up, down and sideways in your file system!

Have no fear, pushd and popd are here! Yep, command line directory systems have stacks too (obviously).

If you’re not familiar with what a stack is, or how it works, it’s simply a data structure, like a stack of plates. You place a plate on top of the stack, you take the plate off the stack, you can get to the next plate, so and and so forth. It’s a data structure seen time and time again throughout computer science, you can read more about it on [Wikipedia](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)).

If your system supports directory stacks, each directory you switch to will be pushed onto the stack. This means if you want to quickly return back to your previous directory, you can pop it back off the stack! These stacks can go on and on, they’re not limited to just a single directory change either!

Let’s start off in our home directory:

```shell
$ cd ~/
```

Let’s change directories into ‘etc/apache2’, because I need to change a few config options in Apache:

```shell
$ cd /etc/apache2
```

Okay, so in theory, we should have two directories now in our stack (providing you’re in a new shell session and you haven’t done any previous navigation around your file system), we can run the ‘dirs’ command to see what's currently in our stack:

```shell
$ dirs
/etc/apache2 ~
```

Okay brilliant, we can see both ‘/etc/apache2’ and ‘~’ are in our stack! The directory stack is a last in first out stack, meaning anything you place on the stack will come off in exactly that order, like a stack of plates.

If we run popd at this point, it’ll take us back to our home directory:

```shell
$ popd
~
```

You can do even more complex navigation and move forwards/backwards where necessary, meaning you don’t need to keep moving up through your command history to find the right directory ever again (or keep those little pockets of brain energy remembering where that darn log file is)!

pushd accepts directories as parameters too, it’ll take parameters and switch your current directory whilst pushing it onto the stack, useful if you’re working with shell scripts:

```shell
$ pushd /etc/apache2/sites_avaiable
/etc/apache2/sites_avaiable ~
$ cd ~/Desktop
$ popd
/etc/apache2/sites_avaiable ~
$ pwd
/etc/apache2/sites_avaiable
$ popd
~
$ pwd
/Users/jacobclark
$ popd
popd: directory stack empty
```

Visit my [website](https://www.jacob.uk.com), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
