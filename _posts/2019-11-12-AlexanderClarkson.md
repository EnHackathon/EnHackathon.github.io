---
layout: post
title:  "Issue Hunting and First Pull Requests"
date:   2019-11-12
author: Alexander Clarkson
---

## Previously on EnHackathon

I spent time hunting on [Python's bug tracker](https://bugs.python.org/) for suitable issues to work on during the Hackathon. I was looking for two kinds of issues:
 - Issues with a small, quick fix to get used to the contribution workflow, and
 - "Interesting" issues which caught my eye for some other reason.

The bug tracker can seem opaque at first. At a first glance, it looks like issues are either already being worked on or have been long forgotten after some initial discussion.

I started by filtering for newcomer-friendly issues, and just kept scrolling back until I found something.

I was encouraged by Brandt Bucher, a member of the CPython triage team, to get started by raising pull requests for a couple of clear-cut but forgotten old issues.

It looks like the best thing to do (unless you stumble upon a new issue in the wild...) is to take one of the "forgotten" issues and take the next step. If there's a clear fix, you can go ahead and raise a pull request after making sure nobody else is actively working on it. If you like the look of an issue which needs some discussion, you could post a first attempt at a patch or bump the thread to move it along.

## Experience of First Contributions

After finding issues to work on, making a first contribution was comparatively the easy part - the workflow is standard and [well documented](https://devguide.python.org/).

I raised pull requests for a couple of small issues:
- <https://bugs.python.org/issue9495>  
	Removing extra confusing context from a traceback.
- <https://bugs.python.org/issue15243>  
	Adding clarification in the documentation that `__prepare__` should be implemented as a classmethod.

These issues were quickly reviewed by Brandt, and at the time of writing are awaiting a core review and merge.

I've also started some work on a couple more issues found during my searching:
- <https://bugs.python.org/issue25866>  
	A few documentation tweaks and corrections. The issue is old, so I'm planning on attaching a patch to the issue for some discussion before a pull request.
- <https://bugs.python.org/issue11354>  
	This is a more exciting one :smile:. This involves enhancing argparse to allow specifying that an argument should accept a fixed range of number of values, without the need for post-processing in your script.

    This has some non-trivial things to work out. The exact notation to be used needs to be decided, and it will require careful testing.

    I find the idea of doing a visible enhancement like this very appealing, so again I plan to create an up-to-date patch.

## Next time on EnHackathon

Fingers crossed, my two pull requests will be merged, and I will have made a small but non-zero contribution to CPython.

I'll also continue work on the other two issues.

## Conclusions

This has been my first journey into open-source contribution, and I found it very rewarding after the first hurdles of pinning down work to do. I look forward to more days of EnHackathon, and I've been motivated to do more open-source work after it's over :smile:.
