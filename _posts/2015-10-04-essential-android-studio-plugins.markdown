---
layout: post
title:  "9 Essential Android Studio Plugins. #4 Will Shock You!"
date:   2015-10-04 12:00:00
comments: true
categories: Android
---

## 1. [IdeaVIM][ideavim-plugin-link]<br>

IdeaVIM brings vim key bindings to Android - a must have for vim lovers! Although not quite as powerful as the real vim, this plugin turns an otherwise intolerable editing experience into a joy. If you're not a vim user, set aside some time to learn it and watch your editing speed skyrocket. Read [this][vim-beginner-link] and then [this][grok-vi-link] for a taste of the power of vim. Congratulations, you are now an initiate of the `Cult of Vim`.

## 2. [Otto][otto-plugin-link]<br>

If you're using the wonderful [Otto][otto-link] event bus (and you should be), you'll know that the decoupling of posting and consuming events can be a double edged sword, since it is frequently difficult to find where an event is being posted or consumed during development. The Otto plugin adds an icon on the left to every `post`,  `produce`, and `subscribe` call, which you can click on to quickly jump between where events are posted and where events are consumed.

## 3. [Butterknife Zelenzy][butterknife-plugin-link]<br>

If you're using [Butterknife][butterknife-link] for view injection (and you should be), the ButterKnife plugin allows for quickly generating view injection code.

## 4. [AceJump][acejump-link]   <br>

This is text file navigation on meth and steroids. Type `ctrl + ;`, then type a character, and all of those characters will be highlighted on the screen, allowing you to quickly jump to them. This is basically the vim plugin [EasyMotion][easymotion-link] for Android Studio. One of my biggest complains about the IdeaVIM plugin is that althought it allows basic vim bindings, it doesn't allow all the power of vim that I'm used to from VIM plugins, specifically EasyMotion. AceJump mostly fixes that.

## 5. [Genymotion][genymotion-link]<br>

Tired of waiting 5 minutes for the Android emulator to start? Wait no more with Genymotion! A breeze to setup and use, and a great way to test on multiple API levels and devices.

## 6. [Mirror][mirror-link]<br>

Mirror allows you to make changes to you UI and see it updated instantly, rather than having to build and run for each tiny visual tweak you make. This is a great way enhance your productivity by not having to wait for Android's slow build times.

## 7. [Key Promoter][key-promoter]<br>

In order to be productive as possible, you have to eliminate every distraction and source of friction which can slow you down. Moving your hands off the keyboard to click on a toolbar button is one of those sources of friction. Key Promoter will alert you whenever you make the carindal sin of clicking on a toolbar too many times, and will remind you of the keyboard shortcut or prompt you to set one.

## 8. [Code Glance][code-glance-link]<br>

Not essential, but fun and useful. Code Glance gives you a 'minimap' of the topology of your whole source file in the top right corner. This can be useful for spotting callback hell and deep nesting, which are frequently code smells. Similar to the minimap found in the Sublime editor.

## 9. [Relative Line Numbers][rln-link]<br>

Controversial, but I like it. It pairs nicely with vim key bindings, allowing you to see how many lines above or below the cursor you need to navigate.




<br>
Know of any Android Studio plugins that I missed? Add them in the comments!

<br>
<b>Edit:</b>

Some nice folks on the [reddit discussion][reddit-link] have suggested a few more useful plugins:

## 10. [ADB Idea][adb-idea-link]<br>

ADB Idea plugin has options to uninstall the app, kill the app, clear the cache, and more with a single hotkey. Very useful if you're debugging a feature that deals with persisting data to disk and you need to clear it out after each run.

## 11. [Kotlin][kotlin-link]<br>

This one isn't essential, but it's awesome. Kotlin is a JVM language replacement for Java, created by JetBrains, the makers of Android Studio. Kotlin has a low learning curve, small memory footprint, some slick syntax, closures, excellent IDE integration, and it's a breeze to start using in your Android apps. I forsee Kotlin becoming a major contender to Java for Android development.

## 12. [Anko][anko-link]<br>

Another gem by JetBrains. If you're using Kotlin, Anko allows you to build user interfaces in code with a declarative DSL instead of XML.

[otto-plugin-link]: https://github.com/square/otto-intellij-plugin
[ideavim-plugin-link]: https://github.com/JetBrains/ideavim
[otto-link]: https://github.com/square/otto
[butterknife-link]: https://github.com/JakeWharton/butterknife
[butterknife-plugin-link]: https://github.com/avast/android-butterknife-zelezny
[easymotion-link]: https://github.com/easymotion/vim-easymotion
[acejump-link]: https://plugins.jetbrains.com/plugin/7086?pr=
[genymotion-link]: https://www.genymotion.com/
[mirror-link]: http://jimulabs.com/
[key-promoter]: https://plugins.jetbrains.com/plugin/4455?pr=clion
[code-glance-link]: https://plugins.jetbrains.com/plugin/7275?pr=clion
[rln-link]: https://plugins.jetbrains.com/plugin/7414?pr=clion
[vim-beginner-link]: https://danielmiessler.com/study/vim/
[grok-vi-link]: http://stackoverflow.com/questions/1218390/what-is-your-most-productive-shortcut-with-vim/1220118#1220118
[adb-idea-link]: https://github.com/pbreault/adb-idea
[reddit-link]: https://www.reddit.com/r/androiddev/comments/3nfk8m/9_essential_android_studio_plugins/
[anko-link]: https://github.com/JetBrains/anko
[kotlin-link]: http://kotlinlang.org/
