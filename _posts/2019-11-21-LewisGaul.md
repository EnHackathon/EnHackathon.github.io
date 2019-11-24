---
layout: post
title:  "First Subinterpreters PR"
date:   2019-11-21
author: Lewis Gaul
email:  lewis.gaul@gmail.com
---


I decided it would be good to have a couple of days of EnHackathon in a row this
time for continuity. I've taken a step back from helping to find things for
other participants to work on - now that we're past the first couple of days
people are clearer on the process.

I continued to focus on issues in the subinterpreters project, with continued
help from Phil and Ben. I think it's been nice for all of us to have people to
work with and bounce questions and ideas off!


## Subinterpreters Continued

Picking up where I left off [last time](../12/LewisGaul.html), I spent a fair
amount of time making final improvements to the `channels_list_interpreters()`
API I'd been working on. I then started taking a look into a couple of other
issues.

I'd also recommend reading [Phil's post](./PhilConnell.html) about the issue he
was working on.


### New 'List Channel Interpreters' API

<https://github.com/ericsnowcurrently/multi-core-python/issues/52>  
<https://bugs.python.org/issue38880>  
<https://github.com/python/cpython/pull/17323>

Having not heard from Eric since last time, I decided to plough ahead with my
interpretation of the implementation required. There were a few concrete things
left to finish off:
- Make the API accept an argument determining whether to list interpreters for
  the send of receive end of the channel.
- Do a git rebase to get rid of the PEP 554 changes I originally based my branch
  off.
- Add tests (in Lib/test/test__xxsubinterpreters.py).

It didn't take too long to get those things finished off with Ben's help. I then
asked Phil to take a look through in case he had any thoughts, which it turned
out he did! Most of his comments were related to details of the C implementation.
Going through his points took a bit of time (e.g. working out how to define
`PY_INT64_T_MAX` without assuming `stdint.h` is available!), but in a way they
were an interesting exercise in C!

I then raised my changes as
[my first CPython PR](https://github.com/python/cpython/pull/17323)! Upon
review, Victor Stinner requested that I revert the addition of the
`PY_INT64_T_MAX` macro in favour of `INT64_MAX` in `stdint.h`, but Eric has
since echoed Phil's question of whether we can assume `stdint.h` will always be
available...


### Other Issues

After getting the 'list channel interpreters' PR up, I then started looking in the
[subinterpreters repo](https://github.com/ericsnowcurrently/multi-core-python/issues/)
for something to work on next. I started looking into the following:
- [issue37292](https://bugs.python.org/issue37292) -
  This issue suggests it should be possible to unpickle an object in a
  subinterpreter when the object's class is defined in the `__main__` namespace.
  Initially it seems surprising that this should be expected to work, but it
  seems something like this is possible with `subprocess`. My next step is to
  work out how it's done with `subprocess` and see if that approach applies in
  the same way to subinterpreters.
- [issue36225](https://bugs.python.org/issue36225) -
  This issue was easy to get started on - the BPO issue has detailed instructions
  from Nick Coghlan for how to write a test that reproduces the problem. The
  problem is that the embedding API
  [states that](https://docs.python.org/3/c-api/init.html#c.Py_EndInterpreter)
  subinterpreters will be implicitly be shut down when `Py_FinalizeEx()` is
  called, when in reality if any subinterpreters are left alive when calling
  this it currently causes a `Py_FatalError`. Reproducing the issue was the easy
  part - next time I will look into fixing it!


## Summary

I feel like the task to add the `channel_list_interpreters()` API was an
excellent first issue to work on, and I feel lucky to have found it - it's
taught me a lot about CPython's C API while also being very satisfying to work
on and introducing me to the subinterpreters module. It's also good to have been
reminded of how to tackle bug fixes: first work out how to reproduce the
issue, then narrow down the problem and identify the culpable function (perhaps
using a crash backtrace, or tools such as GDB). I now feel more capable of
picking up issues and having a hope of being able to make progress!


## What's Next?

Personally I have one work day left to spend on EnHackathon, although there may
be a couple of people who haven't taken part every day that may want to do a day
or two without me.

I'm wondering whether to encourage more people in the company to join us for my
last day, with the aim of getting people started to lower the barrier to
contributing in the future. This needs to be weighed up against the level of
productivity we can expect if people only have one day to spend on it.

There's also the question of whether/how EnHackathon will be happening next year.
Some people have suggested it would be good to do it early in the year while
this month's work is fresh in our minds, which seems like a good suggestion to
me. I think it's unlikely I'll have such a leading role next year, and may
choose not to take part at all (I haven't decided yet, but might like to use
the time to do an entirely different type of volunteering). Regardless of the
future of EnHackathon, I'll definitely be continuing with CPython contribution
in my spare time.
