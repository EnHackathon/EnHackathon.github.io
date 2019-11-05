---
layout: post
title:  "EnHackathon - Taking First Steps in asyncio Contribution"
date:   2019-11-04
author: Callum Ward
---

Today was the first day of `EnHackathon`: the Python open source contribution
attempt of a couple of us working at Ensoft office, Cisco. Whilst it was
a little overwhelming trying to find suitable points of contribution, especially
in such a large and well-worn codebase, by the end of the day I had managed to
make at least a dent into giving back to CPython.

`TL;DR` is in the Conclusions.

## Starting Out

I started out by looking for issues on [BPO, Python's bug
tracker](bugs.python.org) which were marked with the `newcomer-friendly`
keyword, as they are issues specifically marked out by maintainers as attainable
first contributions.

I got a bit of luck, as almost all of the current `newcomer-friendly` issues had
been chased up by fellow contributor Ben: he kindly let me take and work on
a small task to add a new method to an existing implemented class in the
`unix_events` module of `asyncio`. [(bpo-38314)](bugs.python.org/issue38314)

The change basically requests that the method `is_reading()` is added to the
read pipe transport for `asyncio` as implemented on UNIX systems, since it is
available for other read transports described in the `asyncio` docs.

## Making the Change

The change itself was fortunately quite simple: as you can see from the BPO
comments, a maintainer and the issue raiser had basically suggested the
implementation of the method: as a newcomer, I just had to familiarise myself
with a bit of the internal state of the class.

Once I had made the change, I needed to add some tests: the best thing to do for
this is to look under `Lib/test` for the overall module you're contributing to
and `grep` for the name of key methods or functions around those you're adding
to to find existing tests. The alternative if the tests are well laid out is to
look for the file which tests the specific module you're contributing to: for
me, that was `unix_events <=> test_unix_events` (as is conventional).

Fortunately, the style of tests there was quite familiar to me (module unit
tests making heavy use of `unittest.mock`), and had a preset structure I could
copy, changing the assertions to constrain the functionality of my new method.

## Raising the PR

Finally, the potentially scary part: raising the pull request so that my changes
could be reviewed. As has been mentioned in other posts, there are some format
requirements on PRs, but these are
[well-documented](https://devguide.python.org/pullrequest/) and not too onerous.

The trickier part is to get all of the necessary automation and bot interaction
out of the way: one of the things that tripped me up was the creation of
a `NEWS.d` entry to accompany my change: this will be marked in red by the news
bot.

`NEWS.d` entries are needed for almost all Python changes, but especially in the
case that it caused any outward change (like adding an API or change of
behaviour): the exact need is [documented on the
`devguide`](https://devguide.python.org/committing/#what-s-new-and-news-entries).

The process is a little confusing, but essentially:
- Click on the `Details` tab of the `bedevere/news` bot checks item
- Sign in to GitHub for `blurb-it`, and give it access to your
  `<username>/cpython` fork (or all repositories, if you really want)
- Write your blurb and fill in the BPO and PR info: as suggested, don't use the
  BPO number or PR number in the blurb as it appears in the `NEWS.d` entry
  filename 
- It generates a file and adds it to your PR, which should trigger the
  `bedevere/news` bot to re-check and mark the check as passed

By the end of the day, a maintainer had commented on the post to suggest some
changes, and I'm currently working through resolving what the changes should be
and making them. However, now that the PR is raised and all the hurdles from the
CI are finished, I expect the rest of the fix to be the familiar coding and
reviewing cycle(s).

## Conclusions

- Whilst the process of opening PRs can feel a bit daunting, it is quite rewarding
  once the PR exists and feels headed for merge.
- Finding an issue can be tricky, but commenting on issues that seem to have
  stalled or looking for `newcomer-friendly` tags are good ways to get started.
  Even just causing an issue to be closed or reopening the dialogue on a change
  are good contributions to such a large machine!
- There are some hurdles to raising the PR and getting it ready for merge, but
  these are largely well documented on the
  [`devguide`](https://devguide.python.org/).
- There's definitely *still* items out there which the CPython community could
  use help with, despite the codebase's size and age, so there's still a lot of
  reward to be had in contributing!
