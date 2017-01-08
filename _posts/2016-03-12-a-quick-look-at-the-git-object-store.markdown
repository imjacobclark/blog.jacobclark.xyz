---
layout: post
title: A quick look at the git object store
date: 2016-03-12 18:09:10.000000000 +00:00
---
Let's talk about some of the internals of git and how it stores and tracks objects within the `.git` directory. 

If you're unaware of what the `.git` directory is, it's simply a space that git uses to store your repositories data, the directory is created when you run `git init`. Information such as binary objects and plain text files for commits and commit data, remote server information and information about branch locations are stored within.

The key concept throughout this entire article is very simple - pretty much every operation you do in git creates objects with a bunch of metadata which point to some more objects with a bunch of metadata and so on so for forth. That's pretty much it.


### Git Plumbing

First, create a new directory and initialise it as a new git project.

```shell
$ mkdir git && cd git
$ git init
```

We can now take a look at the `.git` directory and the contents within it.

```text
$ tree .git
.git
|___branches
|___config
|___description
___HEAD
|___hooks
| |___applypatch-msg.sample
| |___commit-msg.sample
| |___post-update.sample
| |___pre-applypatch.sample
| |___pre-commit.sample
| |___pre-push.sample
| |___pre-rebase.sample
| |___prepare-commit-msg.sample
| |___update.sample
|___info
| |___exclude
|___objects
| |___info
| |___pack
|___refs
| |___heads
| |___tags
```

For the moment most of the directories git created are empty. This is because we haven't begun tracking any files or directories yet.

Under the hood git is nothing more than a key-value store, it takes a file and compresses it and then stores it against its computed SHA1 as a binary object.

Git has a low level plumbing command called `hash-object`.

[hash-object](https://git-scm.com/docs/git-hash-object) will take something and compute its object ID. For example we can compute the object ID of the string 'Hello World'.

```shell
$ echo "Hello World" | git hash-object --stdin
557db03de997c86a4a028e1ebd3a1ceb225be238
```

You can try this on your machine and you should see the exact same output as above. 

When we ask git to track a file this is exactly what happens in order to compute an ID to store the information against. 

We can take this command one step further and ask git to take the string "Hello World" and store it as an object within its object store.


```shell
$ echo "Hello World" | git hash-object --stdin -w
557db03de997c86a4a028e1ebd3a1ceb225be238
```

Now if we look at our .git directory tree again we will see some additional files and directories.


```text
$ tree .git
.git
|___branches
|___config
|___description
|___HEAD
|___hooks
| |___applypatch-msg.sample
| |___commit-msg.sample
| |___post-update.sample
| |___pre-applypatch.sample
| |___pre-commit.sample
| |___pre-push.sample
| |___pre-rebase.sample
| |___prepare-commit-msg.sample
| |___update.sample
|___info
| |___exclude
|___objects
| |___55
| | |___7db03de997c86a4a028e1ebd3a1ceb225be238
| |___info
| |___pack
|___refs
| |___heads
| |___tags
```

We can see that under the `objects` directory a new subdirectory has been created: `55` with an object of id `7db03de997c86a4a028e1ebd3a1ceb225be238` stored within, note that the first two characters of the SHA1 are omitted from the object files name.

As I said earlier, git stores every object against its computed SHA1, to make the directory structure a little simpler, git takes the first two characters of the SHA1 and uses it as the directory for the object.

If we recall, our "Hello World" strings computed SHA1 was `557db03de997c86a4a028e1ebd3a1ceb225be238` which means the directory named `55` should be our string "Hello World". We can verify this using the `cat-file` command.

[cat-file](https://git-scm.com/docs/git-cat-file) is a git plumming command that can display the content, type and size of objects within gits object store.

```
$ git cat-file 557db03de997c86a4a028e1ebd3a1ceb225be238 -p
Hello World
```

That's git at it's lowest level, you'll almost never use those commands directly in your day to day use of git. 

### Commits

Let's create some files and directories.

```shell
$ mkdir docs
$ touch README.md 
$ echo "Hello World" >> README.md
```

If we do a `git status` now we should see several untracked files.

```shell
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md
	docs/

nothing added to commit but untracked files present (use "git add" to track)
```

Let's fix that by staging and checking in our new files.

```shell
$ git add .
$ git commit -m 'Add README and Docs'
```

If we run a `git log` we should see our new commit with a SHA1 to represent it.

```shell
$ git log
commit b82828ed05ef076db09118994c7c036708973b40
Author: Jacob Clark <jacob.clark@>
Date:   Sat Mar 19 00:40:04 2016 +0000

    Add README and Docs
```

The SHA1 above `b82828ed05ef076db09118994c7c036708973b40` is simply a pointer to an object where git stores the information about that commit.

If we view the contents of the `.git` directory now, we should see several extra objects being stored for the files we have just committed. 

```shell 
$ tree .git
.git
|___branches
|___COMMIT_EDITMSG
|___config
|___description
|___HEAD
|___hooks
| |___applypatch-msg.sample
| |___commit-msg.sample
| |___post-update.sample
| |___pre-applypatch.sample
| |___pre-commit.sample
| |___pre-push.sample
| |___pre-rebase.sample
| |___prepare-commit-msg.sample
| |___update.sample
|___index
|___info
| |___exclude
|___logs
| |___HEAD
| |___refs
| | |___heads
| | | |___master
|___objects
| |___55
| | |___7db03de997c86a4a028e1ebd3a1ceb225be238
| |___66
| | |___801fba9ba0350874eb1f64b87b6cdb609da859
| |___76
| | |___ebb0f0fd6dfc00be4631fd7019d55536f38ea8
| |___b8
| | |___2828ed05ef076db09118994c7c036708973b40
| |___info
| |___pack
|___refs
| |___heads
| | |___master
| |___tags
```

We can see there are now four objects stored. One might think that is a little strange. We already had our "Hello World" object stored to begin with, we know that git stored our commit as an object and we just checked in three new objects (two text files and a directory), so that *should* leave us with 5 objects in the store, lets look at why that isn't the case.

Take a look at our commit object by running `git cat-file b82828ed05ef076db09118994c7c036708973b40 -p`.

The SHA1 of your commit will be different to mine as a timestamp is used to calculate it, so just substitute the one above for the one in your `git log`.

```shell
$ git cat-file b82828ed05ef076db09118994c7c036708973b40 -p
tree 66801fba9ba0350874eb1f64b87b6cdb609da859
author Jacob Clark <jacob.clark@> 1458348004 +0000
committer Jacob Clark <jacob.clark@> 1458348004 +0000

Add README and Docs
```

So here we can see git has stored a bunch of information about the commit we just performed, the tree SHA1, the author, committer and the message. 

Lets take a look at the tree object, again your SHA1 will be different.

```shell
$ git cat-file 66801fba9ba0350874eb1f64b87b6cdb609da859 -p
100644 blob 557db03de997c86a4a028e1ebd3a1ceb225be238	README.md
040000 tree 76ebb0f0fd6dfc00be4631fd7019d55536f38ea8	docs
``` 

A tree is like a directory, it holds several objects, in `.git` you're either looking at a tree or an object. A directory in your filesystem is a one to one mapping with a tree in git.

As I said right at the beginning of this article pretty much everything in git is a pointer. Within this commit we can see two pointers, one to an object and one to a tree. We can see both of these pointers corelate to exactly what we checked in earlier - a `README.md` file (now an object) and a directory named `docs` (now a tree).

The text inside our `README.md` file was simply "Hello World", this object was already stored in gits object store at the time we checked in our `README.md` file, and if we look at the SHA1 listed above we can see it is exactly the same as the SHA1 from the "Hello World" string we manually put into gits object store earlier. 

```shell
$ git cat-file 557db03de997c86a4a028e1ebd3a1ceb225be238 -p
Hello World
```

Git won't track anything it doesn't need too. Git is intelligent in knowing what it has tracked and what it hasn't, the SHA1 of two identical strings will never change. There was no reason for git to create a second object in the store for the exact same string and SHA1 that we checked in earlier. Instead git simply just created a pointer to it from the commit metadata we saw earlier. 

These types of optimisations is what keeps the performance of git high and alleviates unnecessary storage of objects taking up disk space which just are not needed, which is why we only saw 4 new objects and not 5 earlier.

### Branches

A branch is no different than a commit - it is simply a pointer to a commit within a repository (the HEAD). 

Git stores branch information within plain text files in the `.git/refs/heads` directory, git creates a branch by default called `master` which is represented by a file called `master` within this directory.

```shell
$ cat .git/refs/heads/master
b82828ed05ef076db09118994c7c036708973b40
```

This SHA1 is the pointer to the current location of the branch (the HEAD), if we run a `git log` we can see that the SHA1 above is exactly the same as the single commit we just created because that is our current location within that branch.

```shell
$ git log
commit b82828ed05ef076db09118994c7c036708973b40
Author: Jacob Clark <jacob.clark@g>
Date:   Sat Mar 19 00:40:04 2016 +0000

    Add README and Docs
```

If we create a new branch called `hello-moon` git will begin tracking the location of the branch within `.git/refs/heads/hello-moon`.

```shell
$ git checkout -b hello-moon
Switched to a new branch 'hello-moon'
$ cat .git/refs/heads/hello-moon
b82828ed05ef076db09118994c7c036708973b40
```

The above SHA1 is still the same as the `master` SHA1 because we haven't changed the location of the branch yet.

```shell
$ touch hello-moon.txt
$ git add .
$ git commit -m 'Add hello moon'
[hello-moon 683d9df] Add hello moon
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 hello-moon.txt
```

Now git has tracked the new commit and the new file we will now have a new location for the branch `hello-moon`.

```shell
$ cat .git/refs/heads/hello-moon
683d9df7e5c7247c4606c86e21c84f89e9a575bb
```

We can do a `git cat-file` on this SHA1 to see where this branch is now pointing too.

```shell
$ git cat-file 683d9df7e5c7247c4606c86e21c84f89e9a575bb -p                                                                                (hello-moon) 
tree 0a1b6bbf04e5d9448994cae4b5a2cb7eff14302a
parent b82828ed05ef076db09118994c7c036708973b40
author Jacob Clark <jacob.jh.clark@googlemail.com> 1458386031 +0000
committer Jacob Clark <jacob.jh.clark@googlemail.com> 1458386031 +0000

Add hello moon
```

As we expect - it's pointing to our 'Add hello moon' commit with one new addition, this commit has a parent, every commit in Git has a parent (unless it is the first commit on a branch), parents are how git tracks the history of its objects, if we `git cat-file` the parent we can see it is the initial commit we made on the master branch.

```shell
$ git cat-file b82828ed05ef076db09118994c7c036708973b40 -p                                                                                (hello-moon) 
tree 66801fba9ba0350874eb1f64b87b6cdb609da859
author Jacob Clark <jacob.jh.clark@googlemail.com> 1458348004 +0000
committer Jacob Clark <jacob.jh.clark@googlemail.com> 1458348004 +0000

Add README and Docs
```

If we decided on this branch we no longer want the 'Add hello moon' commit we could ask git to reset the branch back to a particular commit.

```
$ git reset --hard b82828ed05ef076db09118994c7c036708973b40
HEAD is now at b82828e Add README and Docs
```

Now if we cat the branch file one last time we will see we are back to where we first started.

```
$ cat .git/refs/heads/hello-moon                                                                                                          (hello-moon) 
b82828ed05ef076db09118994c7c036708973b40
```

In conclusion a branch is *literally* just a file which points to a particular commits SHA1, nothing more.

### Conclusion 

There are some topics that I have not covered in this post such as merges, tags, remotes and what HEAD actually is. I'll be covering these concepts and the above in more detail in a future post.

The [git documentation](https://git-scm.com/documentation) is fantastic and I highly encourage you to read the 'Git Internals - Plumbing and Porcelain' parts.

Visit my [website](https://www.jacobclark.xyz), follow me on [Twitter](https://twitter.com/imjacobclark) and [GitHub](https://github.com/imjacobclark) or view my professional background on [LinkedIn](https://uk.linkedin.com/in/imjacobclark).
