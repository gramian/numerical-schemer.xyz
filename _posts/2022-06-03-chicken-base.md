---
title: "CHICKEN base Module"
date: 2022-06-03
tags: Scheme matrico
---

## CHICKEN Scheme's included `base` module

This is a short guided tour through the functions of the CHICKEN Scheme `base` module,
from my point of view. This summary only highlights and is not comprehensive.
For the full reference, see the [Module (chicken base)](http://wiki.call-cc.org/man/5/Module%20(chicken%20base)) documentation.

### Numeric predicates

First, the predicates for numeric types are an important addition, i.e.:

* `(fixnum? x)` - Is argument a fixed-size integer?
* `(flonum? x)` - Is argument an inexact floating-point number?

And can be used to guard functions using specific fixnum or flonum functionality against generic numbers.

### Little Schemer

Recursion can require counting, particularly when handling indexed types such as vectors.
Furthermore, one may want to ask if something is an atom instead of asking if it is not a pair:

* `(add1 x)` - Return argument incremented by one.
* `(sub1 x)` - Return argument decremented by one.
* `(atom? x)` - Is argument not a pair?

Even though given `+`, `-`, and `pair?`, I wonder, why these very convenient functions have not found their way into R5RS,
especally due to their prominent use in [The Little Schemer](https://mitpress.mit.edu/books/little-schemer-fourth-edition).

### Special Function

A mathematical function closely related to the absolute value is the sign function:

* `(sign x)` - Return argument divided by its absolute value, except for zero return zero.

Since at least one conditional for testing zero is required anyway,
this is likely implemented as a three-way conditional `cond` without division.

### Standard Output

Scheme has `display` to output text, but for generality it does not terminate the output with an end-of-line,
as one often needs:

* `(print x ...)` - Output arguments to standard output and appends a newline sequence.

### Standard Error

R5RS misses functions to abort execution and report an error, and derived thereof
a function to conditionally halt:

* `(error sym msg)` - Abort evaluation and print symbolic location argument and error message string argument.
* `(assert expr)` - Raise error if expression argument evaluates to false.

I mostly, if not exclusively, use `assert` to test function arguments for validity.

### Reducers

While R5RS provides a `map` function, a "fold" function is not part of the standard,
Therefore these reducers are included:

* `(foldl fun ini lst)` - Apply binary function argument pair-wise to argument initialized accumulator and elements of list argument head to tail.
* `(foldr fun ini lst)` - Apply binary function argument pair-wise to argument initialized accumulator and elements of list argument tail to head.

### Vector Utility

Extracting a part of an array is a typical task which could be solved by recursion as for lists,
but for indexable (memory continguous) types like arrays (vectors) a block extraction can be more natural:

* `(subvector vec from to)` - Return vector holding elements of argument vector in given index range.

### Abstract Functions

The following functions are simple: 

* `(identity x)` - Returns argument.
* `(constantly x)` - Returns function returning argument for any argument.
* `(compose f g ...)` - Returns function applying first argument to result of second, and so on.

However, importing these tools from an official module prevents one from rewriting it over and over, and establishes a name for them.

### One-Armed Conditional

In Scheme it is legal to use an `if` only with consequent and without alternative,
a so-called one-armed-if. But what is returned if the one-armed-if's predicate
is false? Well, nothing:

* `(when pred expr)` - Evaluate expression argument if predicate is true.
* `(void any ...)` - "Nothing" return value

This `(void)` is what procedures - functions without return value - like `print` return.

### Variable Arity

To define procedures with variable number of arguments the rest argument can be used,
but leaves one with the task to deconstruct and match this list.
More convenient is directly matching signature patterns, here via [SRFI-16](https://srfi.schemers.org/srfi-16/srfi-16.html):

* `(case-lambda ((arg ...) body ...) ((arg ...) body ...))`

### Substitutions

Scheme has a powerful macro system, but sometimes, not least for readability,
a middle ground between function and macro, or binding and literal is needed:

* `(define-constant name val)` - Compile-time evaluated binding.
* `(define-inline (name arg ...) body ...)` - Guaranteed inlined function local to a compile unit.

### Structures

Structures (or records) are not only useful to compound data, but also to encapsule and thus protect
data elements against random access:

* `(define-record name field1 ...)` - Defines a structure.

This is somewhat similar to an object definition, as a constructor (`make-name`),
a predicate (`name?`), and field getters (`name-field1`, ...) are automatically provided.

### Source Organization

Scheme has `load` to access functionality from other source files, yet `load` already evaluates the argument file.
An alternative is including a source file (without evaluation), and evaluate when the including file is evaluated:

* `(include string-path)` - Paste contents of file at absolute path in string argument.
* `(include-relative string-path)` - Paste contents of file at relative path (to current file) in string argument.

One cannot overstate the utility of the relative file path variant for code organization.


As one can see, the contents of the `base` module are largely simple convenience function,
but significantly decrease boilerplate and increase readability.
This included module makes CHICKEN Scheme practically very useful.


