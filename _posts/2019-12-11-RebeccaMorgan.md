---
layout: post
title:  "EnHackathon - Finding issues and poking around ctypes"
date:   2019-12-11
author: Rebecca Morgan
---


I was a little late to the party joining EnHackathon and spent today looking into an issue in ctypes. Although I haven't written any new code today, I feel that I'm now more comfortable in understanding the bug fix process in Python and have a better idea of how I could contribute more in future.

## Identifying Issues

As many posts on this blog have discussed, one of the biggest challenges when starting out is identifying appropriate issues on the Python bug tracker, [BPO](https://bugs.python.org/). When looking for issues, I took a couple of different approaches:

 * Investigating the 'easy' keyword, which has a helpful shortcut under the 'Summaries' section in the sidebar. There are not many unclaimed issues in this category!
 * Investigating the 'newcomer-friendly' keyword (often found alongside 'easy'), issues which have been identified by maintainers as approachable for first time contributors.
 * Looking for issues within specific components.
 * Looking for issues at a particular stage (needs patch).

Throughout this process, I focussed on issues more than a few days old, but less than several months old. Once I had found an issue that looked approachable and wasn't actively being worked on, I started looking into the code.

## Investigating ctypes

Today I've been looking into an [issue](https://bugs.python.org/issue38860) in ctypes, in which the classmethod `from_buffer_copy` did not invoke `__new__` and `__init__` on a subclass as expected. The person who raised the issue had provided a minimal example, which I explored in gdb.

I started off reading the [documentation](https://docs.python.org/3/library/ctypes.html) for the ctypes library and looking around the code. After looking into PyObjects for while, I found the [tests](https://github.com/python/cpython/blob/master/Lib/ctypes/test/test_frombuffer.py
) for the `from_buffer` functionality. Interestingly, the tests explictly check that the 
`__init__` method is not called for subclasses of structure instantiated by calling `from_buffer_copy`. As a newcomer to both contributing and to ctypes, it wasn't clear how to progress from here - I left a comment on the issue, and am looking forward to getting a response.


## Summary

Contributing to such a large and mature codebase can be intimidating at first, but there are plenty of resources out there to guide you through the process. The [Python Developer's Guide](https://devguide.python.org/) is very thorough and helped me get started quickly. Earlier blog posts on the [EnHackathon website](/) give a good overview of what 'your first issue' workflow looks like. One of the steps I took while setting up was signing up to the [python-dev](https://mail.python.org/mailman3/lists/python-dev.python.org/), [python-ideas](https://mail.python.org/mailman3/lists/python-ideas.python.org/) and [core-mentorship](https://mail.python.org/mailman3/lists/core-mentorship.python.org/) mailing lists - it has already been really interesting to see the ongoing discussions about the future of Python.



