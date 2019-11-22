---
layout: post
title:  "A closer look at Endianness within Unions"
date:   2019-11-21
author: Tomas Hyland
---

## Intro

This post is going to provide a more detailed look at how endianness applies to
Unions.

## Elements of the same size!

Initially, when considering endianness within Unions, we were only looking at
unions containing elements of the same size.

~~~
union example_union
{
    uint16_t first;
    uint16_t second;
}
~~~

However with this example it's harder to spot a potential problem. As first and
second are the same size, they will use the same memory. So on a Big Endian
system.

~~~

example_union my_union;
my_union.first = 1;
~~~

Taking a look at the memory here.
~~~

00 00 00 01
~~~

Likewise setting
~~~

my_union.second = 1;
~~~

Gives
~~~

00 00 00 01
~~~

Whereas on a little Endian system.
~~~

my_union.first = 1;
~~~

Gives
~~~

01 00 00 00
~~~

and
~~~

my_union.second = 1;
~~~

Gives
~~~

01 00 00 00
~~~


## Different sized elements!

Things get more interesting when we have elements of different sizes!
~~~

union example_union
{
    uint8_t first;
    uint16_t second;
}
~~~

So on a Big Endian
system.

~~~

example_union my_union;
my_union.first = 1;
~~~

Taking a look at the memory here.
~~~

00 00 00 01
~~~

Likewise setting

my_union.second = 1;

Gives
~~~

00 00 00 01
~~~

Whereas on a little Endian system.
~~~

my_union.first = 1;
~~~

Gives
~~~

01 00 00 00
~~~

~~~

my_union.second = 1;
~~~

Gives
~~~

01 00 00 00
~~~

Hence, if you are working on a BigEndian system, and want to create a C
structure using LittleEndian byte ordering, it is not sufficient to apply an
endian conversion to just the element of the Union.

E.g. if first is set and we do the conversion.
~~~

my_union.first = 1;
~~~

We might end up with something that looks like this.
~~~

00 00 01 00
~~~

Whereas setting 
~~~

my_union.second = 1;
~~~
~~~

01 00 00 00
~~~

This falls foul of the following from the C99 standard (6.7.2.1.14)

A pointer to a union object, suitably converted, points to each of its members.
