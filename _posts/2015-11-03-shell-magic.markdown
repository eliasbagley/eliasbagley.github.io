---
layout: post
title:  "Shell Magic: A Compilation of Advanced Shell Commands"
date:   2015-11-02 12:00:00
comments: true
categories: shell
---

A list of useful shell commands that I don't want to forget. This list will be continuously updated

# Git stuff

## Delete all files that were deleted by the other branch in a merge conflict

`git status | grep 'deleted by them' | awk '{b=$4" "$5" "$6; printf "%s\0", b}' | xargs -0 git rm`

The $4 $5 and $5 were due to the fact that the `git status` output had spaces in the filename. Adjust these for the `git status` output that you get.
