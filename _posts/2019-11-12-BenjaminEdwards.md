---
layout: post
title:  "EnHackathon - I'll give you a hint"
date:   2019-11-12
author: Benjamin Edwards
---

From my previous delve into Python Development, I had learnt a lot about the basics of the development workflow. I had managed to create a pull request and get it merged into the main Python repo. But the fix was only a search and replace and didn't teach me anything about the codebase itself. So today I focused on another issue marked newcomer-friendly in the typing module, which required a bit more coding.


# Getting Type Hints

The [issue](https://bugs.python.org/issue37838) in question was to do with the `get_type_hints()` api in the typing module. This could be called on a module, class or function to return a dictionary of any type hints on the variables or parameters in the object. For the case where you have:
* A decorated function
* Typed with a forward reference
* And the decorator is defined in a different file with `@wraps`, 

then there was a problem, meaning the following code would crash:

~~~python
class ForwardReferenced:
    @decorator
    def func(a: 'ForwardReferenced'):
        pass

typing.get_type_hints(ForwardReferenced.func)
~~~

The person who raised it has noticed that while this code would break, it would work with `typing.get_type_hints(ForwardReferenced.func.__wrapped__)`. 

So I started by looking at the `get_type_hints()` api, which gets the type hints through the `__annotations__` attribute. This attribute holds a dictionary of parameter names mapped to the value of their type annotation. In the case of forward references, the value was a string and so needed to be mapped into the correct object by looking in the global and local namespaces. But the global namespace, if not given as a parameter, will be retrieved from the `__global__` attribute, so if this is a decorated function you actually get the global namespace for the decorator. 

So my fix was to walk up the (potentially nested) chain of `__wrapped__` attributes and get the global namespace from the unwrapped function. Unfortunately, this was only a best-effort attempt to fix this as the `__wrapped__` attribute only exists if you use `@wraps` to create the decorator. But it was agreed that even though it didn't fix the problem in all cases, the majority of use cases would use `@wraps`.


# A Dabble with Sub-Interpreters

The second half of my day was spent helping Lewis with some segmentation faults he was having with his code. Lewis was working on adding a function to the Sub-Interpreters module to return a list of channels that were associated with a given interpreter. If you haven't looked at his [blog post](LewisGaul.html), a channel is what sub-interpreters use to communicate between one another within a process. It was my first time messing around with the C code in CPython and I had a great time trying to get to grips with it. Towards the end of the day we had made good process through the various bugs the C code was throwing at us. Unfortunately we were not able to fix all of them, but it seemed that we had made good progress.


# Conclusion

I thoroughly enjoyed my second day of Python Development and I feel like I am quickly coming to grips with the process. I am also very happy with what I achieved, setting up my second pull request and ramping up on whatever is happening with sub-interpreters. Next time I will continue to work on my review markups for the pull request and help Lewis out with sub-interpreters if he needs it.