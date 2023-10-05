---
title: "Matrico Algorithms I"
date: 2023-07-10
tags: Lisp Scheme SchemeLang Chicken-Scheme matrico
---

## `matrico`'s Matrix Multiplication

* Part I
* [Part II](http://numerical-schemer.xyz/2023/08/21/matrico-algorithms-ii.html)
* [Part III](http://numerical-schemer.xyz/2023/10/05/matrico-algorithms-iii.html)

### Significance of Matrix Multiplication

[Matrix multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication)
is not only the abstract mathematical operation producing the matrix product,
it has application in many sciences.
Also, it can be interpreted as system of linear equations,
[composition](https://en.wikipedia.org/wiki/Function_composition) of linear maps,
as well as all possible [inner products](https://en.wikipedia.org/wiki/Dot_product) of sets of column vectors,
or (since a vector can be interpreted as a matrix) as plain matrix-vector multiplication -
the application of a linear map to a vector.

The classical algorithm has a cubic compute complexity for two square matrices of the same dimensions.
However, two notes need to be made:
First, as data can be reused, the memory complexity is only quadratic.
Second, the practical efficency highly depends on cache awareness of the nested iteration (recursion);
see "[Scientific Programming and Computer Architecture](https://github.com/divakarvi/bk-spca)" for details.
Lastly, I note there are compute-wise more efficient algorithms with complexities below cubic, such as Strassen's,
as well as memory-wise more efficient algorithms such blocked (tiled) algorithms,
but either are outside the scope of **matrico** and this post.

### The Mathematical Algorithm

Given two matrices `X` and `Y`, an element-wise computation of the matrix product `Z` can be described by:

```
X ∈ ℝᴹ×ᴷ, Y ∈ ℝᴷ×ᴺ
X ⋅ Y := Z ∈ ℝᴹ×ᴺ : Zₘₙ = ∑ₖ₌₁ᴷ Xₘₖ Yₖₙ
```

For the two matrices to be compatible for matrix multiplication,
the first operand's (`X`) column dimension `K` must match the second operand's (`Y`) row dimension `K`.
The resulting matrix `Z` will have the row dimension of the first operand `M`,
and the column dimension of the second operand `N`.
Practically, the inner product of the `m`-th row of `X` with the `n`-th column of `Y`
yields the `(m,n)` element of `Z`.

### `matrico`'s Implementation

To maximize the use of **matrico**'s matrix data structure,
the base matrix multiplication algorithm is designed for the product of two matrices,
where the first argument is passed as transposition of the actual matrix,
which means the sequential memory columns can exploited for both operands.
In other words: The matrix product of matrices `X` and `Y` is computed, given `X`-transposed and `Y`.
And, the arguments are thus compatible for matrix multiplication if their numbers of rows agree.
In mathematical terms we slightly reformulate the above formula to:

```
X ⋅ Y := Z ∈ ℝᴹ×ᴺ : Zₘₙ = ∑ₖ₌₁ᴷ Xᵀₖₘ Yₖₙ
```

In (CHICKEN) Scheme code this is implemented by the internal function [`matrix-dot*`](https://github.com/gramian/matrico/blob/main/src/matrix.scm#L332): 

```scheme
(define (matrix-dot* xt y)
  (define xt-data (matrix-data xt))
  (define xt-cols (matrix-cols xt))
  (matrix-map* (lambda (y-col)
                 (let [(ret (make-column xt-cols))]
                   (let rho [(idx 0)
                             (lst xt-data)]
                     (if (fx= idx xt-cols) ret
                                           (begin
                                             (column-set! ret idx (column-dot (head lst) y-col))
                                             (rho (fx+1 idx) (tail lst)))))))
               y))
```

where the star `*` signifies that this is not the standard matrix multiplication,
but with the first argument transposed.
First, local bindings are set up for the data element (the list of vectors) of the first (transposed) argument,
and its number of columns.
Then, making up the cubic complexity, follow the three nested loops:

1. A `matrix-map*` (map over all columns) of the second argument matrix.
2. A "manual" `for-each` column of the first argument matrix (manual recursion is used to track the index).
3. The inner product of the current column of the first argument with current column of the second argument,

A more functional way would be to assemble a list of resulting inner product results in the middle loop,
and then convert the list via `list->column` (encapsuling `list->f64vector`) to the vector type on return,
but this would have a significant performance impact compared to the mutating the current result column elements.

All in all, for the **matrico** user this means when using matrix multiplication,
prefer the use of [`mx-dot*`](http://wiki.call-cc.org/eggref/5/matrico#matrix-multiplication) as it is the fastest, directly using `matrix-dot*`;
`mx-dot` is slower as it needs to transpose the first argument before applying `mx-dot*`.

Next, I will dissect the QR algorithm and **matrico**'s implementation of it.
