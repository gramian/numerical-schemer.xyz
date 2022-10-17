---
title: "Matrico Module II"
date: 2022-09-30
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `matrico` Module (Part II)

**Series**:

* [Part I](http://numerical-schemer.xyz/2022/09/16/matrico-matrico-i.html)
* Part II
* [Part II](http://numerical-schemer.xyz/2022/09/16/matrico-matrico-iii.html)

In this second part of the series of posts describing the `matrico` module, I will focus on the
_Constructor_, _Dimension_, _Predicate_, and _Accessor_ type functions, which are contained in
[`src/mx.scm`](https://github.com/gramian/matrico/blob/main/src/mx.scm) Scheme source code file,
that is **included** into the module's main file `matrico.scm`.

### Constructors

As I mentioned in an [earlier post](http://numerical-schemer.xyz/2022/08/04/matrico-matrix.html),
the actual `matrix` record is hidden from the user. Hence, to instantiate a `matrix`, constructors are needed,
which are provided in terms of basic as well as specialized matrices and column matrices;
but also a way to manually enter a matrix as a list of lists is given:

* `(mx rows cols val)` - Returns `matrix` of row and column argument `number`s and all entries set to `flonum` value.
* `(mx% lst)` - Returns `matrix` from row-major `list`-of-`list`-of-`flonum`s.
* `(mx-identity dims)` - Returns an [identity](https://en.wikipedia.org/wiki/Identity_matrix) `matrix` of dimension argument `number`.
* `(mx-exchange dims)` - Returns an [exchange](https://en.wikipedia.org/wiki/Exchange_matrix) `matrix` of dimension argument `number`.
* `(mx-hilbert dims)` - Returns a [Hilbert](https://en.wikipedia.org/wiki/Hilbert_matrix) `matrix` of dimension argument `number`.
* `(mx-pascal dims)` - Returns a [lower triangular Pascal](https://en.wikipedia.org/wiki/Pascal_matrix) `matrix` of dimension argument `number`. 
* `(mx-lehmer rows cols)` - Returns a [Lehmer](https://en.wikipedia.org/wiki/Lehmer_matrix) `matrix` of row and column dimension arguments `number`. 
* `(mx-random rows cols low upp)` - Returns uniformly random `matrix` within `flonum` lower and upper limit arguments of row and column argument  `number`s. 
* `(mx-tridiag dims low mid upp)` - Returns tridiagonal `matrix` with lower, middle, and upper band value arguments of dimension argument `number`.
* `(mx-unit dims num)` - Returns column `matrix` of dimension argument `number` with all entries zero except at index argument `number` where it is one.
* `(mx-iota dims)` - Returns column `matrix` of dimension argument `number`, with entries corresponding to the row index.
* `(mx-linspace x y num)` - Returns `matrix` with linearly interpolated rows between column `matrix` arguments and `number` of columns argument. 
* `(mx-logspace x y num)` - Returns `matrix` with base-10 logarithmically interpolated rows between column `matrix` arguments and `number` of columns argument.

### Dimensions

Next, functions are provided to determine the dimensionality of a `matrix`:

* `(mx-cols mat)` - Returns `number` of columns of argument `matrix`.
* `(mx-rows mat)` - Returns `number` of rows of argument `matrix`.
* `(mx-numel mat)` - Returns `number` of entries of argument `matrix`.
* `(mx-dims mat)` - Returns `number` of dimensions of argument `matrix`.

### Predicates

Also, the properties of one `matrix` or jointly of two `matrix`es, need to be testable.
To this end a set of predicates is implemented, including a `flonum`-safe equality comparison of `matrix`es:

* `(mx? any)` - Returns `boolean` answering if argument is a `matrix`.
* `(mx-col? mat)` - Returns `boolean` answering if argument `matrix` has just one column.
* `(mx-row? mat)` - Returns `boolean` answering if argument `matrix` has just one row.
* `(mx-scalar? mat)` - Returns `boolean` answering if argument `matrix` has just one column and row.
* `(mx-vector? mat)` - Returns `boolean` answering if argument `matrix` has just one column or row.
* `(mx-square? mat)` - Returns `boolean` answering if argument `matrix` has same number of rows and columns.
* `(mx-samecols? x y)` - Returns `boolean` answering if argument `matrix`es have same number of columns. 
* `(mx-samerows? x y)` - Returns `boolean` answering if argument `matrix`es have same number of rows.
* `(mx-samedims? x y)` - Returns `boolean` answering if argument `matrix`es have same number of rows and columns.
* `(mx-any? pred mat)` - Returns `boolean` answering if at least one entry of the argument `matrix` fulfills predicate argument.
* `(mx-all? pred mat)` - Returns `boolean` answering if all entries of argument `matrix` fulfill predicate argument.
* `(mx=? x y tol)` - Returns `boolean` answering if the entry-wise absolute difference between argument `matrix`es is below `flonum` tolerance argument.

### Accessors

Given functions to create, measure and test matrices, a basic functionality completing this set of basic matrix functions,
are functions to get or set entries or parts of a `matrix`:

* `(mx-ref11 mat)` - Returns `flonum` being the entry in the first column and first row of argument `matrix`.
* `(mx-ref mat row col)` - Returns `flonum` being the entry at column and row arguments of argument `matrix`.
* `(mx-set mat row col val)` - Returns `matrix` with entry at column and row arguments of argument `matrix` set to `flonum` value argument.
* `(mx-set! mat row col val)` - Returns void, mutates entry at column and row arguments of argument `matrix` to `flonum` value argument.
* `(mx-col mat col)` - Returns `matrix` being the column at index argument of argument `matrix`.
* `(mx-row mat row)` - Returns `matrix` being the row at index argument of argument `matrix`.
* `(mx-diag mat)` - Returns column `matrix` being the main diagonal of argument `matrix`.
* `(mx-submatrix mat row1 row2 col1 col2)` - Returns `matrix` being the sub-matrix between row and column index arguments of the `matrix` argument.

In the next part, I will describe the basic arithmetic and calculator functions
in `matrico` via _Expanders_, _Mappers_, and _Reducers_.

