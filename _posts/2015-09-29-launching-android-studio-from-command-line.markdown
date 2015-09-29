---
layout: post
title:  "Launching Android Studio from the Terminal"
date:   2015-09-29 13:35:00
categories: Android
---

I've been frustrated with Android Studio's lack of a project file which can be quickly opened. When I'm starting the day workin on an iOS project, the first thing I do is [jump][jump-link] to the project's directory in the terminal, then type `open .xcwork<TAB><CR>`, and XCode will open the current project. With Android Studio, there isn't an obvious way to launch it from the terminal like that, since there is no project file that can be opened. First I have launch Android Studio, and then select the project I want from the GUI. Not a huge deal, but annoying enough that I wrote a script to launch the current directory's Android project from the command line

{% highlight sh %}

#! /bin/bash
/Applications/Android\ Studio.app/Contents/MacOS/studio $(pwd) > /dev/null 2>&1 &

{% endhighlight %}

This uses Android Studio's executable to launch the current directory's project, runs it in the background, and directs all the output to `/dev/null` so it doesn't clutter up the screen.

Name that script whatever you want, and put it somewhere on your path. I named mine `openas`. So now I can jump to the project directory in the command line, and type `openas` and Android Studio will open in the context of the current project.

I put this script in `~/bin` (create it if it doesn't exist) and then added `~/bin` to the path so I can execute `openas` from any directory.


[jump-link]: https://github.com/clvv/fasd
