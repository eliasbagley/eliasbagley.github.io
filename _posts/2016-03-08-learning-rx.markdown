---
layout: post
title:  "A Roadmap for Learning Reactive Extensions"
date:   2016-03-08 12:00:00
comments: true
categories: rx
---

A roadmap for learning RX:

I originally typed this up as a slack storm to my coworkers, and figured it might be useful to live on as a blog post I can point peoplt to.


Rx (Reactive Extensions) is an abstraction of computation using functional programming techniques. There are implementations of Rx in most major languages.

Read these first for an introduction:

https://gist.github.com/staltz/868e7e9bc2a7b8c1f754 - Introduction to Reactive Programming
http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/ - 4 part series, and uses RxJava as the implementation, but the concepts should be pretty much directly translateable to RxSwift
http://reactivex.io/documentation/observable.html - the Reactivex documentation for when you want to understand a single class or operator better. The marble diagrams are super helpful when you're trying to wrap your head around what an operator does.

The main things it's important to become familiar with are the Observable, Subscriber, and Subscription, and a handful of common operators. There are dozens of operators, but the ones I use most often in order are are: map, observeOn, scheduleOn, toBlocking, first, filter, reduce, debounce, combineLatest, zip.

About interfacing with non-rx code: calling .toBlocking().first() on an observable is a great way to grab a value of of the stream, without having to subscribe to it.

Another great way to interface with non-rx code is to use a Subject, which acts as both a Subscriber and an Observable. This is the easiest way to create an rx wrapper for something. The wrapping class creates a Subject which it can put values into however it wants, and exposes some sort of `getAsObservable()` method, and the consuming class calls the `getAsObservable()` method and can treat it like any other Observable

Streams have a concept of a "scheduler", which is just an operator that you can add to the chain and it causes operations that flow through that to be applied to whatever thread you want. So something like this:


 ```java
myObservable
.map(value -> someOtherValue)
.observeOn(Schedulers.newThread())
.map(some_computation_heavy_operation())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(... )
```

This is big part of the reason why people love RX. It turns something hard like concurrency into something pretty trivial.

Common problems I've run into with RxJava - forgetting to to unsubscribe a subscription, which causes a memory leak. The solution to this is to put a `CompositeSubscription` into a base class, and add all subscriptions to that, and to unsubscribe from the composite subscription when the class is destroyed. CompositeSubscription just calls unsubscribe on every subscription in it when it receives an unsubscribe call.

The aha! moment for me happened once I understood `flatMap` and how to use it to avoid a bunch of nested callbacks
Slightly more advanced related reading: http://blog.danlew.net/2015/03/02/dont-break-the-chain/


Another essential way to interface with non-rx code is to use `Observable.just()` and `Observable.from()`. `Observable.just(some_value)` will create an Observable that emits a single value, and then completes. `Obsevable.from(some_list)` will create an observable that emits each item into a collection, and then completes. I use these all the time to lift regular code into a stream

Once you understand the basics and want to get into more advanced territory, read a little bit about hot vs cold observables, and backpressure. Backpressure means that whoever is putting the events into the stream is putting them in faster than whoever is subscribing the stream can read them. So there is a "buildup" of events in the queue, hence backpressure. There are a complete techniques to fix this, but honestly I've never had to use any yet since I haven't run into that problem

A "hot" observable is one that is always emitting events. You could thing of a stream of notifications being an example of a hot observable. A "cold" observable is one that only starts emitting events when subscribed to. An example is something like `Observable.from(1, 2, 3, 4, 5,)` which just emits 1 2 3 4 5 once it's subscribed to, and then completes.

