---
layout: post
title:  "PRogress"
date:   2019-11-21
author: Jackson Riley
email:  jackson.riley@btinternet.com
---


Not one, dear reader, not two, but *three* pull requests were submitted today. What a rush.


## What I did

### Collections.ABC docstrings

[Issue 17306](https://bugs.python.org/issue17306) covers adding better docstrings to the Abstract Base Classes in [`collections.abc`](https://docs.python.org/3/library/collections.abc.html).

I hadn't had any comments on my patch that I posted to the bpo thread last week, so I went ahead and submitted a pull request. I think this was the right thing to do, and hopefully it'll get a review before my next day of `EnHackathon`.

### MagicMock and divmod

[Issue 34716](https://bugs.python.org/issue34716) covers fixing up an interaction between [`unittest.mock.MagicMock`](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.MagicMock) and the built-in [`divmod`](https://docs.python.org/3/library/functions.html#divmod) function.

My winners of the options from [last time](../12/JacksonRiley.html) were

* Return a pair of `MagicMock` instances. This is very sensible and in-keeping with the rest of the behaviour of `MagicMock`.

* Change the behaviour of `MagicMock` more generally such that trying to unpack a `MagicMock` instance into two variables, for example, would assign a new `MagicMock` instance to each. This would be a bit more far-reaching than this issue but might be worth thinking about. This would fix the issue as a side-effect, as the current result of e.g. ```divmod(MagicMock(), 2)``` is another instance of `MagicMock`.

I posted as such on the bpo thread and was informed that the second option was a bit ambitious - there's no implementation-agnostic way for `MagicMock` to "know" how many variables it's being unpacked into.

I therefore decided to plump for the first option and looked through `unittest.mock` to find the right place to add this functionality - I ended up taking a bit of a scenic route but got there in the end. The solution that I went for was to add `__divmod__` and `__rdivmod__` (`__divmod__`'s right-hand cousin) to the list of side-effect methods, which includes methods such as `__eq__` and `__iter__` which have a default behaviour within `MagicMock`. 

For instance, `m.__eq__(other)` checks by default for object identity: if `m` and `other` are names for the same object. Adding `__divmod__` and `__rdivmod__` was simply a matter of defining some new default behaviour - return a pair of `MagicMock` instances. This now means that `MagicMock` behaves as

```python
>>> from unittest.mock import *
>>> floordiv, mod = divmod(MagicMock(), 2)
>>> print(floordiv, mod)
<MagicMock id='139748283461136'> <MagicMock id='139748283411024'>
>>> floordiv, mod = divmod(2, MagicMock())
>>> print(floordiv, mod)
<MagicMock id='139748278535024'> <MagicMock id='139748278597184'>
>>> floordiv, mod = divmod(MagicMock(), MagicMock())
>>> print(floordiv, mod)
<MagicMock id='139748278678624'> <MagicMock id='139748278741744'>
```

Nice!

A few updates to the `unittest.mock` tests later, I was able to confirm that I hadn't broken anything and raise a pull request. I haven't had any activity on this yet but will hopefully do so at some point.

### Removing asyncore from tests

[`asyncore`](https://docs.python.org/3/library/asyncore.html) was one of the modules used in Python to implement asynchronous behaviour before [`asyncio`](https://docs.python.org/3/library/asyncio.html) came along, and has been deprecated since Python 3.6, but is still used in some tests. [Issue 28533](https://bugs.python.org/issue28533) covers removing these mentions and reimplementing the internals of a handful of test files using `asyncio`.

This issue was recommended to be broken into subissues for each test file in question, so spotting a trivial one I could get done before heading home, I raised [Issue 38866](https://bugs.python.org/issue38866) and removed a mention from the [`pyclbr`](https://docs.python.org/3/library/pyclbr.html) test file. Third PR raised, and it's since been reviewed! I'll probably work on some other test files next time.

## Thoughts
I think my third day of contribution went pretty well, it felt great to get some PRs raised - hopefully these don't end up blocked on review. I think I'll keep looking out for issues to work on before next time, because I spent a non-zero percentage of the day looking for issues, which is not the most efficient use of time. Still enjoying and feeling good about everything!
