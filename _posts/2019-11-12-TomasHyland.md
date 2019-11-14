---
layout: post
title:  "Big and Little Endian Unions: Slowly Figuring Things Out "
date:   2019-11-12
author: Tomas Hyland
---

## Intro

After my initial scoping, I'd reached the point where there were
some self contained tasks. We set up a mini team, with Dan and Anjani keen to
be involved as well! Dan took a look at the Python part of the task, which was
fairly well defined. I continued to look at how the C code worked, whilst
Anjani was finishing off an unrelated issue during the morning.


## Finding the Hidden Code

I was finally able to find a place where the C code needs to be changed! This
took me a lot longer than expected. In general I think the documentation is
quite lacking and the only way to figure out how things worked was to read the
code, which proved quite slow going. The functionality I was interested in
required following a long code path.

The key breakthrough came when comparing the setattro functions of
PyCStructType and UnionType, both of which call
PyCStructUnionType_update_stgdict. This is a key function that contains a lot of
the behaviour we'll need to modify!

It's a large function (several hundred lines) and begins with a comment "Hack
Alert" which is a little worrying. But reading through it revealed some changes
we'll need to make. One of the key areas is pep-3188, in particular ensuring
that Unions also specify the structured format data string. 

By this time Anjani was ready to help out, so I asked her to take a look at
this function in more detail and make the necessary changes.


## Understanding the rest of the C code

Even though I've found where some of the key code lives I'm still not sure how
the rest of the code fits together - there is still a fair amount of
investigation work to do. 

I'm currently looking into why UnionType_setattro uses PyObject_GenericSetAttr
but PyCStructType_setattro uses type_setattro. The generic function is just a
wrapper around PyObject_GenericSetAttrWithDict, whereas type_setattro does some
additional Unicode handling and checking of the 'name' before calling into
PyObject_GenericSetAttrWithDict. Again I can't find any documentation for this,
so it isn't clear to me yet what the additional handling is needed for, or
whether it relates to Endianness at all.

## Next Time

My plan for next time is to continue to look into the setattro behaviour and
hopefully get to the bottom of what the unicode handling is about. There are a
few other bits and pieces of investigation to do, which will likely keep me
busy for a while yet!

