---
layout: post
title:  "Korge 1.0 Released! 🎉🎉🎊 Modern Multiplatform #gameengine for #kotlin"
author: soywiz
categories: [ Showcases ]
image: assets/images/1.png
featured: true
---

After two years of development, I have finally released the first version of KorGE, a Modern Multiplatform Game Engine fully written in Kotlin.It can produce small native applications, and allows to write the games in Kotlin Common. It provides a gradle plugin with tasks to target to JVM, JS Browser and Cordova, Native Desktop, Native Android and Native iOS.

## Landing page: [https://korge.soywiz.com/](https://korge.soywiz.com/)

Additional links:

*   Hello World and Template: [https://github.com/korlibs/korge-hello-world](https://github.com/korlibs/korge-hello-world)
*   Additional samples (WIP): [https://github.com/korlibs/korge-samples](https://github.com/korlibs/korge-samples)
*   Forum: [https://forum.soywiz.com/category/14/korge](https://forum.soywiz.com/category/14/korge)
*   Documentation (WIP): [https://korlibs.soywiz.com/korge](https://korlibs.soywiz.com/korge)

Tutorials will be available soon.Along KorGE I have released a set of multiplatform independent libraries that are used by KorGE, but that you can use independently in your Kotlin Common projects or in any supported Kotlin target:

*   [Klock](https://github.com/korlibs/klock) - Date and Time utilities
*   [Kmem](https://github.com/korlibs/kmem) - Bits, Arrays and fast memory utilities
*   [Kds](https://github.com/korlibs/kds) - Advanced Data Structures and optimized versions without Boxing
*   [Krypto](https://github.com/korlibs/krypto) - Crypto library (SecureRandom, AES, MD5, SHA1 y SHA256)
*   [Kbox2d](https://github.com/korlibs/kbox2d) - Box2D Port
*   [Korio](https://github.com/korlibs/korio) - Virtual File Systems, synchronous and asynchronous streams, http client, websockets, compression (deflate and lzma), asybchronous utilities, common dynamic access for JVM and JS, and much more.
*   [Korim](https://github.com/korlibs/korim) - Native image loading and in pure kotlin, color management and color formats, image manipulation, QR-code generation, native vectorial rendering, native fonts and TTF y much more.
*   [Korma](https://github.com/korlibs/korma) - Math utils + geometry, affine transforms, Matrix, Matrix3D, points, rects, transforms, bezier curves, vector descriptors, triangulation, pathfinding A* y TriA* with funnel support, unión, intersection, xor, etc. of vector and triangles.
*   [Korau](https://github.com/korlibs/korau) - Load and play of sounds and music (mp3, ogg etc.)
*   [Korgw/Kgl/Korag](https://github.com/korlibs/korui) - Similar to GLUT. Utility to create window with accelerated graphics. Abstract interface for shaders and rendering with an initial implementation using opengl and webgl. There will be future implementations using metal and vulkan.
*   [Klogger](https://github.com/korlibs/klogger) - Logger for Kotlin
*   [KorGE](https://github.com/korlibs/korge) - Engine for games with initial native support for 2D (supports 3D but doesn’t provide high level utilities), and will also provide 3D utilities in the future.
*   [KTCC](https://github.com/soywiz/ktcc) - C Compiler and small web-based IDE fully written in Kotlin that generates Kotlin Code (Ideally to port Libraries written in C)
