---
layout: post
title:  "Transparent Git Repository Encryption"
date:   2015-10-25 12:00:00
comments: true
categories: git
---

I want to use Github to host some journal entries, but I don't want everyone in the world to be able to read them.. and I don't have my Github account setup for private repositories. Furthermore, even if I did have a private repository, what's to stop the employees at Github from reading my jounral? Unacceptable.

The solution I've found is to use some military grade encryption and git filters to hook into git to encrypt files whenever they are staged, and to decrypt them whenever they are checked out. The result is that you can work in a local repository just like normal, but all the contents are encrypted in the remote repository. I used some of the scripts from [this][trans-encryption] gist, and fixed up some errors I was having and packaged some of the manual parts into some easier to use scripts. The resulting scripts and instructions for setting up your own remote repository can be found [here][elias-gitencrypt].

[trans-encryption]: https://gist.github.com/shadowhand/873637
[elias-gitencrypt]: https://github.com/eliasbagley/gitencrypt/
