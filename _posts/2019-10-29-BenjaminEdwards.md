---
layout: post
title:  "Hacktoberfest Event - The First Pull Request"
date:   2019-10-29
author: Benjamin Edwards
---

On Sunday, Lewis and I went to the Hacktoberfest event in the Microsoft reactor centre for our first experience of contributing to open source software, namely CPython. I personally found it
a very rewarding experience as well as getting to meet some experienced Python developers and eat some delicious cakes (although Lewis would disagree).

# First Impressions

I was very nervous as we entered the building, having never been to one of these events before, I had no idea what to expect. But throughout the event everyone was exceedingly friendly and I
immediately felt at home. The space itself was open with all the accessories I've come to expect of a software company, e.g. a table football and table tennis tables, they even had a VR headset. 
With that, we found a seat and got to scouring [Python's issue database](https://bugs.python.org/).


## The Python Community

At first looking through all the [issues](https://bugs.python.org/) it quickly got overwhelming, I went in with no idea on what I wanted to work on. After some direction from Lewis I decided that
tackling one with a newcomer friendly tag was probably a good place to start. Although, even here I was a bit nervous to start messaging people but I soon found that the online community is very welcoming,
especially towards newcomers. 

To start off I picked the easiest issue I could find just to get used to the Pull Request process. All I had to do was search and replace six numbers that someone had forgot to do in a previous fix.
From there it was the usual commit and push to my personal repo. Then create a pull request into the orginal CPython repo. Some things to note though:
* There are some notes in the [Developers guide](https://devguide.python.org/) for [Pull Requests](https://devguide.python.org/pullrequest/#making-good-prs) and [commits](https://devguide.python.org/pullrequest/#making-good-commits).
* The PR's name has to be in the format `bpo-<issue number>:<helpful name>`
* Don't worry about getting something wrong, there are plenty of checks, bots and developers who will give you a helping hand if they see something askew.

And as of Monday evening [my PR](https://github.com/python/cpython/pull/16955) was merged and I have made a small contribution to CPython :smile:.


# First Issues

The rest of my time at the event was spent scoffing cakes and browsing through some of the easier issues in the database. Some of the ones I found that might be suitable for a first PR were:
* [Unix Pipe is_reading method](https://bugs.python.org/issue38314)
* [Misleading documentation for \__prepare__](https://bugs.python.org/issue15243)
* [Debug Hooks](https://bugs.python.org/issue18765)
* [Parse known args bug](https://bugs.python.org/issue16142)
* [Abstract Base Classes](https://bugs.python.org/issue17306)

Some of these do rely on whether people reply to my messages or not.


# End of the day

I really enjoyed my first experience of open source contribution and am definitely looking forward to the EnHackathon event Lewis is organising. While starting out on a large unfamiliar codebase is very daunting, the Python community is very friendly and encouraging and they really want to get as many people as possible to start contributing to Python. The only drawback would be that finding a suitable issue to work on can be a bit overwhelming, but if you go into it having a vague idea for what you're looking for then you'll have a much easier time. I know Lewis spent some time targeting regex and subinterpreter issues and had a lot of success. Otherwise definitely take a look at ones with the easy or newcomer-friendly tag. But I am very happy with how Sunday went and can't wait to continue contributing to the CPython repo.
