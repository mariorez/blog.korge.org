---
layout: post
title:  "Ported DragonBones to Kotlin and Korge"
author: soywiz
categories: [ Articles ]
image: assets/images/titles/ported-dragonbones-to-korge.jpg
---

DragonBones is a great free skeletal-based animation software
with a runtime implemented in several languages and engines.

I have spent several days this week converting DragonBones TypeScript+PIXI implementation to Kotlin+KorGE.
Also I have greatly improved KorGE in the process with things I wanted to do for some time now
and migrated to Kotlin 1.3-M1 with inline classes. I’ll talk about those in other post.

Korge already works fine in JVM and JS, and soon will be available in native targets too.
And it is flexible enough to support all the major modern platforms in a near future.

For now I’m focusing on cleaning up and fixing things for a 1.0 release focused on 2D game development.

With Dragonbones you can leverage Spine and Live2D too (as long as you have a proper license)
with the converters they provide: <https://github.com/DragonBones/Tools>.

The source code lives here:

<https://github.com/korlibs/korlibs/tree/master/korge/korge-dragonbones>

And a demo showing it working in JS:

<iframe src="https://samples.korge.soywiz.com/dragonbones/" style="border: 0px; height: 480px; width: 100%;" data-ss1593033426="1"></iframe>

<https://samples.korge.soywiz.com/dragonbones>

Still there are a lot of things to improve: performance and bugfixing.
For example, the eye tracking demo produces some strange vertices in JS
(though it works fine on JVM) and I still have to figure out why.


Vertices and skinning could also be computed at the GPU with the proper matrices.
But that will be done in the future transparently without changing the public API.