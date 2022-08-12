---
title: "CHICKEN memory representation Module"
date: 2022-08-12
tags: Lisp Scheme SchemeLang ChickenScheme Chicken-Module
---

## CHICKEN Scheme's included `memory representation` module

This is a focused look at a single method from CHICKEN Scheme's included [`(chicken memory representation)`](http://wiki.call-cc.org/man/5/Module%20(chicken%20memory%20representation)) module
from the point of view of `matrico`'s [matrix functor module](http://numerical-schemer.xyz/2022/08/04/matrico-matrix.html).
The particular considered method makes a recursive memory duplicate of its argument:

* `(object-copy any)` - Returns a (recursive) copy of the argument.

This copying functionality becomes visibly relevant as soon as a (`matrico`)-user mutates matrix entries.
In `matrico` only a single user-facing function allows mutation, namely `matrix-set!` which beomces user-facing as `mx-set!`.
As an example, after a matrix is created, a part is extracted and then an element of the extracted sub-matrix is mutated;
this would, without an `object-copy` in the extracting function, also change the original source matrix from which was extracted,
as these are the same objects in memory.

I would have preferred to not have any mutation, but it is sometimes necessary to manipulate single entries,
for example in spatial discretization of a [partial differential equation](https://en.wikipedia.org/wiki/Partial_differential_equation),
in the [method of lines](https://en.wikipedia.org/wiki/Numerical_methods_for_partial_differential_equations#Method_of_lines),
where specific boundary values need to be manually added to a semi-discretized operator.

As mentioned in the last post, this copying issue affects the functions:
`matrix-horcat`, `matrix-vercat`, `matrix-col`, `matrix-submatrix`, `matrix-vec`, and `matrix-explode`,
but does not seem to impact performance.
However, the non-mutating `matrix-set` function, a non-mutating variant of `matrix-set!`,
copying a given matrix and returning the copy after changing one entry, also uses `object-copy`. 

In the next post, I progress from the generic matrix functor to the matrix type used practically in `matrico`.





