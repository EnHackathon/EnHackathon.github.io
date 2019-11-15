---
layout: post
title:  "Overflowing with issues and uniting to work on Unions"
date:   2019-11-12
author: Anjani Agrawal
---

I looked at the overflow issue from last time and got the issue closed in the end. I also joined Tom and Dan in looking at the Big and Little Endians issue and got the chance to start looking at a more involved issue in a small team. Taking part in EnHackathon has motivated me and also helped pave the way for me to contribute to open source in my own time as well. :smile:


## Overflow error issue and related NULL path issue

I picked up the overflow error issue from before. The issue considered the case where a file descriptor greater than the maximum allowed value is passed to the API which checks if it is a file and what the 'right' behaviour should be in this case. This led to looking at another similar issue, which discussed the case where paths, which are either NULL or contain NULL, are passed when checking if they exist in the filesystem. This issue had already been resolved and closed. Following the discussion on both threads, I understood the different arguments for and against each possible approach and added my own point of view. The possible behaviour in this case is briefly as follows:
 * Return FALSE: Would mask bugs in programs as FALSE is also returned for files which don't exist
 * Raise an error: The type of the argument is not the problem here but the value of the argument is, so ValueError seems right.
	- Raise a TypeError
	- Raise a ValueError
As this was already the behaviour seen in Python, I got this issue closed.


## Getting started with Big and Little Endians

Tom had started looking at this issue last time and did the initial scoping of the task. This issue involves adding support for Big and Little Endian Unions to the ctypes library which provides C-compatible data types in Python. There is already code for Big and Little Endian Structures. Since I hadn't worked with ctypes before, I first had a look at the documentation to get some background context. The task was split into smaller self-contained tasks and I was allocated the task to investigate one of the functions, PyCStructUnionType_update_stgdict which is more than 500 lines long. After cloning from our shared team repository, I encountered some issues when building. I tried resolving them in the same way as last time but that didn't work. I currently have one repository which builds successfully and another repository which doesn't build successfully, which seems a bit strange... I made a start on my subtask by using the other working repository and look forward to continuing my investigation next time. 