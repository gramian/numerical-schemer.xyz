---
title: "Matrico Module I"
date: 2022-09-09
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `matrico` Module (Part I)

The `matrico` module is the user-facing part of the `matrico` project.
Over the next few posts, I will describe the available functions therein.
There are two sets of functions implemented in this module,
the ones prefixed with "`matrico-`" are meta-data functions as they provide information about `matrico` (or the current machine),
while the ones prefixed with "`mx-`" are functions operating on or returning matrices.
I start here by listing the "`matrico-`"-functions.

First, a [thunk](https://en.wikipedia.org/wiki/Thunk#Functional_programming) (zero argument function) that returns the
version of the `matrico`-module in use:

* `(matrico-ver)` - returns pair of integers representing major and minor version semantic number in the head (`car`) and tail (`cdr`) respectively.

A pair of integer numbers is returned instead of a string, typically used in CHICKEN Scheme for versions,
so they can directly be compared.

Second, is a thunk that returns information on how to cite this module.
I included this due to my experiences in research software development,
so in the (unlikely) case somebody uses `matrico` for science, it is clear how to cite it:

* `(matrico-cite)` - returns string holding preferred citation for `matrico`.

Also, a thunk with a minimal description of `matrico` and a link to the user-facing documentation is provided:

* `(matrico-about)` - returns string holding summary of `matrico`.

Next, I was contemplating how to classify the hardware `matrico` is running on.
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

* `(matrico-mips)` - returns number of busy-wait million-instructions-per-second.

I note, as have other commentators, that this "benchmark" is rather crude, nonetheless,
and particularly since CHICKEN Scheme is single-threaded, this function gives a hint on the processor's throughput.

Last but not least, a predicate is included that ostensibly tests if the argument, of type symbol,
is a name of a matrix function of `matrico`.
This means, to return true, at the very least, the symbol needs to start with "`mx-`".
However, the purpose of this function is given by its side-effect,
which is printing the docstring of the "`mx-`"-function to the terminal,
if the symbol is the name of such function, and thus supply an online help for the REPL:

* `(matrico? sym)` - returns boolean answering if argument symbol is a name of an "`mx-`"-function in the `matrico` module and if, print its docstring.

The associated source file `matrico.scm` is [here](https://github.com/gramian/matrico/blob/main/matrico.scm), which defines the module.
The remaining user-facing "`mx-`"-functions are contained in the source file `src/mx.scm`,
which is included into the `matrico.scm` source file and will be detailed in the subsequent posts.

