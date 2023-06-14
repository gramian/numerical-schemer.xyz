---
title: "CHICKEN Scheme Metadata"
date: 2023-06-14
tags: Lisp Scheme SchemeLang Chicken-Scheme matrico
---

## CHICKEN Scheme Metadata

One of the highlights of CHICKEN Scheme is it's "egg" (community) [extension repository](http://eggs.call-cc.org/5/).
To describe such an extension to the repository and hence to a potential user,
information about its contents need to be assigned (or deducted).
This information about code (which is data after all) is metadata.
The selection and detail of metadata assigned to a software package in a repository is a task not unique to CHICKEN though.
In this post, I look at the [CHICKEN Scheme egg vocabulary](http://wiki.call-cc.org/man/5/Egg%20specification%20format) and its closest relatives - Scheme(-based) package manifest formats:

* [Snow](https://snow-fort.org/doc/spec/)
* [Akku](https://gitlab.com/akkuscm/akku/-/blob/master/Akku.manifest)
* [Common Scheme](http://synthcode.com/scheme/common-scheme/doc/common-scheme-Z-H-6.html#node_sec_6.3.10)
* [Guix](https://guix.gnu.org/en/cookbook/en/html_node/A-_0060_0060Hello-World_0027_0027-package.html)

As a disclaimer, I will note that only a subset of the respective vocabularies is discussed.

### CHICKEN egg

The egg specification format is an association list where the keys (symbols) are defined values.
Interestingly, an egg can contain multiple modules, libraries, programs, scripts or data files.
This is not obvious as one could assume only a single module is encapsuled by an egg.
The name of the egg is determined by the [filename](http://wiki.call-cc.org/man/5/Extensions#creating-eggs),
thus there is no `name` field needed inside the egg.

Below is a variant of the actual `matrico` egg file (named `matrico.egg`):

```scheme
((version "0.0")
 (author "Christian Himpe")
 (synopsis "A flonum matrix module for CHICKEN Scheme.")
 (license "ZLIB-Acknowledgement")
 (category math)
 (components (extension matrico (csc-options "-O3" "-d0" "-C" "-O2"))))
```

The `version`-value in CHICKEN eggs is of type string, which is suggested to be of the form `"major.minor.patch"`,
which allows it to be compared based on split [string comparisons](http://api.call-cc.org/5/doc/semantic-version/version-compare).
Yet, the egg's defined version matters only for [installing eggs locally](http://wiki.call-cc.org/man/5/Egg%20specification%20format#version),
as the actual version is set by the [`release-info` file](http://wiki.call-cc.org/releasing-your-egg#creating-a-release-info-file).

An interesting aspect of the CHICKEN egg metadata-set is the `category`-key,
which allows a topical grouping of extensions.
The associated `category`-values have to be from a set of [internally defined values](http://wiki.call-cc.org/eggs%20tutorial#egg-categories),
to ensure its validity.

As a sidenote, the `components` value describes how a specific part of the egg is build, here the `matrico` **module**.
The remaining keys are discussed in relation to contenders summarized in the following.

### Snow Package

`Snow` is an implementation independent package repository for R7RS codes.
Here is how a manifest for a `matrico` `snow` package could look:

```scheme
(package
  (name (matrico))
  (version "0.0")
  (authors "Christian Himpe")
  (description "A flonum matrix module for CHICKEN Scheme.")
  (license 'bsd)
  (manual "README.md")
  (test "tests/run.scm"))
```

### Akku Package

`Akku.scm` is a package manager for R6RS codes and a mirror of the `Snow` repository providing R7RS packages.
Following is a prototypical `Akku` manifest for `matrico`:

```scheme
(akku-package ("matrico" "0.0")
  (authors "Christian Himpe")
  (synopsis "A flonum matrix module for CHICKEN Scheme.")
  (description "A flonum matrix module for CHICKEN Scheme.")
  (license "ZLIB-Acknowledgement")
  (homepage "http://numerical-schemer.xyz"))
```

### Common Scheme

`Common Scheme` is a portable module system for various Schemes.
In contrast to the other analyzed metadata formats,
here the metadata is embedded in the module definition.
Notably, `Common Scheme`'s `common-module` do not define a set of valid keys except `summary` and `description`.
Below is a dummy `matrico` `common-module` definition using the suggested declarations:

```scheme
(common-module (matrico)
  ((export ...)
   (version "0.0")
   (author "Christian Himpe")
   (summary "A flonum matrix module for CHICKEN Scheme.")
   (description "...")
   (license "ZLIB-Acknowledgement")
   (homepage "http://numerical-schemer.xyz")))
```

### Guix Package

`Guix` is a [functional package manager](https://en.wikipedia.org/wiki/GNU_Guix) with a Scheme-based package definition.
An incomplete definition for a `matrico` `Guix` package follows:

```scheme
(define-public matrico
  (package
    (name "matrico")
    (version "0.0")
    (synopsis "A flonum matrix module for CHICKEN Scheme.")
    (description "")
    (license zlib-acknowledgement)
    (home-page "http://numerical-schemer.xyz")
    (source ...)))
```

### Compared to CHICKEN egg

First, common among all compared vocabularies are fields for "version", "author(s)", "synopsis/summary/description", and "license".

The `author`-value in CHICKEN is a string.
For multiple authors, I think a list of strings would be preferrable as in the `Snow` manifest.
This will require more logic analyzing the `author`-value, as it can now be of type string for a single author, or a list of strings for multiple authors.

For CHICKEN eggs the `license`-value is a string, which gives maximal flexibility to specify a license,
for example as an SPDX identifier, a URL to the license, or a free form text.
I also assume a string is used, so [multiple licenses may be named](http://wiki.call-cc.org/eggs-licensing#egg-metafile-tagging).
Using a defined value like `Guix` means maximal restriction, as only the exact predefined names may be used.
`Snow`'s symbol-type values for licenses seems to be a good compromise.
For multiple licenses a list of symbols, instead of a single symbol, could then be used, as for the author field.

Particularly interesting in the Common Scheme metadata is the use of the `summary`-value (and if not available the `description`-value).
This value is used to populate an index file with the module descriptions to improve searchability.
A related idea is used in [MATLAB for providing help](https://www.mathworks.com/help/matlab/matlab_prog/add-help-for-your-program.html) in the terminal.
The use of the `synopsis`-value for an interpreter help would be a nice addition to the interpreter.
For example one could add a toplevel special command like `,y EGGNAME` which prints the egg's synopsis.

The `Akku`, `Common Scheme`, and `Guix` vocabularies provide a `homepage` (or `home-page`) key-value pair.
At first-glance, this seems to be missing from the CHICKEN vocabulary.
However, there are two reasons making this information not essential for CHICKEN:
First, each egg has its own wiki page, so a potential homepage (or rather website) can be noted there,
and second, such a key would introduce ambivalence between the homepage and the egg wiki page.

The `Snow` vocabulary has a `manual` key-value pair.
Similar to `homepage`, this is of lesser use for CHICKEN due to the online egg documentation.
However, pointing to a text file that could be installed with the egg files as offline documentation could be useful.

Overall, CHICKEN eggs provide all typical metadata fields,
but in my humble opinion `author` and `license` fields could be improved.
The documentation of the CHICKEN egg metadata is definitely the best among the compared formats,
with the minor remark that it does not state minimally required keys (as none of the format docs do).

I may update this post with further insights,
but next, in a trilogy of posts, I will explain high complexity algorithms in `matrico`.
