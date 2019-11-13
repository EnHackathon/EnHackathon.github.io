---
layout: post
title:  "Visiting the C-side"
date:   2019-11-12
author: Lewis Gaul
email:  lewis.gaul@gmail.com
---


What a welcome return to the office after visiting the seaside over the weekend!
The second day of EnHackathon felt much more organised than the first, which
hopefully helped everyone to able to make progress in their area of focus.
We were also graced with the presence of a couple of new contributors, bringing
our group up to 9 people.


## Subinterpreters Continued

I spent the day continuing progress on the two subinterpreter issues I was
looking at [last time](../04/LewisGaul.html). It was great to have Phil on board
looking into the other issues I mentioned last time (check out
[his blog post](PhilConnell.html)), and also great to have Ben helping me out
towards the end of the day.


### 'Tracing Possible' Field

<https://github.com/ericsnowcurrently/multi-core-python/issues/32>  

This issue suggests copying a `tracing_possible` field from the CPython runtime
state struct to the interpreter state struct.

It seems this field records how many threads have tracing enabled, where
tracing is the mechanism used by tools such as PDB and code coverage analysers.
If the value is zero, this allows the code execution to follow the 'fast path'.

It's still not clear to me what the full scope of the required change is, but
I've put my initial attempt up as a
[PR](https://github.com/LewisGaul/cpython/pull/1/files) within my cpython repo.
I'm also unsure how this ties in with
[previous attempts](https://github.com/python/cpython/commit/ef4ac967e2f3a9a18330cc6abe14adb4bc3d0465)
to move fields from the runtime ceval struct to a per-interpreter ceval struct...
I'm at the point it would be good to get some more input from a maintainer of
the project.


### New 'List Channel Interpreters' API

<https://github.com/ericsnowcurrently/multi-core-python/issues/52>  

As far as I can tell, this issue tracks the addition of an API on the internal
`_interpreters` module (Modules/_xxsubinterpretersmodule.c) to list interpreters
associated with [an end of?] a channel.

Looking at the section detailing the API for the new `interpreters` module in
[PEP 554](https://www.python.org/dev/peps/pep-0554/#interpreters-module-api),
it seems that the `RecvChannel` and `SendChannel` should have an `interpreters`
property, which should return `Interpreter` objects (as opposed to interpreter
IDs). The logic of associating channels with interpreters seems to have already
been implemented in the internal module, so this task should just be a matter of
extracting the association. However, the `Interpreter` type is defined in the
external module, so as far as I can tell it makes most sense for the internal
module to return interpreter *IDs*, and for the external module that wraps the
internal module to convert the IDs into `Interpreter` objects.

I've managed to more-or-less implement a proof-of-concept for the internal
`channel_list_interpreters()` API (thanks to Ben for his help fixing my broken C
code), with the main thing left to sort out being how to handle the distinction
between send and receive channels (maybe just a boolean argument?). I've created
a [PR](https://github.com/LewisGaul/cpython/pull/3/files) in my cpython repo to
track my progress with this and hopefully make it easier for any maintainers to
give me feedback.


### Summary

I'm definitely at a point where some input from a maintainer would be good. Phil
managed to complete the open issue about forking, which leaves us with one open
issue that I was recommended to take a look at by Eric. There is now interest in
the subinterpreters project from 4 of us taking part in EnHackathon, so it would
be good to work out what we should look into ahead of the next day of
contribution.

I've continued to find the subinterpreter project interesting to work on, as it
involves very general CPython skills/knowledge such as: getting familiar with
the CPython runtime, implementing a Python module in C and interfacing with a
pure Python module, and debugging CPython code at the C level. I'm hoping that
as I increase my knowledge around this project it will become easier to
understand the big picture more, and therefore easier to dive in with the
changes that interest me the most.

Some more references (docs for the C API) that I found useful this time were:
- <https://docs.python.org/3/c-api/intro.html#objects-types-and-reference-counts> -
Intro to objects, types and reference counts
- <https://docs.python.org/3/c-api/arg.html> - Parsing arguments
- <https://docs.python.org/3/c-api/refcounting.html> - Reference counting


## Overall Thoughts

In the EnHackathon environment I find it much easier working with others than
working alone, although in general I find I also work well in an isolated
environment. I feel like the choice to work on larger pieces of work in small
groups was a good one, and is something we could still aim to do more.
Personally I see this time as a kick-starter for working on CPython in my own
time - it would be great if other people taking part in EnHackathon would be
interested in doing the same.

Although I received a reasonable amount of support from the Python community
when I emailed before EnHackathon started, it's been hard to translate that into
feeling there are people we can go to with the types of things that people get
stuck on. It seems like a lot of the time what people need is the ability to put
together a draft patch and get some feedback quickly, to allow for iterations to
be made without losing interest or deciding they might as well keep trying
without input from an experienced developer. It seems like the best way we
currently have to do that is to attach a patch to an issue, and maybe add
someone to the nosy list (Kyle Stanley was kind enough to suggest we could add
him to nosy lists or '@' mention on GitHub for issues we're working on). Perhaps
if other devs would be happy to mentor in this way it would [artificially] feel
like the level of support was higher? It feels to me like having a discussion on
a PR open in our personal cpython repos would be preferable so that we (as new
contributors) don't have to worry about creating a messy history in a very public
record of an issue such as on BPO.


## What's Next?

We're planning on having a couple of days in a row doing EnHackathon next week
to get some momentum going - probably Weds-Thurs 20th-21st November. It looks
like we'll be continuing with about 10 people. With everyone gradually getting
more familiar with the code and the process I'm looking forward to continuing
our successful contributions!
