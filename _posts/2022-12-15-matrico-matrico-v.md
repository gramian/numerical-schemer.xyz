---
title: "Matrico Module V"
date: 2022-12-15
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `matrico` Module (Part V)

* [Part I](http://numerical-schemer.xyz/2022/09/16/matrico-matrico-i.html)
* [Part II](http://numerical-schemer.xyz/2022/09/30/matrico-matrico-ii.html)
* [Part III](http://numerical-schemer.xyz/2022/10/17/matrico-matrico-iii.html)
* [Part IIII](http://numerical-schemer.xyz/2022/11/30/matrico-matrico-iiii.html)
* Part V

In this last part of the five part series on the main `matrico` module, I
outline the utility functions as well as the private functions.
These are again contained in the [`src/mx.scm`](https://github.com/gramian/matrico/blob/main/src/mx.scm) Scheme source code file,
that is **included** into the module's main file `matrico.scm`.

### Utility Functions

Up to this post in this series, only functions that instantiate or process matrices were described.
Here, the functions enabling input and output of matrices are described.
In terms of output, `matrico`'s matrices can be printed to the REPL console (stdout),
or saved to file, either in `matrico`'s native ([s-expression](https://en.wikipedia.org/wiki/S-expression)) format or in [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) format.
The former directly maps to `matrico`'s internal column-major matrix format in serialized form,
the latter is a portable format compatible with [MATLAB](https://www.mathworks.com/help/matlab/ref/readmatrix.html), [Octave](https://octave.sourceforge.io/octave/function/csvread.html), [NumPy](https://numpy.org/doc/stable/reference/generated/numpy.loadtxt.html), [Julia](https://csv.juliadata.org/stable/reading.html) or [R](https://www.rdocumentation.org/packages/utils/versions/3.6.2/topics/read.table).
Input-wise, a matrix in `matrico`'s format can be loaded from file.

* `(mx-print mat)` - Returns **void**, prints argument **matrix** to stdout.
* `(mx-export str mat)` - Returns **void**, saves argument **matrix** to file at path **string** argument in CSV format.
* `(mx-save str mat)` - Returns **void**, saves argument **matrix** to file in `matrico` format at argument **string** path.
* `(mx-load str)` - Returns **matrix** loaded from file in `matrico` format at argument **string** path.

### Private Functions

In the `matrico` module, recurring expressions are abstracted into local (non-user-facing) functions.
There are four kinds of abstractions:
predicates testing if an argument is a matrix or flonum,
converters ensuring a matrix or flonum is returned,
converters from one-based frontend indexing allowing positive and negative indices (denoting counting from first entry forward and form last element backward respectively) to zero-based positive-only backend indexing,
and a single-step time stepper for Runge-Kutta method variants.

* `(matrix-or-flonum? val)` - Returns **boolean** answering if argument is a **matrix** or **flonum**.
* `(scalar-or-flonum? val)` - Returns **boolean** answering if argument is a scalar **matrix** or **flonum**.
* `(column-or-flonum? val)`  - Returns **boolean** answering if argument is a column **matrix** or **flonum**.
* `(ensure-mx val)` - Returns **matrix** for **matrix** or **flonum** argument.
* `(ensure-fp val)` - Returns **flonum** for **flonum** or scalar **matrix** argument.
* `(translate-cols mat idx)` - Returns **fixnum** zero-based, positive index from one-based, positive/negative column index **fixnum** argument for **matrix** argument.
* `(translate-rows mat idx)` - Returns **fixnum** zero-based, positive index from one-based, positive/negative row index **fixnum** argument for **matrix** argument.
* `(time-stepper typ sys tim x0)` - Returns **matrix** approximating ordinary differential equation by **procedure** argument method with explicit vector field (and optionally output function) (**pair**-of-)procedure argument over time discretization **pair**-of-**flonum**s argument, and starting at column **matrix** argument initial state.

This concludes the summary of the `matrico` module.
In the next post I will reflect on CHICKEN Scheme and environment.
