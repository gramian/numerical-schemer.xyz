---
title: "Matrico matrix Functor"
date: 2022-08-04
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s Matrix Functor Module `matrix`

The matrix [functor](http://numerical-schemer.xyz/2022/07/27/chicken-module.html#functors)
is the abstract definition of `matrico`'s core matrix type,
which is a back-end module and particularly not user-facing,
even though a subset of functions will be passed through to the user interface.
This setup has two reasons:

1. Generic functional procedures on the matrix type simplify higher-level function definition,
2. and this allows to hide the low-level matrix type definition from the user and thus ensure consistency of the matrix data.

Furthermore, the matrix type is encapsuled in a functor, instead of a standard module,
to keep the underlying entry-type of the matrix flexible. So while, `matrico` will by default use double-precision flonums,
using single-precision flonums, rationals or integers instead remains possible.
Beyond entry type, also the column type is kept exchangeable this way.
This means instead of the standard dense matrix format, also a [sparse matrix](https://en.wikipedia.org/wiki/Sparse_matrix) format could be implemented.
The source file `src/matrix.scm` is [here](https://github.com/gramian/matrico/blob/main/src/matrix.scm).

### Matrix Type

Internally a matrix in `matrico` is designed as a [`record`](http://wiki.call-cc.org/man/5/Module%20(chicken%20base)#define-record) or structure via the `(chicken base)` module.
The matrix itself is encoded as list of vectors, specifically a list of columns - a [column-major](https://en.wikipedia.org/wiki/Row-_and_column-major_order) matrix data model.
This is in opposition to the typically chosen plain vector with strided rows or columns, or vector of vectors.
The reasons for this uncommon choice are again twofold:

1. Using a list plays on the strength of Scheme while keeping each column (a vector) accessible in constant time.
2. For larger problems it is not necessarily preferable to use one large memory block as matrix.
   The list-of-columns approach allows spreading the columns throughout available memory and thus making allocation easier and improve balancing memory use.

It will be important for higher level functions using this matrix type to keep this underlying list-of-columns layout in mind.
This means minimizing list traversal and maximizing per-column work.

As the matrix dimensions are constant after binding, the number of columns remains constant,
hence it can be stored and no list traversal is needed to obtain this number.
Thus, the matrix record is defined with two slots,

1. the list-of-columns storing the matrix entries,
2. and the number of columns,

yielding the implicitly defined functions:

* `(make-matrix lst cols)` - Returns a `matrix` with `list`-of-`column`s argument and `number` of columns argument.
* `(matrix-data mat)` - Returns the `list`-of-`column`s payload of the `matrix` argument.
* `(matrix-cols mat)` - Returns the `number` columns of the `matrix` argument.
* `(matrix? any)` - Returns a `boolean` answering if argument is a `matrix`.

The `make-matrix` function illustrates why this function cannot be user facing.
Would it be, it is not guaranteed that a list of columns of equal length or even a list at all is passed as first slot,
nor that the correct number of columns or a number at all is passed as second slot.
The matrix functor takes care of assigning these correctly.

### Local Functions

A certain composition of procedures appears in quite a few matrix functions, hence this was extracted and made into two local functions.
These functions are incomplete map operations of matrix arguments.

* `(matrix-map* fun x)` - Returns `matrix` resulting from applying function argument to each column of the `matrix` argument.
* `(matrix-map** fun x y)` - Returns `matrix` resulting from applying function argument to each column of the two `matrix` arguments.

Note that these are not complete map operations on matrices, as only the matrix functor knows about the map operation on a column.
Also, for performance reasons, two separate functions were made instead of one with variadic arguments.

### Generators

To create matrices, generators or constructors are used.
First, a basic function to specify dimensions in terms of rows and columns and the value of all entries, is available.
Sometimes one wants to prescribe a complete matrix manually entry-by-entry, for this purpose a generator from a list-of-list,
in [row-major](https://en.wikipedia.org/wiki/Row-_and_column-major_order) format,
which is more natural for terminal input or reading from text files, is provided.
To procedurally assemble a matrix, a function is included which calculates an entry based on its row and column index.

* `(make-matrix* rows cols val)` - Returns `matrix` of row and column argument dimension with all entries set to value argument.
* `(make-matrix** lst)` - Returns `matrix` from `list`-of-`lists` argument.
* `(matrix-generate fun rows cols)` - Returns `matrix` resulting from computing entries by applying function argument to entries' row and column indices.

I use the stars to signify variants of the standard generator `make-matrix` which is automatically provided through creating the matrix record.

### Combiners

While the generator functions create matrices from non-matrix arguments,
combiners create matrices from matrices, particularly by concatenation.
This is implemented by horizontal concatenation, meaning joining lists of all matrix arguments (having the same number of rows),
and vertical concatenation, meaning joining all respective columns of all matrix arguments (having the same number of columns).

* `(matrix-horcat . mat)` - Returns `matrix` resulting from concatenating `list`-of-`matrix`es argument horizontally.
* `(matrix-vercat . mat)` - Returns `matrix` resulting from concatenating `list`-of-`matrix`es argument vertically.

### Dimensions

To determine the dimensions of a matrix, a set of functions is provided.
These include the number of rows, the number of entries, and if a matrix has the shape of a mathematical scalar, vector or matrix;
the latter is sometimes known as rank of a multi-dimensional array, but this is only correct in the sense of the [tensor rank](https://mathworld.wolfram.com/TensorRank.html),
and not as "rank" typically used in [linear algebra](https://en.wikipedia.org/wiki/Rank_(linear_algebra)).

* `(matrix-rows mat)` - Returns the `number` of rows of the `matrix` argument.
* `(matrix-numel mat)` - Returns the total `number` of entries of the `matrix` argument.
* `(matrix-dims mat)` - Returns `0` for 1x1, `1` for Nx1 or 1xN, or `2` for NxM dimensions of `matrix` argument. 

I note again that the `matrix-cols` function is automatically provided through creating the matrix record.

### Accessors

There are various ways matrices can be accessed. Most basic is access to a single element;
I also provide a specific accessor to the entry at the first row and first column, as this 
entry is always available and used for converting between a scalar matrix and its entry type.
Also a function to return an element form the first column is included, not for performance as one may suspect,
but readability, as it highlights that using this call no list traversal is needed.
To manipulate single entries there is a function which returns a copy of the argument matrix, yet with a changed entry,
and a function mutating the entry of the argument matrix.
Beyond single entries, a single column, a single row or a sub-matrix may need to be extracted,
for which functionality is included, the latter needs the `sublist` function form the [`utils` module](http://numerical-schemer.xyz/2022/06/12/matrico-utils.html).
Another important accessor for matrices returns the matrix's diagonal - all entries with the same row and column index of a square matrix.

* `(matrix-ref00 mat)` - Returns the entry of the first column and the first row of the `matrix` argument.
* `(matrix-ref*0 mat row)` - Returns the entry of the first column at the row argument of the `matrix` argument.
* `(matrix-ref mat row col)` - Returns the entry at the row argument and column argument of the `matrix` argument.
* `(matrix-set mat row col val)` - Returns matrix with entry at the row argument and column argument of the `matrix` argument to value argument.
* `(matrix-set! mat row col val)` - Returns `void`, mutate entry at the row argument and column argument of the `matrix` argument to value argument.
* `(matrix-col mat col)` - Returns (column) `matrix` being at the column index argument of the `matrix` argument.
* `(matrix-row mat row)` - Returns (row) `matrix` being at the row index argument of the matrix argument.
* `(matrix-diag mat)` - Returns (column) `matrix` being the diagonal of the square `matrix` argument.
* `(matrix-submatrix mat row1 row2 col1 col2)` - Returns `matrix` being the entries between row index arguments and column index arguments of `matrix` argument.

### Predicates

Given a matrix, one may want to test its shape: Is it a one-column matrix or one-row matrix,
is it a vector so either a column matrix or row matrix, or a scalar, meaning a column matrix and row matrix.
Furthermore, I may want to know if a matrix has the same number of rows and columns.
Given two matrices, I may want to know if those have they same number columns, rows or both.
Lastly, it may be interesting to test for a provided predicate if any or all entries of a matrix fulfill it.

* `(matrix-col? mat)` - Returns `boolean` answering if `matrix` argument has only one column.
* `(matrix-row? mat)` - Returns `boolean` answering if `matrix` argument has only one row.
* `(matrix-scalar? mat)` - Returns `boolean` answering if `matrix` argument has only one column and one row.
* `(matrix-vector? mat)` - Returns `boolean` answering if `matrix` argument has only either one column or one row.
* `(matrix-square? mat)` - Returns `boolean` answering if `matrix` argument has the same number of rows and columns.
* `(matrix-samecols? x y)` - Returns `boolean` answering if `matrix` arguments have same number of columns.
* `(matrix-samerows? x y)` - Returns `boolean` answering if `matrix` arguments have same number of rows.
* `(matrix-samedims? x y)` - Returns `boolean` answering if `matrix` arguments have same number of columns and same number of rows.
* `(matrix-any? pred mat)` - Returns `boolean` answering if any entry of `matrix` argument fulfills predicate argument.
* `(matrix-all? pred mat)` - Returns `boolean` answering if all entries of `matrix` argument fulfill predicate argument.

I note that the `matrix?` predicate is automatically provided through creating the matrix record.

### Reducers

There are three reducers or folders required for a two-dimensional array, such as a matrix:

1. folding all columns to a row matrix,
2. folding all rows to a column matrix,
3. and folding all columns and rows to a scalar,

which are implemented as three functions, as opposed to a single function with an extra argument,
due to the very different internals of these functions and their non-uniform return type.

* `(matrix-colfold fun ini mat)` - Returns (row) `matrix` resulting from folding all columns of `matrix` argument pairwise by function argument starting with initial argument.
* `(matrix-rowfold fun ini mat)` - Returns (column) `matrix` resulting from folding all rows of `matrix` argument pairwise by function argument starting with initial argument.
* `(matrix-allfold fun ini mat)` - Returns entry-type resulting from folding row of all folded column of `matrix` argument by function argument starting with initial argument.

### Mappers

The unary map of a matrix is just a composition of the outer list map and the inner column map, applying the argument function to each entry.
The binary map is more involved, as not only matrices of same dimension should be mappable, but also two matrices that fall in one of the cases:

1. one argument is scalar, the other has any dimension,
2. both arguments share either column or row dimension,
3. or both arguments are vectors.

This functionality is called "automatic brodcasting" (in Octave, NumPy, Julia) or "implicit expansion" (in MATLAB).

* `(matrix-map fun mat)` - Returns `matrix` resulting from applying function argument to all `matrix` argument's entries.
* `(matrix-broadcast fun x y)` - Returns `matrix` resulting from applying function to `matrix` arguments under broadcasting rules.

### Specific

There a some operations specific to matrices, or rather two-dimensional arrays,
and independent of the type of the entries that are generically provided in this functor module.
There is (mathematical) [vectorization](https://en.wikipedia.org/wiki/Vectorization_(mathematics)), meaning stacking all columns of a matrix,
and [transposition](https://en.wikipedia.org/wiki/Transpose) of a matrix, which swaps row and column indices of each entry.
Whereas the transposition is implemented elegantly using a similar approach to [list-of-list matrix tranposition](https://rosettacode.org/wiki/Matrix_transposition#Scheme).

Given two column matrices with the same number of rows, the [scalar product](https://en.wikipedia.org/w/index.php?title=Scalar_product) is an essential operation in (vector) linear algebra,
also known as inner product.
The outer product counter part is not defined here, as it can be more easily achieved by a broadcasting / expansion operation.
In a sense, [matrix multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication)
is then a generalization if the scalar product for two matrices (where the number or columns of the first operand is equal to the number of rows of the second operand),
as it computes all possible scalar products of all rows of the first operand and all columns of the second operand.
I call the matrix multiplication `dot`-product as [NumPy](https://numpy.org/doc/stable/reference/generated/numpy.dot.html) does, since it is mathematically more expressive.
The implemented function is called `dot*` (dot-star, a special variant of `dot`) though,
because the first argument is assumed to be transposed: This allows an efficient implementation for the list-of-column matrix data model.

Lastly, for performance and convenience, twin functions to "explode" a matrix into a list of column matrices,
and "implode" a list of column matrices (with the same number of rows) into one matrix, where the latter wraps the `matrix-horcat` function are included.

* `(matrix-vec mat)` - Returns (column) `matrix` resulting form concatenating columns of `matrix` argument vertically.
* `(matrix-transpose mat)` - Returns `matrix` with entries of argument `matrix`, where the entries' row and column indices are exchanged.
* `(matrix-scalar xt y)` - Returns entry-type resulting from computing scalar product of column `matrix` arguments with same number of rows.
* `(matrix-dot* xt y)` - Returns `matrix` resulting from matrix multiplication of the `matrix` arguments, where the first argument is assumed transposed.
* `(matrix-explode mat)` - Returns `list`-of-column-`matrix`es from `matrix` argument.
* `(matrix-implode lst)` - Returns `matrix` from `list`-of-column-`matrix`es argument.

### Utilities

Lastly, some matrix utility functions are needed, these have, by nature, side-effects.
Most importantly is printing a matrix to the standard output, so typically the terminal.
This functions needs a function argument that processes the internal entry-to-string conversion,
so that each printed entry has the same character width, which is especially necessary for flonums, see `fptaper` from the [`fpmath` module](http://numerical-schemer.xyz/2022/07/08/matrico-fpmath.html).
This print function utilizes the "square bracket" symbols from the [miscellaneous technical](https://en.wikipedia.org/wiki/Miscellaneous_Technical) unicode block.

Also, matrices need to be stored and restored from disk; this can be done in a native format using S-expressions.
Additionally, matrices need to be exported, so other numerical packages,
such as MATLAB, Octave, NumPy or Julia can load them. The common format for all of these is 
the [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) (comma-separated-value) format.

* `(matrix-print fun mat)` - Returns `void`, prints argument `matrix` on the standard output, preprocessing matrix entries by function argument.
* `(matrix-export str mat)` - Returns `void`, output argument `matrix` to CSV file at `string` argument path.
* `(matrix-save str mat)` - Returns `void`, output argument `matrix` to S-expression formatted file at `string` argument path.
* `(matrix-load str)` - Returns matrix loaded from string argument path.

In the next post, I will focus on the `(chicken memory representation)` which provides an essential function
for the matrix functions `matrix-horcat`, `matrix-vercat`, `matrix-col`, `matrix-submatrix`, `matrix-set`, `matrix-vec`,
and `matrix-explode`, not discussed here.
