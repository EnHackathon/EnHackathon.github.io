---
layout: post
title:  "EnHackathon - Won't you sing along with me"
date:   2019-11-12
author: Jackson Riley
email:  jackson.riley@btinternet.com
---


Today was my second day of `Enhackathon`, and I spent most of it writing docstrings, while poking a few other threads on BPO.


## What I did

### Failing test case

[Issue 36092](https://bugs.python.org/issue36092), regarding a failing test case, `test_patch_descriptor`, had been opened in February. Using the handy `git log -S "<search string>"` command, I was able to establish that the testcase in question had been removed in May in the course of some related changes, and get the issue closed. Not exactly a large contribution but one fewer active issue, and all before coffee!

### Collections.ABC docstrings

[Issue 17306](https://bugs.python.org/issue17306) covers adding better docstrings to the Abstract Base Classes in [`collections.abc`](https://docs.python.org/3/library/collections.abc.html).

I spent over half of the day updating/adding class-level docstrings - sounds slow but there are a lot of classes! Doing this involved doing some research about what certain classes and methods were used for, so was educational for me as well as hopefully being useful for others in the future.

I've uploaded my diff as it is and am waiting to get comments/markups from the dev on the BPO thread. Fingers crossed it'll be ready for a PR after a few tweaks.

### MagicMock and divmod

[Issue 34716](https://bugs.python.org/issue34716) covers fixing up an interaction between [`unittest.mock.MagicMock`](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.MagicMock) and the built-in [`divmod`](https://docs.python.org/3/library/functions.html#divmod) function.

`MagicMock` is an incredibly useful way of mocking out classes/functions when testing your code, because it's very easy to set-up and use. You can do pretty much anything with `MagicMock`s and they'll just give you more `MagicMock` instances, like some terrifying calculus problem. However, the way that people tend to use `divmod` conflicts with that.

For example
```python
>>> from unittest.mock import *
>>> floordiv, mod = divmod(5, 2)
>>> floordiv
2
>>> mod
1
>>> floordiv, mod = divmod(MagicMock(), 2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: not enough values to unpack (expected 2, got 0)
>>>
```

This isn't a problem with `MagicMock` per-se, but rather with the way that people tend to unpack the result of divmod, assigning the integer division and modulus result straight away. However, we're used to MagicMock being able to handle anything thrown at it, so there are a couple of solutions (first three suggested on the BPO thread).

1. Return some constant value when `MagicMock.__divmod__` gets called, e.g. `(1,0)`. This could then be happily unpacked. Not very flexible though.

2. Return a pair of `MagicMock` instances. This is very sensible and in-keeping with the rest of the behaviour of `MagicMock`.

3. Redefine the default behaviour of `MagicMock.__divmod__` in terms of `MagicMock.__floordiv__` and `MagicMock.__mod__`. This would return the same as (2) by default, but if the user were to set return values for `__floordiv__` and `__mod__`, the result of the `divmod` call would be changed appropriately. I think this is pretty tasty but it's a fairly niche case and might not fit with how `MagicMock` usually behaves. Would definitely have to be documented!

4. Change the behaviour of `MagicMock` more generally such that trying to unpack a `MagicMock` instance into two variables, for example, would assign a new `MagicMock` instance to each. This would be a bit more far-reaching than this issue but might be worth thinking about. This would fix the issue as a side-effect, as the current result of e.g. ```divmod(MagicMock(), 2)``` is another instance of `MagicMock`.

I had initially thought (3) was best but have been mulling it over and now prefer one of (2) or (4). Time for discussion on the BPO thread!

## Thoughts
I enjoyed my full day of contribution - the first PR remains elusive but I think I have made a decent start. Doc-writing was surprisingly difficult, summing up some of the ABCs in a way that was non-trivial and helpful was not always easy. I'm looking forward to being able to argue the case for (2) or (4) and hopefully make a start on implementing one of them next time.
