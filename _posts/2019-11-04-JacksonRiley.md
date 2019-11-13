---
layout: post
title:  "EnHackathon - Now I know my ABCs"
date:   2019-11-04
author: Jackson Riley
email:  jackson.riley@btinternet.com
---


Today was my first day of `EnHackathon`, and although I was otherwise engaged in the morning I was able to get started properly after lunch. Not a great amount of progress was made but a few nibbles have been had.


## What I did

### Bug in re module

Lewis had found [Issue 23692](https://bugs.python.org/issue23692) on BPO, which looked interesting and like there hadn't been much progress made so far.

This bug concerned `re.match()` not matching certain patterns when it should, due to a feature designed to prevent infinite iterations for non-matching groups. This was causing inconsistent behaviour such as

```python
>>> import re
>>> re.match('(?:()|(?(1)()|z)){2}(?(2)a|z)', 'a')
<re.Match object; span=(0, 1), match='a'>
>>> re.match('(?:()|(?(1)()|z)){0,2}(?(2)a|z)', 'a')
>>>
```

The first pattern matches because:
- `{2}` tells us to look for exactly 2 instances of the preceding (non-matching) group
 - The first time through, we try to match either `()` or `(?(1)()|z)` (an if clause telling us to match `()` if the first capturing group got something and `z` otherwise). NB the null group here will match anything but won't advance the match position and hence won't actually capture anything (it technically captures the empty string).
 - The null group comes first and matches successfully, but doesn't advance the match position.
 - The second time through, the null group matches again, but group 2 not having been set means that the last if clause, `(?(2)a|z)`, uses the 'else' option, tries to match a `z`, and fails, so we backtrack to the second time through the repeated section.
 - This time, we check the if clause `(?(1)()|z)`. Group 1 has matched, so we use the null group and match. Again however, the match position is not advanced.
 - Then we get to the last if clause, `(?(2)a|z)`. The second group captured, so we match `a` and the whole pattern matches.

The only difference in the second pattern is the change from `{2}` to `{0,2}`, which, theoretically, shouldn't affect anything, as this `{0,2}` matches the preceding pattern *greedily*, so should go through two iterations, because we *can*.

However, it seems that somewhere in the `re` module, there is some checking to avoid `(?:)*` type cases. Here, on each iteration of the `*`, the match position wouldn't advance, so we would keep matching forever (not ideal). It has been proposed (I haven't verified this yet) that the way `re` solves this is to stop the `*` if we haven't advanced.

Unfortunately, this means that the second pattern does not match, as after the first iteration, the match position has not advanced, so the `{0,2}` is killed off in its prime, meaning that `(2)` is not set when it comes to the final if clause, so we can't match `a`. This can be naively verified by trying:

```python
>>> import re
>>> re.match('(?:()|(?(1)()|z)){0,2}(?(2)a|b)', 'b')
<re.Match object; span=(0, 1), match='b'>
```

This does not however (in my estimation) explain the following behaviour

```python
>>> import re
>>> re.match('(?:()|(?(1)()|z)){1,2}(?(2)a|z)', 'a')
<re.Match object; span=(0, 1), match='a'>
```

I would have expected this to have not matched for the same reason as the `{0,2}` case. 

In addition, thus far we've been using the example regex provided by the original submitter, but it seemed to me as though the above logic would work just as well if we replaced the `(?(1)()|z)` with `()`. I'd therefore expect `(?:()|()){0,2}(?(2)a|z)` not to match, due to this bug.

```python
>>> import re
>>> re.match('(?:()|()){0,2}(?(2)a|z)', 'a') 
<re.Match object; span=(0, 1), match='a'>
```

Hmm. Obviously something in my understanding is incorrect, so I'm looking forward to working out what that is when I get a chance!

In any case, after it was pointed out to me that 'the regex module' referred to [regex](https://pypi.org/project/regex/) rather than [re](https://docs.python.org/3/library/re.html), I was able to have a poke around in the code and identify a few places that would be relevant for a fix - the current plan is to stop iterating if
- the match position has not been advanced (current) **AND**
- no groups have changed (proposed)


### Collections.ABC docstrings

[Issue 17306](https://bugs.python.org/issue17306) covers adding better docstrings to the Abstract Base Classes in [`collections.abc`](https://docs.python.org/3/library/collections.abc.html), and luckily for us, has been marked with a handy `newcomer-friendly` tag. Maybe I won't have to lie awake at night worrying about this one.

Python's `help()` function pulls the docstrings right from the definitions of classes and functions, and in this case could do with a helping hand when it comes to the ABCs. These are base classes which are made to be subclassed, as they provide some methods and require others to be overwritten when rolling your own (by marking them as an `abstractmethod`). This means that you can't accidentally forget to implement `__iter__` for your custom iterable class!

This issue will just involve making `help()` more useful for all of the ABCs by adding comments in the code. Who doesn't love comments?

## Thoughts
I very much enjoyed my afternoon of contribution and can't wait to continue next week. I'd recommend giving it a try to anyone, it's very interesting (as someone who's used Python a moderate amount) to be able to poke around the codebase, and I'm looking forward to starting that first PR. Devs on BPO give great insight and well-thought-out comments, often within a matter of minutes, so if you're stuck on a bug, ask!

