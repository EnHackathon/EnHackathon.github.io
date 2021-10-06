---
layout: post
title:  "Sh - the baby's coding"
date:   2019-12-11
author: Eshan Singhal
---

I took my first baby steps in the world of open source today with EnHackathon. 
I found an issue quite easily - where shutil.move returns an error related to permissions in a case where the standard unix 'mv' does not. 
There was a patch attached to the issue but this would break anybody relying on the existing behaviour - so there was a suggestion to add a new parameter in order to 
allow for full backwards compatibility. The original issue was raised way back in 2006 and there hadn't been any activity for 5 years, so my first job was 
to see whether this was still an issue today.


After a look at the code, patch and comments on the issue, I felt comfortable understanding the problem at hand and that there hadn't been significant changes in this 
area. The suggestion to add a new parameter throws up an issue, though. Adding a new parameter would require the consumers of the API to amend their calls to use the 
new parameter if they so wished. There was something unsatisfactory in special casing shutil.move, the consumer here - but importantly takes in an 
optional copy function, so I settled for leaving a comment with my thoughts on taking this issue forward.


Overall, a positive day in that I could take an issue and understand it in whole - but slightly unsatisfactory that I concluded by simply adding a comment in
the hopes of stirring activity rather than making progress towards a pull request.

Eshan 
