---
layout: post
title:  "Presenting KorGE-3D"
author: soywiz
categories: [ Articles ]
image: assets/images/Screenshot-2019-03-17-at-06.39.29.png
---

Today we are happy to announce that we have released the first preview of KorGE-3D.

![](/assets/images/Screenshot-2019-03-17-at-06.39.29.png)

KorGE-3D is a new library built on top of all the korlibs libraries and that work seamlessly with plain KorGE to mix 2D
and 3D content easily.

It can describe 3D scenes with a DSL just like KorGE, it allows to add lights, models, materials, custom shaders and
skeletons with skinned meshes. It supports loading Collada (.dae) file format.

The code mixing 2D and 3D looks like this:

![](/assets/images/Screenshot-2019-03-17-at-06.39.55.png)

Note that it is only an experimental preview. The API is likely to evolve, and there are known bugs and problems, and of
course it is not optimized at all.

This preview doesn’t support shadow projections, it doesn’t yet provide APIs for raytracing or physics, and lacks a lot
of other stuff. But still can be used for some simple things.

Along KorGE-3D I have released KorGE 1.2.0 supporting it:

* Changelog: [https://korlibs.soywiz.com/korge/changelog/#v120](https://korlibs.soywiz.com/korge/changelog/#v120)
* Live
  Demo: [https://korge.soywiz.com/samples/korge-3d/index.html](https://korge.soywiz.com/samples/korge-3d/index.html)
* Source Code of the
  sample: [https://github.com/korlibs/korge-samples/tree/master/sample-3d](https://github.com/korlibs/korge-samples/tree/master/sample-3d)

Any feedback, bug report, or help is greatly welcome! KorGE-3D is part of the main KorGE repository and lives here:

* [https://github.com/korlibs/korge/tree/master/korge-3d](https://github.com/korlibs/korge/tree/master/korge-3d)
