---
layout: post
title:  "Hacktoberfest Event - Getting Started!"
date:   2019-10-27
author: Lewis Gaul
---


Today Ben and I attended a Hacktoberfest event in London for what felt like our first step into the world of open source contribution. Personally, a large part of my motivation to attend was to scope out the steps involved in making contributions to CPython, with the plan to run EnHackathon at some point over the coming month - I wasn't sure how much I'd actually be trying to contribute versus just setting things up for EnHackathon. However, being in a room of ~20 people dedicating the main part of their Sunday to open source, to a degree I felt inspired to spend the time attempting to complete some tangible form of 'contribution'!


## The Process

### Getting set up

The morning started off straightforward enough, with a couple of gotchas along the way. Most of the information you need is available in [CPython's developer guide](https://devguide.python.org/). Note I had already looked through the guide previously without following any of the steps except signing up to 'python-dev' and 'core-mentorship' mailing lists. Also note that my personal setup is with WSL (Ubuntu on Windows) and I use VS Code as my editor. The steps I followed to get fully set up were:
- Fork the CPython repo on GitHub
- Add SSH key to GitHub and clone the repo locally using  
  `git clone git@github.com:<user>/<project>`
  - I added my preexisting 2048 bit RSA key (~/.ssh/id_rsa.pub) to my GitHub account
  - To use the SSH protocol to push you need the remote to be configured using 'git@github.com:' instead of 'https://github.com/' (which would use HTTPS, unsurprisingly) - if you cloned using HTTPS you would need to configure the remote with  
  `git remote set-url origin git@github.com:<user>/cpython`
  - Check you can push to your GitHub account without entering credentials (check by running 'git push' - harmless if you haven't made any commits yet)
- Open the workspace in an IDE and keep the directory structure for reference: <https://devguide.python.org/setup/#directory-structure>
- Configure and make the project: <https://devguide.python.org/setup/> - I followed the instructions for Unix (you may have dependencies missing - just look for "Python build finished successfully!")
- Make a note of the required git commit message format: <https://devguide.python.org/pullrequest/#good-commits>
- Register on <https://bugs.python.org/> and add GitHub username
- Sign CLA: <https://devguide.python.org/pullrequest/#licensing> (takes a day or two to be verified, and blocks PRs from being merged)

Although it sounds like quite a lot of steps, they were mostly very straightforward and well-documented, with the main exception being setting up git credentials (which also ended up being a straightforward solution).

By the time I'd completed my setup Ben already had his first PR open! He managed to find an issue tagged with 'newcomer friendly' that was undeniably in an 'unclaimed' state, and for which the required fix was clear. Congratulations to Ben for the first ever EnHackathon PR! I'd recommend reading [his blog post](../29/BenjaminEdwards.html) for his perspective on the day.


### Looking for issues on the bpo

I then began the task that I was aware of being a stumbling block for many newcomers: finding a suitable issue to work on. Having seen that there were very few suitable issues tagged with 'newcomer friendly' (10 open issues, 5 with open PRs, most of the others claimed by other presumed first-time contributors but not seeing any activity for weeks or more), and knowing that 'easy' issues are not necessarily easy (hence the introduction of 'newcomer friendly'), I decided to search based on areas of Python.

Searching by 'Components: Regular Expressions' gave 28 open issues going back as far as 2015, most of which did not have open PRs. I flicked through a few - and by 'flicked through' I mean 'trawled through' - for many issues it takes a non-insignificant amount of time to get a sufficient level of understanding for an issue before it's possible to decide if it might be approachable. The steps I'd recommend for assessing issues are as follows:
1. Avoid issues with open PRs.
2. Avoid very new issues (opened in the last few days).
3. Look at the title and first comment to get a feel for what the issue involves.
4. Check the date it was created, whether it has any patches attached, the spread of dates on the comments, and read the last few comments to assess the state it's currently in.
   1. If it's in a state where the solution is well-defined and nobody has claimed it, you've found something special!
   2. If the last comment is someone saying they'd like to work on it...
      1. if it was recent then best to leave them a chance to do it.
      2. if the last activity was over a month ago then move on to step 5.
   3. If the last comments say something like "waiting on a decision from the maintainer" or involve a discussion about the right solution...
      1. it's probably not the best issue to get involved with... 
      2. however, if the issue seems particularly appealing to work on and the last comment was over a couple of months ago, you can post expressing your interest and asking whether a decision can be reached.
5. Analyse the patch situation - do the patches and corresponding comments indicate someone else is actively working on it? Does it have a complete solution attached that just needs converting into a PR?
6. Read the remainder of the comments, search/read the relevant code if applicable, spend 10-15 minutes getting as much of an understanding as you can.
7. If it still seems potentially doable (even if with some guidance) then post a comment stating your interest in taking it on and ask for guidance if you feel unsure about the solution. If someone had previously expressed interest in taking the issue on then it's curtious to ask them if they mind you looking into it - a week or so should be allowed for them to respond (they may have quietly been working on it). Being polite and friendly is key if you want people to help you out!

Having come across a lot of issues that seemed buried in 4bi. or 4ci., I decided to ask about how to deal with stale issues like these in an email to core-mentorship mailing list, where I received two useful replies later in the day. The responses I received are embedded in my recommendations above.

I spent an hour or two working out some of the above steps while looking through the 'Regular Expressions' issues. In that time I posted comments on 4 issues:
- <https://bugs.python.org/issue38351>  
	This issue proposes updating a single code example in documentation from using the old-style '%' string formatting to use f-strings. This involves some discussion about whether this should be done everywhere, or whether it should be changed at all. There is then interest in taking it on from another contributor, and it is stated that "We're waiting for <maintainer>'s input.". In hindsight I probably should have left this one alone, but given there'd been no activity for over 2 weeks I posted saying I'd be interested in taking it on if there was a decision on whether it's a desirable fix.
- <https://bugs.python.org/issue22491>  
	This issue is an enhancement to support all unicode line boundaries in regular expressions (rather than only '\n'). It seemed like this might be reasonably straightforward, but then I read the last comment: "It seems that large portions of Modules/_sre.c would have to be rewritten in order to do this.". I posted asking for some guidance on what would be needed, to see if it might be a feasible problem to approach with a small EnHackathon team.
- <https://bugs.python.org/issue21002>  
	This issue proposes adding methods to a private regex scanner class so that the available methods match similar public APIs. There was the point from a core-dev saying if anything were to be done it would be to make the private class public (without endorsing doing so). This seemed entirely reasonable, and there was no indication that the API should be made public, so I suggested it be closed, which it subsequently was (after my post to core-mentorship led to the relevant maintainer being added).
- <https://bugs.python.org/issue23692>  
	This issue had no activity since its creation 4 years ago, and gave an example illustrating a subtlety in the implementation of the `re` module, suggesting it should either be fixed or at least documented. I posted asking for some guidance on how to progress the issue, and later in the day got a detailed response. At the time of writing this is an issue I need to take a further look into.


### Looking for issues in the subinterpreters project

I had a specific interest in helping out with the subinterpreters project led by Eric Snow. When I'd had enough of looking for bpo issues, I switched to looking for issues in the subinterpreters project at <https://github.com/ericsnowcurrently/multi-core-python/issues>. There is a much smaller pool of issues, especially when filtered to 'complexity: low' or 'size: small'. However, in contrast to bpo issues, it doesn't feel like there's a sea of opinions to wade through! Having emailed Eric in advance and receiving encouragement from him to seek guidance by posting comments on issues, I posted on 4 issues:
- <https://github.com/ericsnowcurrently/multi-core-python/issues/52>
- <https://github.com/ericsnowcurrently/multi-core-python/issues/32>  
	Seemed like potentially approachable tasks.
- <https://github.com/ericsnowcurrently/multi-core-python/issues/42>
- <https://github.com/ericsnowcurrently/multi-core-python/issues/13>  
	Seemed like they should possibly be closed, based on their corresponding bpo issues being closed.



## Thoughts and Opinions

I found the day useful and feel like I learnt a lot about how to find suitable issues, and how to get a reasonable understanding of what's required in a reasonable amount of time. I have found the community very supportive and friendly, which makes it much less daunting to post comments and ask for help in comments and on mailing lists such as core-mentorship. The more you get involved, the more you appreciate the significance of the community that is the backbone of the whole operation.

As expected, I did find the task of looking through lots of issues tedious, especially given the aim became just to find candidate issues - I didn't find a single issue that I could immediately start working on. I would be interested to hear how this experience is expected to improve for people who give continued contributions, is it that they form areas of expertise and are able to pick up more in that area, or is it that they find things to work on in a different way? In some way, even if it's known to be a slog early on it would be good to know how we can expect it to improve as we gain more experience! It would be great if somehow there could be more newcomer friendly issues left available. Personally I would also like there to be a way to progress or recategorise issues that appear 'stuck' - when going through my assessment of issues as detailed in steps above it would be nice to be able to influence most issues in a positive way rather than spending time understanding an issue before leaving it untouched. For example it could be that for most issues you might post a comment if it seems potentially suitable to work on, but if it seems fundamentally difficult to approach as a newcomer give it a 'newcomer-*un*friendly' tag to give future newcomers a way to search in a better pool of issues than the small pool of explicitly 'newcomer friendly' ones.

Although I didn't make any contributions in the form of PRs today, I do feel I've opened doors for some issues that I can work on over the coming weeks, while also building the beginnings of relationships with people in the community. I feel positive about the prospect of being able to create a couple of PRs at some point for the issues I've posted on today, and am definitely looking forward to spending my next contribution time making code changes as well as trawling through pages of issues!
