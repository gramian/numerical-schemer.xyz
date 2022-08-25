---
title: "CHICKEN plist Module"
date: 2022-08-26
tags: Lisp Scheme SchemeLang ChickenScheme Chicken-Module
---

## CHICKEN Scheme's included `plist` module

Symbol property lists are a long-standing Lisp feature
(see [History of LISP](http://jmc.stanford.edu/articles/lisp/lisp.pdf#page=10))
which attaches a empty list to each (exisiting or defined) symbol.
This list is similar to an associative array, in that data can be stored in conjunction with a naming property;
Clojure's [metadata](https://clojure.org/reference/metadata) functionality is related in concept.
Practically, this allows to attach named (meta-)data to unique identifiers.
In `CHICKEN`, property lists are provided via the included [`(chicken plist)`](http://wiki.call-cc.org/man/5/Module%20(chicken%20plist)) module.
Internally, these property lists are realized, not as a list of pairs,
but a list alternating between property and value for all property-value pairs.
The two central functions from this module are:

* `(put! sym prop val)` - Returns value argument, which is added a under property argument to the property list of argument symbol.
* `(get sym prop)` - Returns value of property argument in property list of symbol argument, or false if symbol or property does not exist.

In `matrico`, property lists are used to emulate [Docstring](https://en.wikipedia.org/wiki/Docstring)s.
This is achieved by using the `define*` macro from the [`utils`](http://numerical-schemer.xyz/2022/06/12/matrico-utils.html) module,
which assigns a string to a symbol (`put!`) with the name of the binding used in the associated `define`.
Then, a `matrico` function is provided to retrieve the docstring (`get`) for a given function name passed as a symbol and display it.

Next, in a multi-part series, the user-facing functions of `matrico` are detailed.

