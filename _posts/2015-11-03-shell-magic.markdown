---
layout: post
title:  "Shell Magic: A Compilation of Advanced Shell Commands"
date:   2015-11-02 12:00:00
comments: true
categories: shell
---

A list of useful shell commands that I don't want to forget. This list will be continuously updated

## Git stuff

# Delete all files that were deleted by the other branch in a merge conflict

`git status | grep 'deleted by them' | awk '{b=$4" "$5" "$6; printf "%s\0", b}' | xargs -0 git rm`

The `$4` `$5` and `$5` were due to the fact that the `git status` output had spaces in the filename. Adjust these for the `git status` output that you get.

## Filesystem stuff

# Show diskspace used of current directory in human readable form

`du -sh .`

Easy to remember since its pronouncable as "douche", as in _who's the douche using up all the disk space_


## Android

# Get SHA1 hash of keystore for Google Developers Console

# Update SDK from terminal

By default, the SDK Manager from the command line does not include the build tools in the list. They're in the "obsolete" category. To see all available downloads available, use

`android list sdk --all`

And then to get one of the packages in that list from the command line, use:

`android update sdk -u -a -t <package no.>`

Where -u stands for --no-ui, -a stands for --all and -t stands for --filter.

If you need to install multiple packages do:

`android update sdk -u -a -t 1,2,3,4,..,n`

Where 1,2,..,n is the package number listed with the list command above

