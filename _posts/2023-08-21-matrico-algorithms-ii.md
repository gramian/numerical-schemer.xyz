---
title: "Matrico Algorithms II"
date: 2023-08-21
tags: Lisp Scheme SchemeLang Chicken-Scheme matrico
---

## `matrico`'s QR Decomposition

* [Part I](http://numerical-schemer.xyz/2023/07/10/matrico-algorithms-i.html)
* Part II
* [Part III](http://numerical-schemer.xyz/2023/10/05/matrico-algorithms-iii.html)

### The Significance of the QR Decomposition

In essence, the [QR decomposition](https://en.wikipedia.org/wiki/QR_decomposition)
aka QR factorization is a (multiplicative) matrix decomposition,
meaning, it is an algorithm that, given a matrix `A`, yields matrices `Q` and `R`
such that `A = Q R`. The factors `Q` and `R` feature properties that are useful
for higher level algorithms, described next. The defining quality of this decomposition is,
that the `Q` factor is [orthogonal](https://en.wikipedia.org/wiki/Orthogonal_matrix),
and the `R` factor is [upper right triangular](https://en.wikipedia.org/wiki/Triangular_matrix).

A central application of the QR decomposition is solving linear systems:
```
A x = b
```
Here the orthogonality of `Q` is exploited by using that the inverse of it is
given by its transpose (for a square `Q`, a rectangular `Q` has a related feature),
which hence can be applied easily to the right-hand side vector `b`. Now, `R` can be
inversely applied to `b` by [forward substitution](https://en.wikipedia.org/wiki/Triangular_matrix#Forward_and_back_substitution):
```
A x = Q R x = b => R x = Q^-1 b = Q^T b => x = R^-1 (Q^T b)
```

The QR decomposition can also be used to compute a matrix's [eigenvalue decomposition](https://en.wikipedia.org/wiki/Eigendecomposition_of_a_matrix):
```
A = S^-1 D S
```
via the [Francis QR algorithm](https://en.wikipedia.org/wiki/QR_algorithm), one of the [Top Ten Algorithms](https://web.archive.org/web/20210507011904/https://pi.math.cornell.edu/~web6140/TopTenAlgorithms/QRalgorithm.html),
which is a variant of the [Power Iteration](https://en.wikipedia.org/wiki/Power_iteration):
```
A_0 = A
A_k = Q_k R_k
A_k+1 = R_k Q_k = Q_k^-1 Q_k R_k Q_k = Q_k^T A_k Q_k 
```

A [singular value decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition):
```
A = U D V^T
```
in turn can be computed [via the QR algorithm](https://math.stackexchange.com/a/4309570/29967),
and thus by extension via the QR decomposition by computing the eigenvalue
decompositions of the matrix products `A A^T` and `A^T A`.

The determinant of a matrix can be (approximately, up to sign) computed by the QR decomposition, too:
```
A = Q R => det(A) = det(Q) det(R) = ±1 det(R) = ±Π_i R_ii => |det(A)| = |Π_i R_ii|
```

### Computing the QR Decomposition

It is critical to understand that there is no one algorithm called **the** QR decomposition,
but a family of algorithms which in exact arithmetic are equivalent, but differ
in accuracy, efficiency and overall complexity.
Now, at the core lies the choice of the orthogonalization algorithm, of which the
top three are:

* [Householder Reflections](https://en.wikipedia.org/wiki/Householder_transformation)
* [Givens Rotations](https://en.wikipedia.org/wiki/Givens_rotation)
* [Gram-Schmidt](https://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process)

`matrico` is using the Gram-Schmidt orthogonalization,
because it works vector-wise and not mainly element-wise as the other two methods.
Now, there are two major variants of Gram Schmidt:

* (Classic) Gram-Schmidt
* Modified Gram-Schmidt

`matrico` uses the modified variant as it is [numerically stabilized](https://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process#Numerical_stability) and thus more accurate in finite precision arithmetic.
Lastly, the orthogonalization can be performed:

* row-wise
* column-wise

Since, `matrico`'s matrix layout is as a list of columns, the column-wise approach is chosen.

Altogether, computing the QR via column-wise modified Gram-Schmidt is called "Schwarz-Rutishauser",
of which I learned about from in [this](https://towardsdatascience.com/can-qr-decomposition-be-actually-faster-schwarz-rutishauser-algorithm-a32c0cde8b9b) post.

### `matrico`'s QR Decomposition

`matrico` implements a hierarchy of six functions related to the QR decomposition,
each for a particular use-case.
Underlying all is the `mx-qr` function:

```scheme
(define (mx-qr mat . rev)
  (define qrows (matrix-rows mat))
  (define rrows (fxmin (matrix-cols mat) qrows))
  (let rho [(Ak (matrix-explode mat))
            (Q  nil)
            (R  nil)]
    (if (empty? Ak) (cons (matrix-implode Q) (matrix-implode (if (optional rev #t) (reverse R) R)))
                    (let [(Rk (make-matrix* rrows 1 0.0))]
                      (rho (tail Ak)
                           (let cmgs [(i  0)
                                      (Qk (head Ak))
                                      (Qi Q)]
                             (cond [(fx>= i qrows) Q]
                                   [(empty? Qi)    (append* Q (mx/ Qk (matrix-set! Rk i 0 (mx-norm Qk 'fro))))]
                                   [else           (cmgs (fx+1 i) (mx-axpy (fpneg (matrix-set! Rk i 0 (matrix-scalar (head Qi) Qk))) (head Qi) Qk) (tail Qi))]))
                           (cons Rk R))))))
```
It consists of a tail-call recursion over all columns of the input matrix `A`,
in which the another recursion (`cgms` **C**olumn-wise **M**odified **G**ram-**S**chmidt) over all currently orthogonal columns of `Q` runs, that in turn computes an inner product to obtain the necessary projections,
and yields the to be expected cubic complexity.

This function is meant to be used if the `Q` and `R` matrices are required explicitly,
for example for use in higher-level algorithms, like the Francis-QR algorithm.

In case linear systems for one linear operator (matrix) `A` but many different right-hand sides `b_i`
are supposed to be solved, the function `mx-solver` first computes a QR decomposition via `mx-qr` and
then returns a (closure) function that if called with a suitable vector performs forward substitution.
Also, I note that the optional boolean argument `rev` of `mx-qr` is only supposed to be used by this
function, as it can save considerable work for large matrices.

If only one linear system needs solving the all-in-one function `mx-solve`,
expecting a matrix and a vector, is available.

Another convenience wrapper is the function `mx-orth` which just computes the `Q` factor of a matrix.
To compute the absolute value of the determinant,
the function `mx-absdet` computes the product of diagonal values of the factor `R`;
and for positive-definite matrices, the `mx-logdet` function computes the logarithm of the determinat
as the sum of logarithms of `R`'s diagonal.

Finally, if you want to learn more about the Gram-Schmidt algorithm, I recommend
the review paper "[Gram-Schmidt orthogonalization: 100 years and more](https://doi.org/10.1002/nla.1839)",
of which you can find non-paywalled documents via [google scholar](https://scholar.google.com/scholar?hl=en&as_sdt=0%2C5&q=intitle%3A%22Gram-Schmidt+orthogonalization%3A+100+years+and+more%22&btnG=).
In the final part of this series, I will layout **matrico**'s time stepping solvers.
