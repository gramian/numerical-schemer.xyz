---
title: "Matrico utils Module"
date: 2022-06-12
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s Utilities Module `utils`

This is the base module of the `matrico` project and imported by every other module.
It contains a collection of miscellaneous bindings, macros and functions for general use.
You can take a look at the `src/utils.scm` source file [here](https://github.com/gramian/matrico/blob/main/src/utils.scm).

### Meta Helpers
Scheme's macro system is insanely powerful.
However, I often only need a handful of macro helpers.
First, the generic `define-syntax` is to bulky if you only have a single rule;
for this, I use a macro that makes the syntax definition look like a function definition, inspired by [Guile](https://www.gnu.org/software/guile/manual/html_node/Syntax-Rules.html#Shorthands).
Second, I need to rename bindings on-the-fly, for which I use a simple macro also PC Scheme had.
Also, I require a shortcut for asserting a variable argument list of tests, typically to check argument types and properties.
There are two reason for not using a function here: asserts are removed by the CHICKEN compiler in "unsafe"-mode, so this might litter,
and, I am using `and` which I cannot pass into `apply` since it is special.
Lastly, I was intruiged by Clojure's [rich comments](https://clojuredocs.org/clojure.core/comment), which were already part of the [T language](http://mumble.net/~jar/tproject/).
Altogether:

* `(define-syntax-rule )` - Wrapper for single rule macros.
* `(alias aka name)` - Replaces first argument with second argument.
* `(must-be arg ...)` - Wrapper for `assert` of `and` with variable number arguments.
* `(comment ...)` - S-expression comment, all content is ignored and void is returned.

The `comment` macro will not be found in the release code bases, I just use it during development.

### List Aliases
I never warmed up to Lisps' `car` and `cdr` function names, even given the
sometimes useful predefined compositions of `car`s and `cdr`s (See
[`car` and `cdr`](https://en.wikipedia.org/wiki/CAR_and_CDR) for more info).
I prefer StandardML's "head" (`hd`) and "tail" (`tl`), and even if
[not all cdrs are tails](https://irreal.org/blog/?p=8500), every coin has a pair
of heads and tails. 
Then, Lisp has `nil` as a name for the empty list,
and I prefer three letters to three symbols typing-wise.
I note here, that in Lisp the empty list also is a boolean false value, while in Scheme only and only `#f`
is false. Finally, I find [Racket](https://docs.racket-lang.org/reference/pairs.html#%28def._%28%28lib._racket%2Flist..rkt%29._empty~3f%29%29)'s
`empty?` is much more readable then `null?`. Thus the following names are bound:

* `nil` - Name for the empty list symbol `'()`.
* `(head lst)` - Alternate name for `car` returning first member of argument pair.
* `(tail lst)` - Alternate name for `cdr` returning second member of argument pair.
* `(empty? lst)` - Alternate name for `null?` returning true if tail of argument pair is nil.

These are bindings and can't use the `alias` macro, otherwise they couldn't be passed as functions.

### Fixnum helpers
For performance and readability reasons I want to ensure and document that for
indexing operations fixed-size integers and integer operations are used. CHICKEN
has the matching [(chicken fixnum)](http://wiki.call-cc.org/man/5/Module%20(chicken%20fixnum))
module for this purpose. However, I needed to add a few functions, like 
increment and decrement as well as predicates comparing to zero:

* `(fx+1 x)` - Increment fixnum argument.
* `(fx-1 x)` - Decrement fixnum argument.
* `(fx=0? x)` - Returns true if fixnum argument is zero, false otherwise.
* `(fx<0? x)` - Returns true if fixnum argument is less than zero, false otherwise.
* `(fx>0? x)` - Returns true if fixnum argument is greater than zero, false otherwise.
* `(fx<=0? x)` - Returns true if fixnum argument is less or equal to zero, false otherwise.
* `(fx>=0? x)` - Returns true if fixnum argument is greater or equal to zero, false otherwise.

As integers are so fundamental for programming, I let the `utils` module
`reexport` the `(chicken fixnum)` module.

### List Functions
Rarely, one needs to extract a part of list between two indices, similar to
the non-standard but prevalent `subvector` for vectors. 
Furthermore, heuristically it turned out using folding via (`foldr`) instead of
`append` to append an element to a list is more memory efficient, thus a
fold-based append variant is supplied.

Unlike in [SRFI-1](https://srfi.schemers.org/srfi-1/srfi-1.html), my `any?` and
`all?` ("all" instead of "every") functions return boolean values, and thus have
question marks signifying a predicate. These leads to:

* `(append* lst any)` - Returns argument list appended with element argument.
* `(sublist lst start end)` - Returns list partition of list argument inbetween the index arguments.
* `(any? pred lst)` - Returns true if any element of list argument applied to function argument is true, false otherwise.
* `(all? pred lst)` - Returns true if all elements of list argument applied to function argument are true, false otherwise.

### Factorials
A classic example for recursive computation is the factorial.
Here, it is implemented to be able to test another function from another module.
Furthermore, the related binomial cofficient is provided where the implemented algorithm,
[Pascal's rule](https://en.wikipedia.org/wiki/Pascal%27s_rule), hints at its use.

* `(factorial n)` - Returns product of consecutive integers up to argument, starting at one.
* `(binomial n k)` - Returns binomial coefficient for given integer arguments.

Note, that a computation of the binomial coefficient via its iterative
definition (using the factorial) is very inefficient.

### Standard Variants
Scheme, particularly CHICKEN Scheme, is awesome. Whatever you can imagine, be it
docstrings or a data-file loading, one can implement with a few lines of code:

* `(define* ...)` - Defines a binding together with a docstring for various forms of `define`.
* `(load* pathstr)` - Loads and evaluates file a argument string path and returns the result of the last expression.

While `define*` is a language extension via a pattern matching macro, `load*` is a function using `eval`.

