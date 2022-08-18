---
title: "Matrico dense Module"
date: 2022-08-18
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `dense` module

The `dense` module provides the actual matrix type used in `matrico` by
specializing the generic [`matrix` functor module](http://numerical-schemer.xyz/2022/08/04/matrico-matrix.html) with the [`f64vector` module](http://numerical-schemer.xyz/2022/07/20/matrico-f64vector.html) as column type.
To this end, the `dense` module "is" a `matrix` module ([`dense = matrix`](http://wiki.call-cc.org/man/5/Modules#functors),
see also the [`chicken module`](http://wiki.call-cc.org/man/5/Modules) module)
and [reexport](http://wiki.call-cc.org/man/5/Modules#reexport)s as well as [rename](http://wiki.call-cc.org/man/5/Modules#rename)s
the `f64vector` module's (reexported `srfi-4`) functions from `f64vector-X` as `column-X`,
since the matrix functor requires the following functions, all prefixed by `column`, to be provided by the specializing module:

* `(column . val)`
* `(make-column dim val)`
* `(column-length col)`
* `(column-ref col idx)`
* `(column-set! col idx val)`
* `(subcolumn col idx1 idx2)`
* `(column-unfold dim fun)`
* `(column-concat . cols)`
* `(column-any? pred col)`
* `(column-all? pred col)`
* `(column-map fun . cols)`
* `(column-map-index fun . cols)`
* `(column-foreach fun . cols)`
* `(column-foreach-index fun . cols)`
* `(column-fold fun ini . cols)`
* `(column-fold* fun ini . cols)`
* `(column-dot x y)`
* `(list->column lst)`

This means a `matrico` matrix is a list of double-precision flonum vectors, and as such a __dense matrix__.

The associated source file `src/dense.scm` is [here](https://github.com/gramian/matrico/blob/main/src/dense.scm).

Before finally presenting the "top-level" user-facing `matrico` module,
the next post will summarize one more included CHICKEN Scheme module used for emulating docstrings and REPL help.

