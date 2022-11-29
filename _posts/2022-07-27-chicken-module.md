---
title: "CHICKEN module Module"
date: 2022-07-27
tags: Lisp Scheme SchemeLang ChickenScheme Chicken-Module
---

## CHICKEN Scheme's module system and included `module` module

A module or library bundles bindings, functions and macros.
While the Scheme standards [R6RS](https://www.r6rs.org) and [R7RS](https://small.r7rs.org) define module systems, R5RS, which CHICKEN Scheme is implementing, does not.
Nonetheless, CHICKEN defines its own module system which bears some resemblence
to the library forms of [R6RS](http://www.r6rs.org/final/html/r6rs/r6rs-Z-H-10.html) (see also [SRFI-83](https://srfi.schemers.org/srfi-83/srfi-83.html))
and R7RS, while integrating nicely with CHICKEN's [egg extensions](http://wiki.call-cc.org/man/5/Extensions#a-module-exporting-syntax).
Because it's Scheme, the module system is implemented in a module itself, called [`(chicken module)`](http://wiki.call-cc.org/man/5/Modules).

### Syntax

A module is defined by a list starting with the keyword `module`, followed by
the module's name. The next element is a list of the exported identifiers. An
import form, a list of imported modules, lead by the `import` keyword, follows.
Then, the body of the module, comprised of module-local and exported bindings,
is implemented.

```
(module name
  (bind1
   bind2
   ...)

  (import mod1 mod2 ...)

  (define bind0 ...)

  (define bind1 ...)

  (define (bind2 ...) ...)

)
```

Compared to the R6RS library definition, I would have preferred that the list of exports
is also lead by an `export` keyword in CHICKEN.

### Re-Export

In addition to imports and exports, the body may contain a [re-export](http://wiki.call-cc.org/man/5/Modules#reexport) form.
This is a list, lead by the `reexport` keyword, listing imported modules to be
also exported, and hence are re-exported.

* `(reexport mod1 ...)` - Re-exports an imported module from the importing module.

### Import (and Re-Export) Specifiers

The entries in the import and re-export forms can be fine tuned via [specifiers](http://wiki.call-cc.org/man/5/Modules#import),
similar to R6RS. Each of those is used in place of an entry in the import or
re-export lists:

* `(only mod id1 ...)` - Import (re-export) only bindings with listed identifiers from given module.
* `(except mod id1 ...)` - Import (re-export) all except bindings with listed identifiers from given module.
* `(rename mod (old1 new1) ...)` - Import (re-export) bindings with renamed identifiers given by two element list from given module.
* `(prefix mod sym)` - Import (re-export) bindings with prefix given by symbol from given module.

### Functors
A great feature of CHICKEN Scheme's module system is "[functors](http://wiki.call-cc.org/man/5/Modules#functors)", which are modules parametrized by other modules,
not unlike template classes in [template metaprogramming](https://en.wikipedia.org/wiki/Template_metaprogramming).
This means a generic module can be defined that is then specialized by the (template) argument module.
```
(functor (name
            (argmod
               argbind1
               argbind2
               ...))
  (bind1
   bind2
   ...)

  (import mod1 mod2 ...)

  (define bind1 ...)

  (define (bind2 ...) ...)
)
```
A specialization of a functor is then given by:
```
(module name1 = name)
```
Beyond specializing the functor, the specialization can also import or re-export additional modules,
and export further bindings, making functors also base classes from which the specialization [inherits](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)).

A functor is used in `matrico` to define the abstract (back-end) matrix module,
which is explored in the next post.

