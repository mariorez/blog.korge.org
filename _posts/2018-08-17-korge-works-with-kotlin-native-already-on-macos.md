---
layout: post
title:  "Korge/Korlibs already works with Kotlin-Native on MacOS"
author: soywiz
categories: [ Articles ]
image: assets/images/titles/korge-works-with-kotlin-native-already-on-macos.jpg
---

After some tries and several bug reports, Korge already works on Kotlin/Native.I have uploaded an `.app` file here
showing the
results.[https://soywiz.com/content/images/demos/korge-dragonbones-demo-mac.app.7z](https://soywiz.com/content/images/demos/korge-dragonbones-demo-mac.app.7z)For
the icon I have used a WIP ilustration that is finishing my
wife.Edit: [Final version of the image for mmo-poc](https://github.com/mmo-poc/mmo-poc/blob/ccc0c2ba2a9444ea32f8724fe0f5464f857ab4f7/mmo/common/resources/chara/pricesa5.png).The
sample is the same as the other article that worked on JS and JVM, but this time in native for macOS without additional
dependencies or frameworks.It includes three dragonbones samples: one with simple skeletal animation, other imported
from live2d that follows the cursor and has tons of computations, and other showing slot replacement from an animation
imported from Spine.The only difference is that text in buttons is not shown, and the edges of the buttons are not
antialiased. The K/N version uses the integrated software-based rasterizer of KorIM. This rasterizer is not optimized,
and do not perform antialiasing yet. Though it should work with any shape, gradients and bitmap fills. I have not
implemented native fonts on KorIM for macOS, so fonts are not shown. Though you should be able to use TTF fonts or
bitmap fonts as required already. Probably I will embed a TTF font or small bitmap font as default font so it works in
all targets consistently.

## Performance journey

Right now, the demo performance is starting to get similar to the Java version. Except for JSON loading. Note that I
have manually disabled bound checkings and freezing checkings to achieve this.Also please note that Kotlin/Native is not
even 1.0 yet, so no heavy work has been done for performance just yet.

### Allocating is expensive at this point

JS and JVM do all kind of tricks to do cheap allocations in an insane way. Starting from allocation memory in fixed
chunks and some other tricks. You can check [nedmalloc](http://www.nedprod.com/programs/portable/nedmalloc/)
and [tcmalloc](http://goog-perftools.sourceforge.net/doc/tcmalloc.html) for details. Maybe they can even group together
in memory frequently used objects to improve the data cache usage of the processor. JS and JVM can also totally prevent
heap allocations in some cases and allocate objects in the stack if they can know for sure that they are not used
somewhere else. Also kotlin-native right now expend some additional time doing reference counting. That also helps to
collect memory faster and prevent the heap from growing too fast.(The demo I have created starts with 30MB of memory
only including the images from the first demo and starts immediately which is super nice.)All these tricks require some
time to implement, and kotlin-native is still in the phase of fixing bugs and refining things. But I’m confident it will
get much better in future versions.

### Boundchecking and freeze checking overhead

I have not benchmarked it yet, but disabling both checks makes the application faster. The boundchecking can be
optimized by LLVM (though not always work as expected yet), but the freeze seems more hard to optimize by LLVM without
additional help from outside. The freeze checks only help when debugging the application to detect potential data-race
errors. So I hope there will be a way to disable those checks for final release builds. Since those checks are not doing
anything useful once you get it right.I would love to see something
like `--unsafe-disable-freeze-checks` `--unsafe-disable-boundchecking` to totally get the best performance possible and
to be able to compare the overhead caused from this. Maybe even being able to disable some of those checks per function
with an annotation `@UnsafeDisableFreezeCheck` and `@UnsafeDisableBoundChecking`. If the compiler use intrinsics it
could select one array getting function or another.In addition to the overhead, halting in each iteration of an array to
check freezing or bounds, totally prevent optimizations like vectorization.In the future would be nice to be able to
reduce those checks if possible. Specially to move them outside loops. But also if a function is called millions of
times per frame, would be nice if it doesn’t try to check for freeze on each one.

### Some standard lowering are still not being done

For example, the IDE suggests you to change `a >= 0 && a < 10` into `a in 0 until 10`. Both JVM and JS lowers this into
a simple couple of comparisons.Another example is `for (n in listOf(1, 2, 3, 4))`. By default when iterating an
array/list, it creates an iterator. But in fact, arrays and lists have random access, and a size so you can convert them
into: `for (n in 0 until array.size) { val item = array[n]; ... }`I initially thought that all those transformations
would be done by the compiler in a standard way that worked for all targets. But somehow it seems that K/N has its own
IR lowerings. So right now it optimizes for + ranges, but not other standard stuff.Also for a `for (n in 0 until 10)`
K/N is consuming and transforming something like `var iterator = IntRange(0, 10); while (iterator.hasNext()) { ... }`.
Which is pretty hard to optimize and slow to peephole it. I would initially thought that the for + range optimization
would be done with the AST IR when it was already a for.So it seems that right now each target does its own
lowerings.For JTransc for example I did a different approach. There was a common AST-based IR with very few stuff. And
then an extended IR with more things (for example lambdas).By default each target, just supports the base IR, so when
the code generator for each target gets the AST, it gets the base IR and just have to know how to generate very simple
stuff. But then, each target have a whitelist of supported “features”.For example, in this target I support “lambdas”.
In a common place, all the lowering happens. It gets all the supported features by that version of JTransc, and gets the
target supported features, then subtracts it, and lowers all the features not supported by the generator. So when adding
a new feature to JTransc, all the generators continued working witout changes, and then I can optimize specific cases by
marking specific features as supported.
