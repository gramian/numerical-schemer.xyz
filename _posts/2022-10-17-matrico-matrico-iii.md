---
title: "Matrico Module III"
date: 2022-10-17
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `matrico` Module (Part III)

**Series**:

* [Part I](http://numerical-schemer.xyz/2022/09/16/matrico-matrico-i.html)
* [Part II](http://numerical-schemer.xyz/2022/09/30/matrico-matrico-ii.html)
* Part III
* [Part IIII](http://numerical-schemer.xyz/2022/11/30/matrico-matrico-iiii.html)

In this third part of the series of posts,
I will describe the various _map_ and _reduce_ specializations provided by the `matrico` module.
These are again contained in the [`src/mx.scm`](https://github.com/gramian/matrico/blob/main/src/mx.scm) Scheme source code file,
that is **included** into the module's main file `matrico.scm`.

### Map and Reduce

Map and reduce are two of the fundamental operations in [functional programming](https://en.wikipedia.org/wiki/Functional_programming):
The [map](https://en.wikipedia.org/wiki/Map_(higher-order_function)) function
applies a given unary function to each element in a container, such as a list, vector or matrix.
The [reduce](https://en.wikipedia.org/wiki/Fold_(higher-order_function)) (aka fold) function
applies a given binary function to an accumulator and sequentially each element in a container,
whereas this function's return value becomes the accumulator for the next evaluation.
These concepts of map and reduce have application beyond functional programming, such as in parallel programming,
and was mainstream popularized by the [MapReduce](https://en.wikipedia.org/wiki/MapReduce) programming model.

In `matrico`'s [matrix functor](https://numerical-schemer.xyz/2022/08/04/matrico-matrix.html), map and fold functions for `matrico`'s matrix type are provided,
which are specialized for typical numerical operations on `flonum` matrices in the `matrio` module.
These are summarized in three sections:
first the binary _expanders_, second the unary _mappers_, and lastly the reducers.

### Expanders

_Expanders_ refer in this context to binary map operations, which allow automatic broadcasting (aka implicit expansion).
This means the two matrix arguments do not need to be of the same dimension,
rather they have to be compatible dimensions, leading to the following five options:

* Both `matrix` arguments have the same number of rows and columns.
  The binary map function is applied to the arguments' respective entries with the same row and column index.
```
XXX   XXX   XXX
XXX o XXX = XXX
XXX   XXX   XXX
```

* One `matrix` argument is a column matrix, matching the column number of the other argument.
  The binary map function is applied as if the column matrix argument had as many columns as the other `matrix` arguments with all the same columns.
```
XXX   X   XXX     X   XXX   XXX
XXX o X = XXX  ,  X o XXX = XXX
XXX   X   XXX     X   XXX   XXX
```

* One `matrix` argument is a row matrix, matching the row number of the other argument.
  The binary map function is applied as if the row matrix argument had as many rows as the other `matrix` arguments with all the same rows.
```
XXX   XXX   XXX     XXX   XXX   XXX
XXX o     = XXX  ,      o XXX = XXX
XXX         XXX           XXX   XXX
```

* One `matrix` argument is a row matrix, the other is a column matrix.
  The binary map function is applied as if the row matrix argument had as many rows as the column matrix arguments with all the same rows, and the column matrix argument had as many columns as the row matrix arguments with all the same columns.
```
X   XXX   XXX     XXX   X   XXX
X o     = XXX  ,      o X = XXX
X         XXX           X   XXX
```

* One `matrix` argument is scalar.
  The binary map function is applied as if the scalar matrix or `flonum` had as many rows and columns as the other `matrix` argument all the same entries.
```
XXX       XXX         XXX   XXX
XXX o X = XXX  ,  X o XXX = XXX
XXX       XXX         XXX   XXX
```

#### Basic Arithmetic

* `(mx+ x y)` - Returns `matrix` of entry-wise sums of argument `matrix`es entries.
* `(mx* x y)` - Returns `matrix` of entry-wise differences of argument `matrix`es entries.
* `(mx- x y)` - Returns `matrix` of entry-wise products of argument `matrix`es entries.
* `(mx/ x y)` - Returns `matrix` of entry-wise divisions of argument `matrix`es entries.

#### Advanced Functions

* `(mx^ x y)`  - Returns `matrix` of entry-wise exponentiation of base `matrix` argument to exponent `matrix` argument.
* `(mx-log x y)` - Returns `matrix` of entry-wise logarithm to base `matrix` argument of anti-logarithm `matrix` argument.
* `(mx-where pred x y)` - Returns `matrix` of entries of first or second `matrix` argument, depending on binary predicate `function` argument evaluated on respective `matrix` arguments' entries.
* `(mx*+ a x y)` - Returns `matrix` of entry-wise sums of first `matrix` argument, scaled by `flonum` argument, with second `matrix` argument.

### Mappers

_Mappers_ describe an entry-wise processing of a matrix.
The unary map function is applied to each entry yielding a result matrix of the same dimensions. 
```
XXX    XXX
XXX -> XXX
XXX    XXX
```

#### Elementary Functions

* `(mx- x)` - Returns `matrix` of entry-wise negation of argument `matrix`es entries.
* `(mx/ x)` - Returns `matrix` of entry-wise reciprocal of argument `matrix`es entries.
* `(mx*2 x)` - Returns `matrix` of entry-wise double of argument `matrix`es entries.
* `(mx^2 x)` - Returns `matrix` of entry-wise square of argument `matrix`es entries.

#### Rounding Functions

* `(mx-round mat)` - Returns `matrix` of entry-wise roundings of argument `matrix`.
* `(mx-floor mat)` - Returns `matrix` of entry-wise greatest integer less than or equal to argument `matrix`.
* `(mx-ceil mat)` - Returns `matrix` of entry-wise least integer greater than or equal to argument `matrix`.

#### Generalized Functions

* `(mx-abs mat)` - Returns `matrix` being entry-wise absolute value of argument `matrix`.
* `(mx-sign mat)` - Returns `matrix` being entry-wise sign of argument `matrix`.
* `(mx-delta mat)` - Returns `matrix` being entry-wise Kronecker delta of argument `matrix`.
* `(mx-heaviside mat)` - Returns `matrix` being entry-wise Heaviside step of argument `matrix`.

#### Trigonometric Functions

* `(mx-sin mat)` - Returns `matrix` being entry-wise sine of argument `matrix`.
* `(mx-cos mat)` - Returns `matrix` being entry-wise cosine of argument `matrix`.
* `(mx-tan mat)` - Returns `matrix` being entry-wise tangent of argument `matrix`.

#### Arcus Functions (Inverse Trigonometric Functions)

* `(mx-asin mat)` - Returns `matrix` being entry-wise arcsine of argument `matrix`.
* `(mx-acos mat)` - Returns `matrix` being entry-wise arccosine of argument `matrix`.
* `(mx-atan mat)` - Returns `matrix` being entry-wise arctangent of argument `matrix`.

#### Hyperbolic Functions

* `(mx-sinh mat)` - Returns `matrix` being entry-wise hyperbolic sine of argument `matrix`.
* `(mx-cosh mat)` - Returns `matrix` being entry-wise hyperbolic cosine of argument `matrix`.
* `(mx-tanh mat)` - Returns `matrix` being entry-wise hyperbolic tangent of argument `matrix`.

#### Area Functions (Inverse Hyperbolic Functions)

* `(mx-asinh mat)` - Returns `matrix` being entry-wise area hyperbolic sine of argument `matrix`.
* `(mx-acosh mat)` - Returns `matrix` being entry-wise area hyperbolic cosine of argument `matrix`.
* `(mx-atanh mat)` - Returns `matrix` being entry-wise area hyperbolic tangent of argument `matrix`.

#### Haversed Trigonometric Functions

* `(mx-hsin mat)` - Returns `matrix` being entry-wise haversine of argument `matrix`.
* `(mx-hcos mat)` - Returns `matrix` being entry-wise havercosine of argument `matrix`.

#### Logarithmic Hyperbolic Functions

* `(mx-lnsinh mat)` - Returns `matrix` being entry-wise logarithmic hyperbolic sine of argument `matrix`.
* `(mx-lncosh mat)` - Returns `matrix` being entry-wise logarithmic hyperbolic cosine of argument `matrix`.

#### Squareroots

* `(mx-sqrt mat)` - Returns `matrix` being entry-wise squareroot of argument `matrix`.
* `(mx-signsqrt mat)` - Returns `matrix` being entry-wise sign times squareroot of absolute value of argument `matrix`.

#### Logarithms

* `(mx-ln mat)` - Returns `matrix` being entry-wise natural logarithm of argument `matrix`.
* `(mx-lb mat)` - Returns `matrix` being entry-wise common logarithm of argument `matrix`.
* `(mx-lg mat)` - Returns `matrix` being entry-wise binary logarithm of argument `matrix`.

#### Exponentials

* `(mx-exp mat)` - Returns `matrix` being entry-wise exponential of argument `matrix`.
* `(mx-gauss mat)` - Returns `matrix` being entry-wise Gaussian of argument `matrix`.

#### Special Functions

* `(mx-sinc mat)` - Returns `matrix` being entry-wise cardinale sine of argument `matrix`.
* `(mx-sigm mat)` - Returns `matrix` being entry-wise sigmoid of argument `matrix`.
* `(mx-stirling mat)` - Returns `matrix` being entry-wise Stirling approximation of argument `matrix`.

### Reducers

_Reducers_ compose a matrix's entries yielding a `matrix` of lower dimension.
There are three variants:

* Composition of row entries yielding a column `matrix` by _fold_ ing each row into a scalar.
```
XXX    X
XXX -> X
XXX    X
```
* Composition of column entries yielding a row `matrix` by _fold_ ing each column into a scalar.
```
XXX    XXX
XXX ->
XXX
```
* Composition of all entries yielding a `flonum` by _fold_ ing all entries into a scalar.
```
XXX
XXX -> X
XXX
```

#### Sums

* `(mx-rowsum mat)` - Returns column `matrix` of row sums of argument `matrix`.
* `(mx-colsum mat)` - Returns row `matrix` of column sums of argument `matrix`.
* `(mx-sum mat)` - Returns `flonum` sum of all `matrix` argument's entries.

#### Products

* `(mx-rowprod mat)` - Returns column `matrix` of row products of argument `matrix`.
* `(mx-colprod mat)` - Returns row `matrix` of column products of argument `matrix`.
* `(mx-prod mat)` - Returns `flonum` product of all `matrix` argument's etnries.

#### Minima

* `(mx-rowmin mat)` - Returns column `matrix` of row minima of argument `matrix`.
* `(mx-colmin mat)` - Returns row `matrix` of column minima of argument `matrix`.
* `(mx-min mat)` - Returns `flonum` minimum over all `matrix` argument's entries.

#### Maxima

* `(mx-rowmax mat)` - Returns column `matrix` of row maxima of argument `matrix`.
* `(mx-colmax mat)` - Returns row `matrix` of column maxima of argument `matrix`.
* `(mx-max mat)` - Returns `flonum` maximum over all `matrix` argument's entries.

#### Midrange

* `(mx-rowmidr mat)` - Returns column `matrix` of row midranges of argument `matrix`.
* `(mx-colmidr mat)` - Returns row `matrix` of column midranges of argument `matrix`.
* `(mx-midr mat)` - Returns `flonum` midrange over all `matrix` argument's entries.

#### Generalized Means

* `(mx-rowmean mat typ)` - Returns column `matrix` of row power-means of argument `matrix` of power `symbol` argument.
* `(mx-colmean mat typ)` - Returns row `matrix` of column power-means of argument `matrix` of power `symbol` argument.
* `(mx-mean mat typ)` - Returns `flonum` power-mean over all `matrix` argument's entries of power `symbol` argument.

#### Vector and Matrix Norms

* `(mx-rownorm mat typ)` - Returns column `matrix` of row norms of argument `matrix` of type `symbol` argument.
* `(mx-colnorm mat typ)` - Returns row `matrix` of column norms of argument `matrix` of type `symbol` argument.
* `(mx-norm mat typ)` - Returns `flonum` matrix norm over all `matrix` argument's entries of type `symbol` argument.

Overall, these functions make up the basic matrix transformation operations,
which in `matrico` are specializations of the generic _map_ and _reduce_ matrix functions.
In the next post, I will go through the core linear algebra and analysis functions of `matrico`.

