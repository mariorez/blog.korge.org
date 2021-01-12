---
layout: post
title: "KorGE 2.0 feature list preview"
author: soywiz
categories: [ Preview ]
image: assets/images/titles/korge-2-0-feature-preview.jpg
---

We are happy to announce that we started sketching the roadmap for KorGE version 2.0\. Together with fixing all currently known bugs and a code cleanup including the removal of deprecated sources, these features are planned for the new milestone:

* Full Kotlin 1.4 support
* Better camera support and more camera features like following or centering on a view
* New and better UI elements including a completely reworked Text view and editable text fields
* More tightly integrated Box2D experience in line with the existing KorGE api
* A hardware accelerated container for sprites and animations which should tremendously boost the performance, although KorGE already has no problem, animating [10.000 sprites at once](https://github.com/korlibs/korge-samples/tree/master/sprites10k)
* Apple Metal backend
* Spine support
* Full support of TMX format specification
* A refined Tilemap editor built in IntelliJ
* Support for more tilemap formats and features
* A debug overlay showing hitShapes, fps and additional debugging info
* Autocompletion of resource paths

These features are of course subject to change, but we will work hard to implement all of them.

If there are any features you want to suggest, do not hesitate to contact us via forum or slack.

Right now we are doing all the work on master, and we make all releases on it. This is something that complicates being able to target several Kotlin versions or to prepare a new major version while maintaining the stable one.

So our first work on this direction is to update our CI to be able to deploy different branches by using tags. And then we will try to release all the libraries using the work in progress milestone of Kotlin 1.4-M2\. And we will keep our code updated to the latest version until the final release.
