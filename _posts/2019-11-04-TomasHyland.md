---
layout: post
title:  "Big and Little Endian Unions: Getting Started"
date:   2019-11-04
author: Tomas Hyland
---

## Intro

I stumbled across the following issue when looking through the bugtracker.

It seemed like a fairly approachable issue for a couple of reasons.

1. It was within the ctypes module, and I'm more experienced with C than
Python.
2. Big and Little Endian structures already exist, so surely most of the work
will involve copying how they're implemented???

My aim for the first day was to come up with a plan for the work needed to
implement this change. Dividing it into tasks which could potentially be
addressed by different people next week.

## First Steps

I thought the best way to get up to speed was to understand how the existing
Big and Little Endian stuctures were implemented.

Unfortunately grepping for obvious references within the Modules/ctypes
directory (where the C code lives) didn't yield anything useful.

I decided to take a look at Lib/ctypes (where the Python code lives) instead.
This was a lot more fruitful. It didn't take long to come up a plan for what
was needed on the Python side.

## Struggles with TypesTypes and Types 

So having made good progress on the Python side, I took another look at the
Modules/ctypes code to see if I could figure out how things worked.

My assumption was that changes would be needed to the C code, however what those
would be did not seem obvious when reading through! It even seemed possible
that things could just work without any changes!???

The main thing to get my head around initially was the distinction
between all of the TypeTypes and Types defined within this module.

There were the following:
* UnionType
* UnionTypeType
* StructType
* PyCStructTypeType

It still isn't clear to me why StructTypeType is prefixed by PyC, UnionTypeType
seems to be the only TypeType without this prefix (e.g. PyCArrayTypeTYpe).

Fairly late in the day I came across
[the types docs](https://docs.python.org/3/c-api/type.html#c.PyTypeObject) which
helped clear things up somewhat.

PyCStructTypeType is a meta type/class. Creating a new class using it as a
metaclass will call the constructor PyCStructTypenew, which replaces the Types
Dict member with a new instance of StgDict (a dictionary subclass, containing 
additional C accessible fields).

Both contructors PyCStructTypenew and UnionTypenew call into the same common
constructor StructUnionTypenew. A flag is passed to indicate if it's a
structure or a union. I think the key to figuring out what changes are needed
will be understanding how this constructor works.

Other areas that need further investigation are:
* StructUnionTypeparamfunc: Also shared between both types
* The setattro functions: PyCStructTypesetattro and UniontTypesetattro

## Next Time

So I haven't quite gotten to the point I wanted to. I still need to spend more
time getting my head around the C code before I can be sure about what changes
are needed. However I think the Python part of this task is pretty clear. It
should be possible for someone to work on it in parallel!

