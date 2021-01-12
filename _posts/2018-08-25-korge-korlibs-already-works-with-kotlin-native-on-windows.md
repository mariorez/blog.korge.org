---
layout: post
title:  "Korge/Korlibs already works with Kotlin-Native on Windows"
author: soywiz
categories: [ Articles ]
image: assets/images/titles/korge-korlibs-already-works-with-kotlin-native-on-windows.jpg
---

Last Week Korge started to work with Kotlin/Native on MacOS target. And of course, it also run on the JVM and the
browser. Today, it starts to work on Windows x64 too.

Korge for windows uses the Win32 and Opengl32 APIs (and MSVCRT used by MingW but ships with windows since Windows 95
OSR2), thus does’t have any external dependencies or DLLs required like SDL or similar.

The executable size (without assets) is about ~7.3MB uncompressed, and ~1.2MB compressed as .7z including all the
Korlibs required to run Korge with dragonbones.

![](/assets/images/korge-dragonbones-win64-1.jpg)

![](/assets/images/2018-08-25--10-.png)

![](/assets/images/2018-08-25--15-.png)

Download sample ~6 MB

Performance note: This time I have not recompiled Kotlin/Native without freezing checks and bound-checking removal, so
the performance is not as good as the MacOS vesion.

The source-code is available at:

[https://github.com/korlibs/korlibs](https://github.com/korlibs/korlibs)

And the specific commit supporting win32 in korui (fake, since right now it doesn’t allow to create components in
addition to the main windows like the JS and the JVM targets).

[https://github.com/korlibs/korlibs/commit/84ff61a92b0b529069c469efa666fad5482b013f](https://github.com/korlibs/korlibs/commit/84ff61a92b0b529069c469efa666fad5482b013f)

The most important file supporting this: KoruiEventLoopNative.kt

The demo running on the browser using Kotlin.JS:

<iframe src="https://samples.korge.soywiz.com/dragonbones/" style="border: 0; height: 480px; width: 100%;" data-ss1593033752="1"></iframe></div>
