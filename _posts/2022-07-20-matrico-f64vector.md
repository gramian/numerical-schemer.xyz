---
title: "Matrico f64vector Module"
date: 2022-07-20
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s Double-Precision Flonum Vector `f64vector` Module

For `matrico`, the basic homogeneous vector support of the included [`srfi-4`](https://gramian.github.io/numerical-schemer.xyz/2022/07/13/chicken-srfi4.html) module needs to expanded to provide basic functional tools,
such as "map" and "reduce" functions.
I limit myself to the `f64vector` variant of homogeneous vectors, as this type will be a basis for `matrico`'s matrix type.
This module imports the [`fpmath`](https://gramian.github.io/numerical-schemer.xyz/2022/07/08/matrico-fpmath.html) module,
solely for its `fp*+` function which is needed for the implemented dot product function.
However, in the future, once `(chicken flonum)` provides `fp*+` natively and former versions don't need to be supported,
i.e. Ubuntu LTS repositories feature a CHICKEN including `fp*+`, `fpmath` can be removed from the list of imports.
As this module is useless without the `(chicken srfi-4)` module, it is reexported.
You can take a look at the `src/f64vector.scm` source file [here](https://github.com/gramian/matrico/blob/main/src/f64vector.scm).

### Generators
First, a procedural generation of vectors needs to be available,
to avoid the user having to do such operations via a mutation and consequentially
also avoid an unnecessary zero initialization of the vector.

* `(f64vector-unfold dim fun)` - Return `f64vector` with dimension-argument-many elements resulting from applying function argument to respective elements' indices.

Another constructor of sorts is the concatenation of two or more vectors:

* `(f64vector-concat . vecs)` - Return `f64vector` holding all argument list-of-vectors elements concatenated vector-by-vector.

### Predicates
Next, a generalization of predicates for vectors, similar to `any` and `every` of [SRFI-1](https://srfi.schemers.org/srfi-1/srfi-1.html) is provided.
These functions return true or false and are thus marked with a question-mark.
Naming-wise, I prefer Matlab's [`all`](https://www.mathworks.com/help/matlab/ref/all.html) to Scheme's `every`:

* `(f64vector-any? pred vec)` - Return true if any element of argument `f64vector` fulfills predicate argument.
* `(f64vector-all? pred vec)` - Return true if all elements of argument `f64vector` fulfill predicate argument.

Testing elements of a vector on a predicate and returning matching elements and/or indices is not covered by these functions,
and may be added later on as a `f64vector-find` function.

## Mappers
An important sub-set of iterator functions are the mappers.
R6RS defines a `vector-map` in the standard, R5RS does not,
and hence (I assume), CHICKEN's [`srfi-4`](http://wiki.call-cc.org/man/5/Module%20srfi-4) does not provide a `f64vector-map`.
For performance reasons the basic map operation is implemented for one, two, and variable number of vector arguments (see [`case-lambda`](http://wiki.call-cc.org/man/5/Module%20(chicken%20base)#case-lambda)).
As opposed to lists, the index of a vector element is not only readily available without traversing the container,
but also a lot more relevant than for lists.
Hence, I added a variant of map, whose function has, in addition to the vector element, the element's index as argument,
similar to Gauche's [`vector-map-index`](https://practical-scheme.net/gauche/man/gauche-refe/Vector-family.html#index-vector_002dmap_002dwith_002dindex)
and `vector-map` in [SRFI-43](https://srfi.schemers.org/srfi-43/srfi-43.html).
For lists however, I made it a rule, that whenever I was considering list indices (and thus may want a `map-index`),
I would rethink the algorithm; of course, there are always exceptions, i.e. [`sublist`](https://gramian.github.io/numerical-schemer.xyz/2022/06/12/matrico-utils.html).

A sibling of the `map` function, is the `for-each` function, which, opposed to `map`, does not return a list (or vector),
but always iterates in order through the container.
Here, I implemented `f64vector` versions of `for-each` and `for-each-index`,
whose practical use lies in my case only in side-effect operations, like printing to terminal or writing to file.

* `(f64vector-map fun vec)` - Return `f64vector` of elements of function argument applied to `f64vector` argument's elements.
* `(f64vector-map fun vec wec)` - Return `f64vector` of elements of function argument applied to `f64vector` arguments' elements.
* `(f64vector-map fun . vecs)` - Return `f64vector` of elements of function argument applied to `f64vector` arguments' elements.
* `(f64vector-map-index fun . vec)` - Return `f64vector` of elements of function argument applied to `f64vector` arguments' elements and their index.
* `(f64vector-foreach fun . vec)` - Return `void`, apply function argument applied to `f64vector` arguments' elements in-order.
* `(f64vector-foreach-index fun . vec)` - Return `void`, apply function argument applied to `f64vector` arguments' elements in-order and their index.

### Reducers
Finally, and of vital importance, the folders (or reducers) are defined.
Naturally, two folds, forward and backward, join the mappers.
It might not be obvious why one variant is not sufficient:
First, a "fold" reducer can return anything and not only `f64vector`s or `flonum`s.
Second, even if one thinks of numerical operations like summing the elements of a vector,
it may be beneficial to sum from the front (back) if the vector is sorted increasingly (decreasingly),
which is, for example, typically the case for [Singular values](https://en.wikipedia.org/wiki/Singular_value_decomposition) of matrices, to prevent numerical annihilation.
Furthermore, the "backward fold" will play a central role in a specific matrix operation.
Due to its principal role in (numerical) linear algebra,
a specialized third reducer is included - the [dot product](https://en.wikipedia.org/wiki/Dot_product) (also known as: "inner product" or "scalar product") of two vectors (of same dimension).
While the dot product is a prime example of a "map" (`fp*`) and a "fold" (`fp+`) operation,
again performance dictates to specialize this function.

* `(f64vector-fold fun ini . vec)` 
* `(f64vector-fold* fun ini . vec)` 
* `(f64vector-dot x y)`

In principles the predicates and mappers could have been implemented as special cases of the reducers,
which, while elegant, is not achieving best performance.

Altogether, the `f64vector` module is balancing minimally required functions
with maximum performance for `matrico`.
Next time, I will discuss CHICKEN Scheme's outstanding module system.

