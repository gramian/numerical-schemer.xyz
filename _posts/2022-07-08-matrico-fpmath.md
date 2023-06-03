---
title: "Matrico fpmath Module"
date: 2022-07-08
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s Floating-Point Addon-Module `fpmath`

To extend the flonum functionality of CHICKEN Scheme and particularly the
`(chicken flonum)` module, an addon-module named `fpmath` is implemented,
that supplies various flonum specific functions for `matrico` specific, but also
general, use. And the `(chicken flonum)` naming of prefixing functions with `fp`
is utilized.

As the `fpmath` module is not very useful without the imported
`(chicken flonum)` module, it is reexported. And as the `fpmath` module is
also practically useful for high-level use in the overarching `matrico` module,
`matrico` in turn reexports `fpmath`.
You can take a look at the `src/fpmath.scm` source file [here](https://github.com/gramian/matrico/blob/main/src/fpmath.scm).

### Converters
To bridge between the exact fixnum numbers and the inexact flonum numbers,
converters are needed. R5RS provides a fixnum to flonum conversion, but its name
`inexact->exact` is too unwieldy, thus a shorter alias `fp` is introduced.
Another very useful converter is, given two fixnums representing numerator and
denominator, a conversion to a flonum from such rational components, which is
used below for constant provisioning (inspired by [fp-utils](http://wiki.call-cc.org/eggref/5/fp-utils#fp)).

* `(fp n)` - Alias for `exact->inexact`, returns inexact flonum from exact fixnum argument.
* `(fp% n d)` - Returns flonum from numerator fixnum and denominator fixnum arguments.

### Predicates
A tricky issue is comparing flonums for equality, due to their inexactness.
However, any equality comparison boils down to a comparison against zero, in
example via the difference of two flonums. To handle this, two functions are
implemented, one to compare against exact zero, which is emphasized by two
question-mark predicate indicators, and one to compare against an absolute tolerance:

* `(fpzero?? x)` - Returns true for zero, else false.
* `(fpzero? x tol)` - Returns true if first argument is less than second argument, else false.

### Operators
The `(chicken flonum)` module only provides one unary elementary arithmetic
operations, namely `fpneg`, the unary shortcut for `(fp- 0.0 x)`. For division,
addition, and multiplication no unary operations are defined. These are added
here: In terms of division, the multiplicative inverse `(fp/ 1.0 x)` is defined,
for addition the double, and for multiplication the square of the argument.

Also, an operation central to numerical computation, for example in the
dot-product, and thus matrix multiplication, matrix decompositions, like the
QR decomposition, or integrators such as Runge-Kutta methods, is included.
This three operand operation of a combined flonum multiplication and addition
is classically known as a [Flop](https://en.wikipedia.org/wiki/FLOPS) (short for
FLoating-point OPeration) or ["axpy" in BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms#Level_1),
and technically as [FMA (Fused Multiply-Add) in CPUs,
or MAD (Multiply-Add) in GPUs](https://en.wikipedia.org/wiki/Multiply%E2%80%93accumulate_operation).
Also, this operation has improved accuracy, as only the final result is rounded to the closest flonum.
This operation is important enough to be provided in the [`C` library](https://cplusplus.com/reference/cmath/fma/),
and also in the flonum modules of various Schemes such as [Gauche](https://practical-scheme.net/gauche/man/gauche-refe/R7RS-large.html#index-fl_002b_002a)
and [MIT Scheme](https://www.gnu.org/software/mit-scheme/documentation/stable/mit-scheme-ref.html#Flonum-Operations).

* `(fp*2 x)` - Returns the double of the flonum argument.
* `(fp^2 x)` - Returns the square of the flonum argument.
* `(fprec x)` - Returns the reciprocal of the flonum argument.
* `(fp*+ x y z)` - Returns the fused multiply-add of the flonum arguments.

In future versions of CHICKEN Scheme (maybe version 5.4), `(chicken flonum)`
will include the `fp*+` function, hence it is wrapped in a CHICKEN version
dependent `cond-expand`.

### Constants
The `(chicken flonum)` module does not provide any of the typical mathematical
constants. So, the `fpmath` module fills this gap by defining the three most
important real-valued constants (in my opinion, outside of: 0, 1, √2) as thunks
(zero argument functions):

* A circle constant: **tau** (exclusively), and (particularly) [not **pi**](https://tauday.com/tau-manifesto).
* Euler's number: **e**
* Golden ratio: **phi**

I am using thunks, because I am not storing the constants' value to precision
in-source, but computing them as the module loads. This could be done, in
example, for **tau** via `(8.0 * (atan 1.0))`, for **e** via `(exp 1.0)`,
or for **phi** `(* (+ 1.0 (sqrt 5.0)) 0.5)`. However, there are two issues with
this: First, how would I know that these are correctly computed, so the
functions are correct and the formulas are not suffering from numerical effects
like annihilation. And second, I would need to call functions whose actual
complexity I don't know. Also, I want to use only a single elementary arithmetic
function to approximate these constants. Since floating-point numbers are of
finite precision, I will use rational approximation ([like `355/113` for **pi**](http://davidbau.com/archives/2010/03/14/the_mystery_of_355113.html))
and thus a mere single division operation. Luckily, this is facilitated by the
"[On-Line Encyclopedia of Integer Sequences](https://oeis.org)" which provides
numerators, denominators, and number of correct decimal digits, such that a
rational approximation with accuracy matching flonums can be selected, and
tested (verified) against the closed forms mentioned above:

* `(fptau)` - Returns `(fp% 491701844 78256779)`, see: [https://oeis.org/A002485](A002485) , [https://oeis.org/A002486](A002486) , and [https://oeis.org/A114526](A114526) (17th entry, for 17 correct digits).
  Note, that I have pre-multiplied the numerator with `2`, but this is a precise integer operation.
* `(fpeul)` - Returns `(fp% 410105312 150869313)`, see: [https://oeis.org/A007676](A007676) , [https://oeis.org/A007677](A007677) , and [https://oeis.org/A114539](A114539) (21st entry, for 16 correct digits).
* `(fpphi)` - Returns `(fp% 165580141 102334155)`, see: [https://oeis.org/A000045](A000045) , computed via [consecutive quotients of Fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_number#Limit_of_consecutive_quotients) (41st, and 40th entries for 16 correct digits).

I have not seen use of rational approximations in the context of approximate 
floating-point constants in numerics before, so maybe somebody has some pointers
to other projects using this? Anyway, I think this is a nice exploitation of
flonum inexactness.

### Generalized Functions
Beyond the absolute value function `fpabs` in `(chicken flonum)` there are some
further "generalized functions" (in simple terms, these mathematical mappings
are generalized because they require conditionals, due to their piecewise
defintion) I consider useful:

* `(fpdelta x)` - Returns zero except for argument zero it returns one, see [Delta](https://en.wikipedia.org/wiki/Kronecker_delta#Alternative_notation).
* `(fpheaviside x)` - Returns one for positive arguments zero otherwise, see [Heaviside](https://en.wikipedia.org/wiki/Heaviside_step_function).
* `(fpsign x)` - Returns minus one for negative, one for positive, and zero for zero arguments, see [Sign](https://en.wikipedia.org/wiki/Sign_function).

### Logarithms
The `(chicken flonum)` module only provides a natural logarithm `fplog`, which
is the minimal necessary, but not too comfortable. So I am adding typical
calculator functionality (in their naming), based on the [logarithm rules](https://en.wikipedia.org/wiki/Logarithm#Change_of_base):

* `(fpln x)` - Alias for `fplog`, returns natural logarothm of flonum argument.
* `(fplb x)` - Returns binary logarithm (base 2) of flonum argument.
* `(fplg x)` - Returns decimal logarithm (base 10) of flonum argument.
* `(fplogb b x)` - Returns logarithm to **positive** flonum base argument of flonum argument.

Note, that the utilized values `ln(2)` (`(fplog 2.0)`) and `ln(10)` (`(fplog 10.0)`) are neither
hard-coded as constants nor computed during a function call to `fplb` or `fplg`, but pre-computed once on load of the module. 

### Hyperbolic Functions and Inverse Hyperbolic Functions
As I stated in an [earlier post](http://numerical-schemer.xyz/2022/06/23/chicken-flonum.html),
a shortcoming of R5RS and also `(chicken flonum)` is the absence of hyperbolic
functions and inverse hyperbolic functions. So, these are provided in this addon
module via the [exponential definitions](https://en.wikipedia.org/wiki/Hyperbolic_functions#Exponential_definitions) of the hyperbolic functions and the
[logarithmic definitions](https://en.wikipedia.org/wiki/Inverse_hyperbolic_functions#Definitions_in_terms_of_logarithms) of the inverse hyperbolic functions:

* `(fpsinh x)` - Returns hyperbolic sine for flonum argument.
* `(fpcosh x)` - Returns hyperbolic cosine for flonum argument.
* `(fptanh x)` - Returns hyperbolic tangent for flonum argument.
* `(fpasinh x)` - Returns area hyperbolic sine (inverse hyperbolic sine) for flonum argument.
* `(fpacosh x)` - Returns area hyperbolic cosine (inverse hyperbolic cosine) for flonum argument.
* `(fpatanh x)` - Returns area hyperbolic tangent (inverse hyperbolic tangent) flor flonum argument.

### Haversed Trigonometric Functions
A lesser used variant of the trigonometric functions are the [haversed sine
(haversine) and haversed cosine (havercosine)](https://en.wikipedia.org/wiki/Versine#Haversine). These scaled
and shifted sine and cosine are sometimes useful in systems and control theory,
my field of mathematical research.

* `(fphsin x)` - Returns haversine for flonum argument.
* `(fphcos x)` - Returns havercosine for flonum argument.

### Logarithmic Hyperbolic Functions
Another pair of lesser known functions are logarithmic hyperbolic functions,
which are implemented in the [GNU Scientific Library](https://www.gnu.org/software/gsl/doc/html/specfunc.html#hyperbolic-trigonometric-functions).
While log-sinh is a differentiable function over the positive real numbers
that behaves like a logarithm for small (positive) values and like a linear function for large (positive) values,
the log-cosh is a differentiable function over all real numbers,
that behaves like a quadratic function for small values and like the absolute value for large values.

* `(fplnsinh x)` - Returns log-sinh for flonum argument.
* `(fplncosh x)` - Returns log-cosh for flonum argument.

### Special Functions
Some useful extra functions are implemented:

* `(fpsignsqrt x)` - Returns the square root of the absolute value of the flonum argument times the sign of the argument.
* `(fpsinc x)` - Returns the (unnormalized) [cardinale sine](https://en.wikipedia.org/wiki/Sinc_function) of the flonum argument.
* `(fpsigm x)` - Returns the [sigmoid](https://en.wikipedia.org/wiki/Sigmoid_function) (standard logistic function) of the flonum argument.
* `(fpgauss x)` - Returns the standard [Gauss function](https://en.wikipedia.org/wiki/Stirling%27s_approximation) (zero expectation, unit deviation) of the flonum argument.
* `(fpstirling x)` - Returns the [Stirling's approximation](https://en.wikipedia.org/wiki/Stirling%27s_approximation) for factorials of the flonum argument.

Note, that `fpsinc` can be normalized by scaling its argument,
`fpsigm` and `fpgauss` can be generalized by shifting and scaling its argument and also scaling its result,
while `fpstirling` can be tested with the [`utils` module](http://numerical-schemer.xyz/2022/06/12/matrico-utils.html)'s `factorial` function.

### Utilities
To print matrices on the terminal a fixed element print width is necessary.
Given the `flonum-print-precision` from `(chicken flonum)`, this is only
workable with a precision of `17`, which in turn would mean on a standard 80x25
terminal window that only a matrix with 4 columns could be printed without
additional breaks. Now, with a maximum width of `8` already a 9-column matrix
becomes printable. This tapered flonum to string conversion is implemented by:

* `(fptaper x)` - Returns 8-character string approximately describing flonum argument.

Practically this means the integer and fractional parts can occupy maximally `6`
characters, as one character is reserved for the sign and another for the
decimal point. So the smallest fully printable number is `±0.00001`, where as the
largest is `±99999.9`. Any smaller number is partially printed as `±0.000…` and
larger numbers are printed as `±100000.` or `±100000…` respectively. To highlight
an exact zero, it is printed as `____0___`. Special values are printed as:
`___±∞___` and `___NaN__`.

However, as this function is supposed only for internal use, it is excluded from
reexport in `matrico`.

