---
layout: post
title:  "The End of The Beginning"
date:   2019-12-11
author: Lewis Gaul
email:  lewis.gaul@gmail.com
---


Today was my last day of EnHackathon for 2019. I feel like it went fairly well, with things being a bit clearer given a number of us have now spent a number of days getting familiar with the code and the process. We had a couple of new faces in the group this time, as well as a couple following up on previous contribution efforts.

This time I felt like I had a bit more involvement in a couple of the things other people were working on - I found these issues quite interesting and would recommend reading the respective blog posts:
 - [Subinterpreters channel locking](./BenjaminEdwards.html)
 - [Subclassing `ctypes` classes](./RebeccaMorgan.html)
 - [A issue/feature request in `shutil.move`](./EshanSinghal.html)
 - [Looking into contributing to BeeWare](./CallumWard.html)


## Cleaning up Subinterpreters on Shutdown

I spent most of the day looking into the following issue: <https://bugs.python.org/issue36225>, following on from my brief look last time. The documentation for the CPython C API says:

"`Py_FinalizeEx()` will destroy all sub-interpreters that haven’t been explicitly destroyed at that point.".

However, this is [no longer?] true - `Py_FinalizeEx()` does not clean up subinterpreters, and if there are remaining subinterpreters when calling this function then a `Py_FatalError` occurs.

This issue was actually hit by HexChat, a project that presumably embeds CPython and makes use of subinterpreters in some way (they were able to fix the issue from their end, but CPython behaviour should still match what's documented!).

Thanks to some detailed comments on the BPO issue from Nick Coghlan and Eric Snow I was able to get going on this issue without direct guidance - I first added a testcase to reproduce the issue, and then started working on the fix (test-driven development, woo!).

By the end of the day I was able to get a fix in place and [opened a PR](https://github.com/python/cpython/pull/17575) (although I made the mistake of not waiting for the tests to finish running first, and it turns out one of them is failing - I'll look into that when I get the chance!).


### C API References

A selection of C API functions I had to get somewhat familiar with, all taken from <https://docs.python.org/3/c-api/init.html#high-level-api>.

`void Py_EndInterpreter(PyThreadState *tstate)`  
Destroy the (sub-)interpreter represented by the given thread state. The given thread state must be the current thread state. When the call returns, the current thread state is `NULL`. The global interpreter lock must be held before calling this function and is still held when it returns. **`Py_FinalizeEx()` will destroy all sub-interpreters that haven’t been explicitly destroyed at that point.**

`void Py_Initialize()`  
Initialize the Python interpreter. In an application embedding Python, this should be called before using any other Python/C API functions.

`int Py_FinalizeEx()`  
Undo all initializations made by `Py_Initialize()` and subsequent use of Python/C API functions, and destroy all sub-interpreters that were created and not yet destroyed since the last call to `Py_Initialize()`.

`PyThreadState* Py_NewInterpreter()`  
Create a new sub-interpreter. The return value points to the first thread state created in the new sub-interpreter. Note that no actual thread is created.

`PyThreadState* PyThreadState_Get()`  
Return the current thread state. The global interpreter lock must be held.

`PyThreadState* PyThreadState_Swap(PyThreadState *tstate)`  
Swap the current thread state with the thread state given by the argument tstate, which may be `NULL`. The global interpreter lock must be held and is not released.

`void PyEval_ReleaseThread(PyThreadState *tstate)`  
Reset the current thread state to `NULL` and release the global interpreter lock. The lock must have been created earlier and must be held by the current thread.

`void PyEval_RestoreThread(PyThreadState *tstate)`  
Acquire the global interpreter lock (if it has been created) and set the thread state to tstate, which must not be `NULL`.

`PyGILState_STATE PyGILState_Ensure()`  
Ensure that the current thread is ready to call the Python C API regardless of the current state of Python, or of the global interpreter lock.

`void PyGILState_Release(PyGILState_STATE)`  
Release any resources previously acquired.


## Summary

### My Contribution Experience

Overall I've had a good - albeit presumably slightly unusual - experience with first-time contribution to CPython, and open source contribution in general. I feel proud of having introduced so many others to open source and CPython, and hope some of them will maintain an interest in contributing and in being involved in the Python community in the future. I aim to continue and increase my involvement in the Python community over the coming years, and will continue to support/encourage others who are interested in getting involved.

I'm planning on writing a standalone blog post soon to talk about my overall experience as a first-time contributor to CPython, intending to join Tal Einat's collection of [first-time contribution stories](https://github.com/taleinat/python-contribution-feedback). This will probably mostly be bringing together thoughts from previous my blog posts, which you can read at:
 - [Hacktoberfest Event - Getting Started!](../../10/27/LewisGaul) (27/10/2019)
 - [Head-first into Python's Runtime](../../11/04/LewisGaul) (04/11/2019)
 - [Visiting the C-side](../../11/12/LewisGaul) (12/11/2019)


### What's Next for EnHackathon?

Having spoken to other EnHackathon participants, it seems likely that EnHackathon will be going forward in some form in 2020, with some people hoping to continue their contribution early in the year.

I think it's unlikely I'll personally be using the days offered by Cisco for open source next year - I'd quite like to spend my time doing something a bit different. This is partly because I know I'm invested in Python anyway and can continue my involvement in my own time! However, if there are more people who would like to get started through EnHackathon next year then I'll be more than happy to help do some organising and get people going. I'll also keep trying to think up imaginative ways to encourage more people at work (or out of work) to get involved in Python - if anyone has any suggestions please feel free to [email me](mailto:lewis.gaul@gmail.com)! :)
