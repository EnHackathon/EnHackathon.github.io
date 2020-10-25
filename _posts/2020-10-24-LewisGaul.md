---
layout: post
title:  First CPython Core Sprint
author: Lewis Gaul
email:  lewis.gaul@gmail.com
---

I was very excited to receive an email from Eric inviting me to join the week-long [CPython sprint](https://python-core-sprint-2020.readthedocs.io/), which took place this week.

This follows on from when I worked with Eric on the subinterpreters CPython project a year ago as part of EnHackathon - see my last EnHackathon [blog post](https://enhackathon.github.io/2019/12/11/LewisGaul.html) for context.


Note: This post is cross-posted on my personal blog at <http://lewisgaul.co.uk/blog/coding/2020/10/24/cpython-sprint/>.


## First Impressions

We were using Discord as the platform for communication during the sprint, but when I signed in on Monday morning I found nothing was really happening. It turned out that the sprint was largely centered around US time, so I found myself working some unusual hours!

I did manage to get in a call with Tal before the official start of the sprint to talk about the new-contributor experience. Big thanks to him for committing time and effort to improve things in this area.

The sprint officially began with an opening presentation/meeting at 4pm. My favourite part of this was everyone giving a short introduction - there were a lot of familiar names that I wasn't previously able to put a face to (or even pronounce in some cases!).

After the opening meeting everything really kicked off! There were lots of different discussion channels, which I described at one point as "a more interesting and active version of the python-dev mailing list".

My first real involvement was in a meeting about how to improve the new-contributor experience, with Tal and a few others. During this meeting I felt incredibly welcomed and that my input was respected despite being a relative outsider to the group.

It was really quite amazing to be a part of such a dedicated group of people, who all do so much for Python in all kinds of different ways (e.g. Steering Council governance, packaging improvements, migrating to GitHub, research into potential major enhancements, adding support for new platforms, ...).


## My Focus

I was allowed into the sprint as a 'mentee', with Eric being kind enough to act as my mentor. My focus was therefore to continue working on the subinterpreters project, with the high-level project aim of landing [PEP 554](https://www.python.org/dev/peps/pep-0554/) in upcoming Python 3.10. The pending work before this is expected to be accepted is to make the GIL per-interpreter, which would allow running multiple interpreters in separate threads that are actually able to run in parallel on multiple cores!

I had an open [PR](https://github.com/python/cpython/pull/17575) from EnHackathon last year, which was my focus when working on subinterpreters during the sprint. I expected to have time to take on more than this one issue, but I ended up spending a lot of time following/sitting in on other discussions and generally making the most of the event!

The PR required a bit of thought to be sure of the direction we wanted to go in - I had a couple of chats with Eric to make sure he was happy with the proposed solution.

The decision we made was:
 - `Py_FinalizeEx()` will implicitly clean up any running subinterpreters, but will emit a `ResourceWarning` if it has to do so.
 - An error will be returned if trying to call `Py_FinalizeEx()` from a subinterpreter.

These two points address the main issue [bpo-36225](https://bugs.python.org/issue36225), but also address related issues [bpo-38865](https://bugs.python.org/issue38865) and [bpo-37776](https://bugs.python.org/issue37776) relating to the question of calling `Py_FinalizeEx()` from a subinterpreter.

In the future there should be no distinction between subinterpreters and the 'main' interpreter at the C level, and it should be possible to call `Py_FinalizeEx()` from any interpreter, which would implicitly clean up all other interpreters. However, this is not a priority in the short-term.


## Improvements for New Contributors

One of the areas of the sprint I had a strong interest in was the discussion around how to improve the experience for new contributors to CPython. This is an area that has been acknowledged as lacking, and Tal has taken the lead on trying to improve things. I enjoyed getting involved in this as a fairly new contributor myself, meaning I felt I could actually provide some useful input!

Some of the points I had/agreed with were:
 - Having visible metrics could be a good way to encourage newcomers to contribute in ways like trying to revive/close stale issues, e.g. graphs/counters of number of open issues/PRs.
 - Friendly responses and encouragement to newcomers would be great, even if in the form of a bot message.
 - We want newcomers to get responses, and to make them feel welcome we want them to feel like they're on a team with the person responding. This may be easier/less daunting if the people on the 'first line of response' are relatively new themselves.
 - Support guidance/mentoring at all levels. The idea being for everyone to have someone to go to if they don't have the answer to a question, e.g. perhaps I'd help someone with some basic stuff but then be able to ask someone else if things got out of my depth (which would also be great for learning at lower levels of experience).
 - Would be nice to have a support chat room intended for new contributors and mentors, hopefully providing quick answers to basic questions and building up more of a new-contibutor community.
 - Perhaps some public messaging to would-be-contributors could help to clearly acknowledge there's a problem and talk through how we're trying to do address it. It could also be worth being clear about the best ways for them to help out, if we decide on what that would be.

Another thing that came up is that there are often 'nitpick' requests to fix formatting (in C, Python and RST files), which can add hours/days to the PR review process and be quite discouraging to newcomers. The expanded into a discussion about auto-formatting in the entirety of the CPython codebase! I look forward to seeing further discussion on that - I think it could set a great precedent for other large Python projects.


## Other Interesting Things

There were lots of interesting discussions going on during the sprint. I couldn't keep up with all of them, but there were a few that stood out to me as being especially exciting.

Alongside the work on multiple interpreters as a way to improve CPython's ability to run on multiple cores (i.e. within a single process, rather than requiring multiprocessing), there was discussion around removing the GIL entirely. This follows on from Larry Hastings's [Gilectomy project](https://pythoncapi.readthedocs.io/gilectomy.html), where it was determined the approach should be to use a tracing garbage collector instead of reference counting. This could in theory make CPython truly multi-core, and would likely require a major version bump to Python 4.0 due to the major changes required to the C API.

There was ongoing discussion around the proposed pattern matching feature, which is now split into three separate PEPs: [PEP 634](https://www.python.org/dev/peps/pep-0634/), [PEP 635](https://www.python.org/dev/peps/pep-0635/) and [PEP 636](https://www.python.org/dev/peps/pep-0636/). I think this could be a very interesting new feature and I look forward to trying it out if/when it is collapsed into the main branch.

Mark Shannon sent a proposal promising a 5x speed-up of CPython over the next few years in four phases of 50% increase each. See the [email to python-dev](https://mail.python.org/archives/list/python-dev@python.org/message/RDXLCH22T2EZDRCBM6ZYYIUTBWQVVVWH/) for details!

There was some early discussion around native support for 'exception groups', perhaps involving the introduction of `try: ... catch: ...` in Python - we'll have to wait and see what comes out from this!


## Overall Thoughts

This was the first ever virtual CPython sprint, and the general sentiment seemed to be that people enjoyed it more than expected! One of the clear positives was that it opened things up to people from all over the world, meaning this was the first sprint for quite a few of the participants.

Overall I enjoyed the sprint, and the highlight was just generally being involved in the community and discussions that were going on. I'm excited to see what some of the discussions will lead to, and would like to continue to support the multi-core projects and any new-contributor initiatives.

Once again thanks to Eric for having me along, to Tal for the work he's doing to help new contributors like me, and to everyone else who contributes to Python as a volunteer!
