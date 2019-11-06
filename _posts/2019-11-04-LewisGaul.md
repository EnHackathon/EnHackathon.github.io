---
layout: post
title:  "Head-first into Python's Runtime"
date:   2019-11-04
author: Lewis Gaul
email:  lewis.gaul@gmail.com
---


Today was the first official day of EnHackathon! It feels like I've been building up to this for quite a while - gathering interest from co-contributors and members of the Python community, working out how we should structure our time, and figuring out ways to maximise our potential.

Of the 13 Ensofters who have expressed an intent to take part we had just 6 today, with the focus largely being on 'having a go' and hopefully setting ourselves up better for future dates.


## My Personal Experience

### Leading EnHackathon

I put quite a lot of effort and energy into encouraging people to join in with EnHackathon, as well as doing my best to get core devs on board (if you ever find me doing marketing I either really believe in what I'm selling or I'm having an awful time!). Because of this I feel responsible for the success we achieve, which has led me to put even more effort into making it go as smoothly as possible (big thanks to Alex for taking on the responsibility of setting up the blog!).

Now that EnHackathon is in motion I feel like I can start to relax, as others are becoming more involved and less of the responsibility lies with me, which is a relief. However, being one of the more experienced (having done more research and got started at the Hacktoberfest event), I found I spent some of my time helping get other people started. One example of this is that I now feel much more comfortable than I first did at finding potentially suitable issues and posting comments on them, so I was able to provide some suggestions in that regard.

A couple more tips (since my [first blog post](../../../10/27/LewisGaul.html)) for finding suitable issues:
- Try searching for issues in stage 'patch review'. Most of these have patches attached but no open PR - these can be a good place to start.
- Try searching for issues with just a few messages. These are often simpler and/or less controversial than issues with loads of comments.
- Search a keyword in the title field, e.g. 'argparse'.


### The Subinterpreters Project

Last time I posted comments on a few issues in [the subinterpreter project](https://github.com/ericsnowcurrently/multi-core-python/issues). I spoke with Eric (owner of the project) for a while over the weekend about how a few of us could potentially make some progress during EnHackathon. Eric sparing some of his weekend to talk with me was greatly appreciated, and gave me some very useful context about the state of the project and an insight into the relevant areas of the codebase.

There were two issues to look at that I'd commented on last time, as well as two issues Eric suggested might be good for us to look at in a small team.

<https://github.com/ericsnowcurrently/multi-core-python/issues/32>  
This issue looked like the simplest one on there - copying a `tracing_possible` field from one struct to another. I had a go at doing this in the morning. The actual copying of the field (and creating a new sub-struct) was simple enough, but I wasn't sure how the copied value should be used (as my comment on the issue says).

<https://github.com/ericsnowcurrently/multi-core-python/issues/52>  
This is the issue I spent most of my day looking into. It seems that basically what's needed is a new API on the internal `_interpreters` module (Modules/_xxsubinterpretersmodule.c) to list interpreters associated with [an end of?] a channel. It took me a while of looking through the C code to come to the conclusion that the logic of associating channels and interpreters is already in place, which just about makes this seem like an approachable first proper issue! I've managed to write a sketch solution that builds, but it definitely doesn't work! I'll need to ask for some more guidance I think.

<https://github.com/ericsnowcurrently/multi-core-python/issues/24>  
An issue that was patched for Python 3.8 but had to be reverted. Should be enough information on the bpo issue to get started, using the previous patch as a starting point.

<https://github.com/ericsnowcurrently/multi-core-python/issues/44>  
Another issue that was reverted after causing issues in Python 3.8. I spoke to Eric a bit about this one - basically fork from within a subinterpreter should be disallowed, and this could be addressed by adding a 'isolated interpreter mode' flag (true indicating forking *is* allowed) to the interpreter state struct.

I think this project is quite interesting to work on for a number of reasons. Firstly, it could eventually add a great option for improving Python's use of multiple cores. Secondly, it feels somewhat separate from the standard bpo issues, providing the opportunity to implement some new functionality rather than get stuck in discussions about how to fix bugs of varying importance/obscurity.

One of the main appeals of this project is that it involves working with the core of the Python runtime, as well as involving the implemention of a new stdlib module with both C and Python components. It feels like gaining knowledge in this area is a great way to gain an understanding of the inner-workings of Python.

Some resources I found useful while working on issue 32 were:
- <https://www.python.org/dev/peps/pep-0554/#interpreters-module-api> - The PEP for exposing the subinterpreters API
- <https://pythonextensionpatterns.readthedocs.io/en/latest/parsing_arguments.html> - How to handle Python arguments in C
- <https://docs.python.org/3.9/c-api/list.html> - How to create a Python list in C code


### Other Issues of Interest

There were two regex issues I commented on last time that are still of interest to me:

- <https://bugs.python.org/issue22491>  
	This issue is an enhancement to support all unicode line boundaries in regular expressions (rather than only '\n'). I posted asking for some guidance last time, thinking it might be suitable for a small EnHackathon team. However, I haven't yet had a response. I'm not sure if this is a matter of needing to be patient, or whether there's something that can be done to get the attention of the right person...
- <https://bugs.python.org/issue23692>  
	This issue highlights a seemingly very obscure limitation/bug in the `re` module (which is addressed in the third-party `regex` module). I hadn't realised its complexity when I suggested to Jackson he might like to investigate it further! See [his blog post](../JacksonRiley.html) for more insight!



## Thoughts and Opinions

Today I definitely gained some insight into how some of Python's core C code works, and how the Python code interacts with C code. Of course there's a lot more to learn, but it's good to start the journey! I should probably take some time to familiarise myself with some more of the C API docs - it can be quite difficult trying to work most of it from the code alone (documentation on C functions seems to be lacking).

I also now feel much more comfortable with the process of asking questions in comments and assessing which bpo issues might be suitable to work on.

I think a part of the issue we've had is to do with timezones. I get the feeling quite a few active Python devs are based in the US, meaning some replies come in at the end of our working day. In the future this may work better when I'm likely to be working on this in my evenings, but will also be working in a more intermittent fashion.

Although it might not be entirely reasonable to do so, it can be very tempting to measure your success by the number of PRs opened - I noticed people have been happy to have got to the stage of opening a PR, and the size/importance of the change is not a big factor. Personally I haven't yet got very close to opening a PR ready for review. Logically I don't feel I should be bothered by this, because my focus has been on improving my ability to contribute, which I feel I can do best by getting stuck in with something more involved and impactful (assuming there's someone happy to give the guidance I'll need!).


## What's Next?

The plan is to have the next day of EnHackathon on Tuesday 12th November, hopefully with a few more people than today. We would benefit from doing some preparation in our own time before then. There is then a question of whether to also use Wednesday 13th to get some momentum over the two days, or whether to leave a gap to wait for replies. The remaining two days should then be used over the following two weeks.

I think next time I'd like to push more for working in teams of 2 or 3 people to hopefully increase the practicality of bouncing ideas off each other, and to encourage bringing our minds together while we're in the position of having to work a lot of things out.

I think we would also benefit from having more of a plan of some specific issues that could be worked on, and make sure we're clear that there is some tangible work that can be done. This would also allow us to notify interested Python devs about what we'd like to work on, and see if anyone has any extra pointers for us. Searching for issues that have patches may well be a good place to start.
