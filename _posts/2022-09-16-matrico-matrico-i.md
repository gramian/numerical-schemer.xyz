---
title: "Matrico Module I"
date: 2022-09-16
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `matrico` Module (Part I)

**Series**:

* Part I
* [Part II](http://numerical-schemer.xyz/2022/09/30/matrico-matrico-ii.html)

The `matrico` module is the user-facing part of the `matrico` project.
Over the next few posts, I will describe the available functions therein.
Almost all functions in this module are prefixed with "`mx`",
with the exception of the `matrico` function, implemented in the [`matrico.scm`](https://github.com/gramian/matrico/blob/main/matrico.scm) source file.
While the functions prefixed with "`mx`" are operating on, or returning, matrices,
the `matrico` function is a metadata function providing information about `matrico` itself (or the current machine).
I start here by describing this "`matrico`"-function:

On the one hand, the `matrico` function is a [thunk](https://en.wikipedia.org/wiki/Thunk#Functional_programming)
(zero argument function) that explains its own use, so knowing this one function allows a REPL user to explore
`matrico` in a self-contained manner:

* `(matrico)` - returns void, prints a short help on the use of the `matrico` function with an symbol argument.

On the other hand, the `matrico` function is also a one-argument function taking a symbol argument.
Below, the specific symbols recognized are listed.
First, functionality to list all of `matrico`'s "`mx`"-prefixed functions is given:

* `(matrico 'list)` - returns void, prints list of all matrix processing functions.

Next, a way to print a minimal description of `matrico` and a link to the user-facing documentation is provided:

* `(matrico 'about)` - returns void, prints summary of `matrico`.

Also, printing the `matrico` logo, used on-load of the module (egg) is encapsuled:

* `(matrico 'banner)` - returns void, prints the `matrico` logo, name, and version.

The `matrico` version can also be returned by the `matrico` function:

* `(matrico 'version)` - returns pair of integers representing major and minor version semantic number in the head (`car`) and tail (`cdr`) respectively.

A pair of integer numbers is returned instead of a string, typically used in CHICKEN Scheme for versions,
so they can directly be compared.

Next, is a symbol argument that causes returning information on how to cite this module.
I included this due to my experiences in research software development,
so in the (unlikely) case somebody uses `matrico` for science, it is clear how to cite it:

* `(matrico 'citation)` - returns void, prints preferred citation string for `matrico`.

Lastly, I was contemplating how to classify the hardware `matrico` is running on.
Initially, I looked into the included [(chicken platform)](http://wiki.call-cc.org/man/5/Module%20(chicken%20platform)) module,
which did not provide an adequate function, as I am interested in a (rough) performance assessment.
Another option is to read out a processor identifier,
yet this has to be adapted for every operating systems, i.e. Linux (`lscpu`), MacOS (`sysctl`), Windows (`wmic`),
and even if one would be willing to maintain this, a performance estimate has to be deduced by looking up capabilities
in the chip's (online) documentation.
The Linux kernel presented a workable alternative: It tests the processor's speed on boot by looping through an empty loop
for a high number of times  (so called `busy wait`) and measuring the elapsed time.
This measure is called [BogoMips](https://en.wikipedia.org/wiki/BogoMips),
for "Bogus" million-instructions-per-second, and also implemented in `matrico`:

* `(matrico 'benchmark)` - returns number of busy-wait million-instructions-per-second.

I note, as have other commentators, that this "benchmark" is rather crude, nonetheless,
and particularly since CHICKEN Scheme is single-threaded, this function gives a hint on the processor's throughput,
but does not rank performance, as the lead in BogoMips of the "Raspberry Pi 4" over the "M2" illustrates.

Last but not least, any other symbol is tested if it is a name of a matrix function of `matrico`.
This means, true is returned, if the symbol is the name of a matrix function in `matrico`;
at the very least, the symbol needs to start with "`mx`".
However, the purpose of this function is given by its side-effect,
which is printing the docstring of the "`mx`"-function to the terminal,
if the symbol is the name of such function, and thus supply an online help for the REPL:

* `(matrico 'sym)` - returns boolean, answering if argument symbol is a name of an "`mx`"-function in the `matrico` module and if, print its docstring.

Together with this functionality, a user just needs to know the `(matrico)` thunk to explore all of `matrico`:

1. Help about the single argument `matrico` function via `(matrico)`.
2. List all of `matrico`'s matrix functions via `(matrico 'list)`.
3. Print help on a matrix function via `(matrico 'mx-...)`.

The associated source file `matrico.scm` is [here](https://github.com/gramian/matrico/blob/main/matrico.scm), which defines the module.
The remaining user-facing "`mx`"-functions are contained in the source file `src/mx.scm`,
which is included into the `matrico.scm` source file and will be detailed in the subsequent posts.

