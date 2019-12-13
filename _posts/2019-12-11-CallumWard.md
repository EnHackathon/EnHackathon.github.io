---
layout: post
title:  "Writing Native Apps in Python - Contributing to Beeware"
date:   2019-12-11
author: Callum Ward
---

After having taken a stab at fixing a couple of issues within the main `CPython`
implementation of Python, I decided whilst waiting for some of those pull
requests to be merged I might try to contribute to some other open source
projects in the Python ecosystem that are endorsed by the PSF.

Lewis suggested taking a look at [Beeware](https://beeware.org/), which recently
received a grant from the PSF Education group to improve support for Python on
Android, as part of their mission to enable cross-platform Python-based native
apps.

## Getting Started

There's a lot of pieces to Beeware that are in early stages of development and
support, and should be ripe for further contribution. However, the first stages
revolved primarily around familiarising myself with the ecosystem to understand
which pieces would be most suitable.

Beeware has several constituent projects, some of the relevant ones are:

- [`toga`](https://github.com/beeware/toga): a cross-platform native widget
  toolkit, that provides a high-level API wrapping the concept of an `App`,
  allowing users to build GUI applications which resolve to using native widgets
  on the target platform

- [`colosseum`](https://github.com/beeware/colosseum): an implementation of the
  CSS specification for resolving positions and locations of elements on
  a canvas, used by `toga` for styling applications

- [`voc`](https://github.com/beeware/voc): a transpiler which converts Python
  bytecode allowing Python to be compiled into Java bytecode and run on the JVM,
  enabling Android native apps

- [`batavia`](https://github.com/beeware/batavia): an implementation of the
  Python virtual machine in Javascript, enabling Python bytecode to run on the
  browser or over `node.js`

The one which is most obviously approachable starting from a background of just
Python (as opposed to native widget libraries like Cocoa, GTK+, or Java and
Javascript) is `colosseum`, which is a complex problem but well-defined, and
comes with a suite of (currently failing) tests which verify it meets the CSS
specification.

## Attempting to Contribute

As suggested on the `colosseum` [contributing
guide](https://colosseum.readthedocs.io/en/latest/how-to/contribute.html),
I selected a test (at random) from the list of "known failures" and removed it.
Re-running the tests, I took a look at what exactly fails as part of the test.

What I found was not particularly amenable to a contribution: it appears that an
implementation of an object expects a certain class which depending on the type
of the object would contain a certain field. But for the main `layout` API, this
object was always being passed in as `None`, seemingly because that whole class
of objects hadn't been implemented:

```python
def layout(display, node, standard=HTML5):
    containing_block = Viewport(display, node)
    font = None  # FIXME - default font

    node.layout.reset()

    # 10.1 1
    layout_box(display, node, containing_block, containing_block, font)
```

Note the `FIXME`. It appears that a lot of the current issue with `colosseum` is
simply missing functionality: code which has been written in a certain way
(knowing it will be needed to meet the CSS spec) but is being fed by defaults
and stubs.

This represents both a difficulty and an opportunity: once the requirements are
understood, and the problem fleshed out (probably via interacting with the core
devs) there is certainly clear work to be done which just requires skills
already available to our team, designing and coding Python.

After talking to some of the core devs on the
[`gitter`](https://gitter.im/beeware/general), I'm somewhat hopeful that they'll
be able to direct us at some work to be done, which might represent good group
projects for further contribution in the New Year.
