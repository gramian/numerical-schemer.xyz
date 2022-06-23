---
title: "CHICKEN flonum Module"
date: 2022-06-23
tags: Scheme matrico
---

## CHICKEN Scheme's included `flonum` module

This is another short guided tour, this time through the functions of the CHICKEN Scheme `flonum` module.
The purpose of this module is, similar to the [`fixnum` module](http://wiki.call-cc.org/man/5/Module%20(chicken%20fixnum)), to provide typed operations for floating-point numbers for improved performance.
Additionally, in my opinion, using these functions increases readability, as the use of these functions underscores (and enforces) the type of their arguments.
For the full reference, see the [module (chicken flonum)](http://wiki.call-cc.org/man/5/Module%20(chicken%20flonum)).

### Arithmetic Operations
Essentially, this module provides specialized functions for all mathematical functions defined in R5RS,
yet, with notable exceptions:
The elementary arithmetic operations only and only accept two arguments.
On the one hand, this means unary variants are not available.
So, the subtraction function `(fp- x y)` cannot be used to negate and the division function `(fp/ x y)` cannot be used to invert with only one argument.
Of course, one could curry the respective neutral element to these functions as first argument (`(fp- 0.0 y)` and `(fp/ 1.0 y)`),
and for negation the function `fpneg` is supplied.
On the other hand, no variadic function calls are allowed, for example a sum with more than two arguments,
as is for the (R5RS) standard elementary arithmetic functions.
This is unfortunate, but I assume that a performance argument applies here, too;
specifically, a mapping from the `flonum` functions to `C`'s functions is targeted.
Similarly, the flonum specific comparison functions `fp=`, `fp<`, `fp>`, `fp<=`, `fp>=`,
as well as minimum and maximum functions `fpmin` and `fpmax`, are fixed to two arguments.

Interestingly, The functions in the `flonum` module are prefixed with `fp` rather than `fl` as many other Scheme implementations,
such as:

* [Chez](https://scheme.com/tspl4/objects.html#g112),
* [Gambit](http://www.gambitscheme.org/latest/manual/#Flonum-specific-operations),
* [Gauche](https://practical-scheme.net/gauche/man/gauche-refe/R7RS-large.html#scheme_002eflonum-_002d-R7RS-flonum),
* [Guile](https://www.gnu.org/software/guile/manual/html_node/rnrs-arithmetic-flonums.html),
* or [Racket](https://docs.racket-lang.org/reference/flonums.html).

Given the Gauche documentation, the `fl` prefix seems to relate to R7RS, so maybe CHICKEN flonums highlights in this way being R5RS,
or it was just a timing issue.

Furthermore, [hyperbolic functions](https://en.wikipedia.org/wiki/Hyperbolic_functions)

* hyperbolic sine (`sinh`),
* hyperbolic cosine (`cosh`),
* hyperbolic tangent (`tanh`),

and [inverse hyperbolic functions](https://en.wikipedia.org/wiki/Inverse_hyperbolic_functions)

* area hyperbolic sine / inverse hyperbolic sine (`asinh`),
* area hyperbolic cosine / inverse hyperbolic cosine (`acosh`), 
* area hyperbolic tangent / inverse hyperbolic tangent (`atanh`),

are not included in this module.
While these functions are not part of R5RS (which I think is a shortcoming),
they are defined in the `C` [math library](https://cplusplus.com/reference/cmath/),
and even though there are closed forms for these functions, their (flonum-)accurate implementation,
is not necessarily that simple
(unless exponential and natural logarithm `fpexp` and `fplog` switch to appropriate underlying `C` functions based on their arguments).

### Print Precision
Lastly, the `(flonum-print-precision num)` function controls how flonums are printed to the terminal or a sequential file.
This function is cleverly designed by always returning the current precision,
so temporarily setting another print precision is facilitated.
However, this function enforces only a minimum precision in characters.
If more characters than given by this precision are needed to print a certain number, more characters are used.

