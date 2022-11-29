---
title: "The matrico blueprint"
date: 2022-05-23
tags: Lisp Scheme SchemeLang matrico
---

## How is the `matrico` source code organized?

Overall, the `matrico` project is supposed to be a module wrapped in a CHICKEN Scheme egg.
Hence, a module named "matrico" contains all user-facing functions.
There are two types of function in the matrico module:
first, functions beginning with "matrico", which provide `matrico` specific meta-data functionality,
second, functions beginning with "mx", which provide the core **m**atri**x** functionality.
While the former ("matrico" functions) are defined in the `matrico.scm` source file,
the latter ("mx" functions) are defined in the `src/mx.scm` source file, that is included (CHICKEN Scheme provides `include-relative`) into `matrico.scm`.

All but one module depend on the `utils` module, defined in `src/utils.scm`.
This module provides extra generic or universal functionality.
The `fpmath` module, defined in `src/fpmath.scm` depends on `utils`,
and provides additional flonum (floating-point number) functions.
The `f64vector` module, defined in `src/f64vector.scm`, depends on `utils` and
`fpmath` (the latter may become redundant with newer versions of CHICKEN Scheme),
and provides map-reduce functions for homogeneous f64 (double-precision flonum) vectors.
Next, the `matrix` functor (CHICKEN Scheme has functors which are modules parametrizable by other modules)
depends only on `utils` and provides map-reduce functionality for a generic matrix type.
The `dense` module then specializes `matrix` functor via its `f64vector` dependency.
Finally, the main `matrico` module depends on the `utils`, `fpmath`, and `dense` modules.  

In summary, following is the module hierarchy:
* `matrico` module (user-facing; depends `utils`, `fpmath`, `dense`)
* `mx` library (included by `matrico`)
* `dense` module (specializes `matrix`; depends `f64vector`)
* `matrix` functor (depends `utils`)
* `f64vector` module (depends `utils`, `fpmath`)
* `fpmath` module (depends `utils`)
* `utils` module

This means a user will only see functions prefixed with `matrico` or `mx`, and
hence, only these functions will do argument checks. All lower level functions
will assume admissible arguments.

Furthermore, a basic test library is defined in `tests/check.scm`,
while the per module tests are set up in `tests/test-MODULENAME.scm`,
with `MODULENAME` either `utils`, `fpmath`, `f64vector`, or `matrico`,
and called from `tests/run.scm`. 

