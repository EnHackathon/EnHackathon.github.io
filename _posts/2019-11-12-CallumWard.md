---
layout: post
title:  "Writing Asyncio Functional Tests with Pipes"
date:   2019-11-12
author: Callum Ward
---

On the second day of `EnHackathon`, I continued to work on the outstanding item
from my last blog post on `asyncio` contribution.

After I submitted the pull request for [bpo-38314](https://bugs.python.org/issue38314),
I was told by the maintainer that `asyncio` is trying to transition from using
primarily `unittest.mock` based testing to more functional style testing, which
verifies the behaviour as the user would see it.

The only issue is that this kind of functional testing infrastructure doesn't
yet exist for the `UNIX` transport pipes which I was working on: this turned the
task from a very simple method add to a more difficult piece of testing
infrastructure work.

## Understanding Pipes

In order to write a functional test for my pipe method, I would have to actually
string together a pipe between a pair of input and output and verify its
behaviour. This requires a more in-depth understanding of how pipes work.

### Sockets

On a `UNIX` system, most things are represented as "file objects". Things that
can be read from and written to are obviously analogous to files, but this
metaphor is stretched.

This includes the idea of a "socket", where a connection to an external service
like a network or server can be "written to" or "read from" the file object on
disk. This has the effect of sending and receiving data to and from the server.

### Pipes

The "file object" analogy even extends to pipes. A pipe is a file object
representing "connection" between two endpoints; typically one endpoint reads,
and the other writes. Both see the pipe as a "file", however what is actually
being done is a transfer of information between processes on the system (like
a client and server, or the `stdout` of some command and the `stdin` of a `grep`
process).

To activate a pipe, both the read endpoint and the write endpoint must be
opened.

### `asyncio` transport

The `asyncio` module provides a wrapper around pipes when they're attached to
readers on the event loop. When you've registered the read (or write) endpoint
of the pipe to the loop, you're given back an object which represents the
`transport`.

The `transport` object provides methods to pause/resume reading, or in the case
of [bpo-38314](https://bugs.python.org/issue38314) a new method `is_reading()` to check
it it is actually able to read data across the pipe.

## Mistakes

Getting to even that rudimentary understanding took a lot of trial an error.
A few of the choice mistakes I made throughout the day, or particular gotchas of
working with `asyncio` event loops:

- Opening a pip file-object in read/write mode blocks until the other endpoint
  is opened: this must be done across threads, and can't be done with `async`
  due to the blocking nature (pipes only really make sense across threads
  anyway)
- Closing some objects requires information in the buffer to be flushed: if
  they're attached to the event loop this can only be done whilst it is running!
  So closing `transport` objects won't happen unless the event loop runs the
  queued callbacks: this can be done with the internal testing method on `loop`
  objects, `loop._run_once()`
- Mixing `async` and threading is hard: a lot of the issues I found with reading
  from my pipe came from the fact that the `async` event loop, the write
  endpoint handle and the read endpoint handle were all on different threads
  (the main thread and two I opened with `threading`)

However, the process of contributing a piece of reusable infrastructure off the
back of what should have a been a simple two-line method was rewarding.

## Conclusion

Hopefully, soon the PR will be ready to be merged in, and alongside my simple
method will be a whole new functional test infrastructure for testing `asyncio`
`UNIX` pipe objects in `cpython`!

