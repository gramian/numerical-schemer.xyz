---
title: "CHICKEN SRFI-4 Module"
date: 2022-07-13
tags: Scheme matrico
---

## CHICKEN Scheme's included `srfi-4` module

The next mini (incomplete) guided tour through an included CHICKEN Scheme module is about [`(chicken srfi-4)`](http://wiki.call-cc.org/man/5/Module%20srfi-4), an implementation of [SRFI-4](https://srfi.schemers.org/srfi-4/srfi-4.html).
This module provides homogeneous vectors, particularly double-precision flonum vectors (`f64vector`s), which can only hold flonums.
In the same way `(chicken flonum)` is an included module for performance reasons,
`(chicken srfi4)` provides increases performance for vectors of entries of a single type.
While SRFI-4 defines ten types of homogeneous vectors:

* unsigned integer: 8-bit (`u8`), 16-bit (`u16`), 32-bit (`u32`), 64-bit (`u64`),
* signed integer: 8-bit (`s8`), 16-bit (`s16`), 32-bit (`s32`), 64-bit (`s64`),
* floating-point numbers: 32-bit (`f32`), 64-bit (`f64`),

due to the use in `matrico`, I will only focus on the `f64`-typed vector.

The base functionality, as for a record (structure), is a constructor and a predicate testing for this type:

* `(make-f64vector len [val])` - Return `f64vector` with length-argument-many entries; optionally, all entries are set to the value argument.
* `(f64vector? any)` - Return boolean answering if argument is `f64vector`.

To obtain the length (dimension) of the vector, similar to the non-homogeneous vector, one can use:

* `(f64vector-length vec)` - Return number of entries of `f64vector` argument.

Similarly, the accessors follow consistently:

* `(f64vector-ref vec idx)` - Return entry of `f64vector` argument at index argument.
* `(f64vector-set! vec idx val)` - Mutate entry of `f64vector` argument at index argument to value argument.

Lastly, a converter from lists to `f64vector`s is available (again, as for the standard vector),
and also the non-R5RS counterpart to `subvector` (from [`(chicken base)`](http://numerical-schemer.xyz/2022/06/03/chicken-base.html)):

* `(list->f64vector lst)` - Return `f64vector` from list of flonums.
* `(subf64vector vec from to) ` - Return `f64vector` holding entries of first index argument till second index argument.

Overall, this is a very natural addition of homogenous vectors to the standard
vectors, even regarding extended functionality in CHICKEN.
This attention to consistency and detail, is what makes CHICKEN Scheme so enjoyable.
Unfortunately, some functions, like a `vector-map`, are neither part of this module nor SRFI-4 in general,
which will be explored in the next post.
