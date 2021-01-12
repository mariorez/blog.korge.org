---
layout: post
title: "Why choose KorGE over other alternatives?"
author: soywiz
categories: [ Articles ]
image: assets/images/2340457563.jpg
---

You might wonder why should you decide to use KorGE instead of other Game Engines.  
Compared to some other engines, KorGE itself has been less time in the market, and it has still a lot of room for
improvement. But it already has some great features you can consider.

## Price

KorGE is completely free and opensource. You can use it to create commercial games without having to pay anything: nor
variable or fixed rates.

## Open

KorGE and all its subprojects are fully opensource. That means that you can check the source code, propose changes and
improvements and troubleshoot any problem you might find.

## Native

Other engines might use C++, C#, interpreted languages and they either interpret your game logic or end compiling it to
C++ and transpiling into something else. KorGE instead uses Kotlin Multiplatform that generates truly native code to
each platform. In desktop or iOS it generates native code, in Android it generates DEX java-like code, and in the
browser it generates plain JavaScript.

## Dependency-less

The engine allows to create executables that do not have external dependencies other than the ones provided by the
platform itself. So no extra DLLs, .frameworks or .so files required for KorGE games.

## Small

By being dependencyless and using Kotlin, KorGE allows to create native executables that are really small compared with
other engines.You can create native compressed executables that fit in an old 1.4MB magnetic diskette, and compressed
.js bundles in less than half a megabyte.

## Cohesive

KorGE is not just the project itself, but a set of reusable multiplatform libraries called Korlibs. These libraries
cover lots of areas and allows the engine use them in a pleasant way that feels nice and cohesive.

## Productive

Kotlin is by far one of the most productive languages out there. Starting with its syntax, its features, and all the
IntelliJ platform tooling, including intents. With KorGE you have all that productivity power by your side.

## Asynchronous

KorGE is fully asynchronous and embrace Kotlin coroutines. Other languages support async/await constructs, but the
Kotlin one is simpler and support cancellation out of the box without the need of having to propagate cancellation
tokens everywhere, or needed to place await everywhere.

## Usable as library

Even when KorGE supports creating executables out of the box by using its gradle plugin, it is also just a Multiplatform
Kotlin library that can be referenced and used from another application. In fact, some people already embedded KorGE in
their own Android applications.

## Testable

KorGE supports headless testing, and view testing so you can test your code easily. In addition, you can use the CI of
your choice supporting gradle to automatically test your code in CI platforms.

## Tooling

KorGE provides a gradle plugin that allows building and testing all the supported platforms. It also provides an
IntelliJ-based plugin that adds project templates and visual editors for some of the supported formats.

## Potential

Kotlin, Kotlin/Native and the JVM and are evolving. JVM's Project Valhalla and GraalVM will enable potential new
optimizations and to leverage SIMD instructions and Kotlin/Native is in continuous development. KorGE is fast enough for
lots of cases, but will be even faster in the future.
