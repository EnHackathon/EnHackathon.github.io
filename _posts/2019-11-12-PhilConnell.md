---
layout: post
title:  "Keep Your Fork, There's Pie"
date:   2019-11-12
author: Phil Connell
---

Today was my first day joining everyone else for EnHackathon, aiming to
contribute to a couple of issues in [the subinterpreter project](https://github.com/ericsnowcurrently/multi-core-python/issues).

My last large attempt at contributing to CPython was an experiment with
"unboxing" integers (i.e. encoding them into `PyObject *` pointer values rather
than actually allocating objects). Unfortunately it turned out that this
effectively reduced performance across the board by about 10%!<sup>1</sup>

It was good to have a compelling reason to look at some (different aspects) of
the CPython internals again -- the subinterpreter project has the potential to
address some of the major missed use-cases for CPython, so it's great to get
involved.

*<sup>1</sup>: It turns out that extra adding branches to the most
frequently-used C APIs (e.g. INCREF and DECREF), regardless of which way the
branch is predicted, is not great for performance!*

## Forking Subinterpreters

I started by looking at the issue about [disallowing fork in
subinterpreters](https://github.com/ericsnowcurrently/multi-core-python/issues/44).
After catching up on discussion on github and [bpo](https://bugs.python.org/)
the situation seemed to be:

1. Eric already had pushed changes to block calls to `fork()` from any
   interpreter that's not the main interpreter (i.e. exactly what we want!)
2. This [broke mod_wsgi](https://bugs.python.org/issue37951), so part of Eric's
   changeset (in the subprocess module internals) was backed out.
3. A follow-up change on [issue37951](https://bugs.python.org/issue37951)
   reintroduced the logic to block fork-exec in subprocess, but *only* if a
   pre-exec function is specified.

Where did this leave everything? Well, everyone on the bpo tracker seemed happy
that the issue was fixed, but Eric's latest comment on the github issue
suggested that there might be more to do.

So the first thing to do was figure out what the latest state really was.

### Deciphering Runtime State

Digging into the code to look at the various bits of runtime or interpreter
state, it's easy to get lost in a maze of interrelated objects and accessor
functions.

To help us navigate this, I summarised the key bits visually as a point of
reference:

![Runtime state objects]({{ "/images/2019-11-12-PhilConnell_runtime-state.jpg" | relative_url }} "CPython runtime state objects")

### Subprocess Calls In Subinterpreters

Back to the fork-ing issues: were there actually any problems remaining?

It turned out the answer to this was "no", but it took me longer than I'd have
hoped to come to this conclusion!

- After calling `fork()`, only the interpreter and thread that made the call
  survive in the child.

- The potential problem is if this interpreter was a subinterpreter rather than
  the "main" interpreter: there are various assumptions that there *is* a
  "main" interpreter.

- However none of this is really an issue for the subprocess use of fork: exec
  is called almost immediately afterwards, wiping out the entire heap (and so
  all of the Python runtime state).

- The only exception to this is if the (legacy) `preexec_fn` parameter of
  `Popen` is used: this is the only way that Python code can be executed after
  fork but before exec. So if we ban that, everything is ok!

### Conclusion

My conclusion on all of this was that there's no further code change needed,
just a small [docs update](https://bugs.python.org/issue38778) to note the
change in `os.fork()` behaviour.


## Pending Calls

One of the major changes needed for the overall subinterpreter project is
moving state from the global runtime, to a per-interpreter granularity.

There's [another
issue](https://github.com/ericsnowcurrently/multi-core-python/issues/24)
tracking one part of this: moving "pending" calls from the global runtime to be
per-interpreter.

Running out of time towards the end of the day I started looking at the
history of the issue, seeing a now-familiar pattern: Eric had pushed some
sensible-looking changes to implement this, only to find that it broke a
handful of tests in an unpredictable way on a small handful of platforms!

I didn't have any time to look any deeper than this, but the outline of the
problem looked interesting, essentially:

- The regressions were crashes in interpreter finalisation during process exit:
  subinterpreters were being finalised *after* the main interpreter, and so
  trying to access freed global state.

- There's [some discussion](https://bugs.python.org/issue33608) that this is
  probably related to daemon threads that (deliberately) refuse to exit during
  shutdown, together with some (mis)use of these in the multiprocessing module.

- To make matters more difficult, nobody has managed to find a minimal test
  case to reproduce this, and nobody has managed to reproduce the problem at
  all on linux!

So no actual progress beyond digesting some history...but this could be an
interesting problem to try and dig into in future!

