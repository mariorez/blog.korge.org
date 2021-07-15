---
layout: post
title: "KorGE 1.12.8.0: Huge Optimizations!"
author: soywiz
categories: [ Releases ]
image: assets/images/titles/korge-11290-optimizations.jpg
featured: false
---

There is a new version of KorGE available now!

This version includes a lot of optimizations done to make all the targets faster, specially on Kotlin/Native.

We have rewritten the view's components system to be much faster and we have done tons of optimizations to reduce
overhead of Kotlin/Native.

Before this version, the 10.000 sprites sample Nico Emig did, was terribly slow on Kotlin/Native. After profiling a bit
we found a few places worth optimizing, and with a few changes in all the libraries, we got it working much faster. It
is still not as fast as the JVM counterpart, but much closer than before.

Also a few versions ago, the Kotlin/JS optimization for instance checkings stopped from being applied due to a slight
change in the output of the kotlin.js runtime. This version enables it again, making the JS target much faster than
before.

After the profiling, we noticed that moving the mouse in the 10K sample caused the game from pause for a long time. This
was the components system trying to propagate the mouse event to all those views. We have done a lots of optimizations
here, and now that pause has totally gone away.

So far, this is the fastest version of KorGE to the date! Go ahead and try it. Just use 1.12.8.0 as your
korge-gradle-plugin version.
