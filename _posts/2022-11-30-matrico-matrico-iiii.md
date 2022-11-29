---
title: "Matrico Module IIII"
date: 2022-11-30
tags: Lisp Scheme SchemeLang matrico matrico-Module
---

## `matrico`'s `matrico` Module (Part IIII)

**Series**:

* [Part I](http://numerical-schemer.xyz/2022/09/16/matrico-matrico-i.html)
* [Part II](http://numerical-schemer.xyz/2022/09/30/matrico-matrico-ii.html)
* [Part III](http://numerical-schemer.xyz/2022/10/17/matrico-matrico-iii.html)
* Part IIII

For the fourth part in this series of posts about the main `matrico` module of the `matrico` project,
I summarize the core numerical functionality.
These are again contained in the [`src/mx.scm`](https://github.com/gramian/matrico/blob/main/src/mx.scm) Scheme source code file,
that is **included** into the module's main file `matrico.scm`.

### Linear Algebra

Usual basic linear algebra functions that `matrico` provides are concatenation of matrices,
[vectorization](https://en.wikipedia.org/wiki/Vectorization_(mathematics)) and [transposition](https://en.wikipedia.org/wiki/Transpose) of matrices, [symmetric and anti-symmetric (skew-symmetric) part](https://en.wikipedia.org/wiki/Symmetric_matrix#Decomposition_into_symmetric_and_skew-symmetric)s of a matrix,
and extracting the [diagonal](https://en.wikipedia.org/wiki/Diagonal_matrix#Matrix-to-vector_diag_operator) of a matrix:

* `(mx-horcat A B)` - Returns **matrix** consisting of a copy of horizontally concatenated argument **matrix**es.
* `(mx-vercat A B)` - Returns **matrix** consisting of a copy of vertically concatenated argument **matrix**es.
* `(mx-vec A)` - Returns column-**matrix** with entries of argument **matrix** column-wise stacked.
* `(mx-transpose A)` - Returns **matrix** with entries of argument **matrix** with swapped row and column indices.
* `(mx-sympart A)` - Returns **matrix** being the symmetric part of argument **matrix**.
* `(mx-skewpart A)` - Returns **matrix** being the skew-symmetric part of argument **matrix**.
* `(mx-diagonal A)` - Returns column-**matrix** being the diagonal of argument **matrix**.

#### Linear Problems

A core competency of numerical linear algebra is the solution of linear problems,
meaning solving a system of linear equations, typically presented in vectorized form
as matrix times unknown vector equals vector: `A x = b`.
A class of algorithms to solve such linear problems utilize matrix decompositions.
The idea behind these approaches is to factorize the matrix `A` such that the factors
facilitate a solution.
Among the available matrix decomposition algorithms, I selected the [QR decomposition](https://en.wikipedia.org/wiki/QR_decomposition) (or QR factorization).
This algorithm factors the matrix `A = QR` into an orthogonal matrix `Q` and a right triangular matrix `R`.
While the former (`Q`) can be [inverted by transposition](https://en.wikipedia.org/wiki/Orthogonal_matrix),
the latter (`R`) corresponds to [back substitution](https://en.wikipedia.org/wiki/Triangular_matrix);
this means `A = QR => QR x = b => R x = Q^T b`.
Advantages of the QR decomposition compared to other decompositions are on the one hand, that the matrix `A`
does not need to be symmetric, invertible, or even square (in the singular or rectangular case a [least-squares solution](https://en.wikipedia.org/wiki/Numerical_methods_for_linear_least_squares#Orthogonal_decomposition_methods) is obtained),
on the other hand, the QR decomposition is the basis for more complex algorithms,
such as the [Singular Value Decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition#Numerical_approach) (SVD), or eigenvalue computation via the [Francis-QR algorithm](https://en.wikipedia.org/wiki/QR_algorithm).
Among the various variants of practical QR factorization algorithms, I specifically chose the Gram-Schmidt-based [Schwarz-Rutishauser](https://towardsdatascience.com/can-qr-decomposition-be-actually-faster-schwarz-rutishauser-algorithm-a32c0cde8b9b) algorithm,
also known as [column-wise modified Gram Schmidt](https://doi.org/10.1002/nla.1839);
because first, this column-wise algorithm fits `matrico`'s matrix data structure,
second, its lower complexity, third, its numerical stability, and lastly its simple implementation.

`matrico` provides functions computing QR decompositions for different applications, 
all using the same core algorithm:

* `(mx-qr A)` - Returns a pair of **matrix**es (Q and R) for an argument **matrix**
* `(mx-solver A)` - Returns a **function** (expecting a column matrix argument) that solves the linear problem formed with the argument **matrix**. 
* `(mx-solve A b)` - Returns a column matrix solving the linear problem formed by argument **matrix** and column **matrix**.

Use `mx-qr` if no linear problem is to be solved, but the factor matrices are required,
for example in [orthogonalization](https://en.wikipedia.org/wiki/Orthogonalization).
Use `mx-solver` if multiple linear problems need to be solved with the same matrix `A` but different right-hand-sides `b`, as the returned function caches the QR decomposition as closure and thus prevents recomputation.
Use `mx-solve` if only a single solution to a linear problem is required.

#### Determinants

The QR decomposition also allows to compute a matrix's [determinant](https://en.wikipedia.org/wiki/QR_decomposition#Connection_to_a_determinant_or_a_product_of_eigenvalues) up to its sign.
Thus, the absolute value of the determinant is computable.
Given a [positive definite matrix](https://en.wikipedia.org/wiki/Definite_matrix), also the [logarithm of the determinant](https://en.wikipedia.org/wiki/Determinant#Trace) can be computed,
which is useful for badly conditioned matrices. 

* `(mx-absdet A)` - Returns the absolute value of the determinant of the argument **matrix**.
* `(mx-logdet A)` - Returns the logarithm of the determinant of the argument **matrix**.

#### Traces

Functions computing accumulated quantites from the matrix diagonal are subsumed under "Traces".
Usually, a matrix trace refers to the sum of its diagonal values;
here, also a the product of diagonal values is provided,
as well as functions calculating the (additive) trace of a product of matrices,
since the [trace of a product](https://en.wikipedia.org/wiki/Trace_(linear_algebra)#Trace_of_a_product) can be computed more efficiently than computing first the product and then its trace.

* `(mx-trace A)` - Returns the sum of diagonal entries of the **matrix** argument.
* `(mx-multrace A)` - Returns the product of diagonal entries of the **matrix** argument.
* `(mx-prodtrace A B)` - Returns the sum of diagonal entries of the product of **matrix** arguments.
* `(mx-prodtrace* AT B)` - Returns the sum of diagonal entries of the product of **matrix** arguments, with first factor transposed.

#### Matrix multiplication

Matrix multiplication is also an important pillar of (numerical) linear algebra.
It has interpretations, among others, as compositions of linear transformations, 
or all possible inner products of two sets vectors (column matrices).
`matrico` provides variants of the generic [matrix multiplication](https://gramian.github.io/numerical-schemer.xyz/2022/08/04/matrico-matrix.html)
mainly for convenience:

* `(mx-scalar AT B)` - Returns scalar product (inner product) of column-**matrix** arguments.
* `(mx-dyadic A B)` - Returns dyadic product (outer product) of column-**matrix** argument and row-**matrix** argument.
* `(mx-dot* AT B)` - Returns product of **matrix** arguments with first factor transposed.
* `(mx-dot A B)` - Returns product of **matrix** arguments.
* `(mx-gram A)` - Returns product of transposed **matrix** argument with itself.
* `(mx-gram* A)` - Returns product of **matrix** argument with its transposed self.
* `(mx-square A)` - Returns product of **matrix** argument with itself.

The `mx-dot*` functions should always be preferred for non-column-matrices, as it is the most performant.

### Multivariate Statistics

Some statistical functions are available in `matrico`, too:
[cross](https://en.wikipedia.org/wiki/Cross-covariance_matrix)-[covariance](https://en.wikipedia.org/wiki/Covariance_matrix),
[cross](https://en.wikipedia.org/wiki/Cross-correlation_matrix)-[correlation](https://en.wikipedia.org/wiki/Autocorrelation#Matrix),
[variance](https://en.wikipedia.org/wiki/Variance), [standard deviation](https://en.wikipedia.org/wiki/Standard_deviation),
[angle](https://en.wikipedia.org/wiki/Cosine_similarity), and [coherence](https://en.wikipedia.org/wiki/Coherence_(signal_processing)).
For these functions the argument matrices are assumed to be arranged as
each row corresponding to all observations of one variable,
while each column represents all variables of one observation:

* `(mx-xcov A B)` - Returns cross-covariance matrix of **matrix** arguments.
* `(mx-cov A)` - Returns covariance of **matrix** argument.
* `(mx-var A)` - Returns variance of **matrix** argument.
* `(mx-std A)` - Returns standard deviation of **matrix** argument.
* `(mx-xcor A B)` - Returns cross-correlation of **matrix** arguments.
* `(mx-cor A)` - Returns correlation of **matrix** argument.
* `(mx-angle A B)` - Returns angles between **matrix** argument columns.
* `(mx-coher A B)` - Returns coherence of **matrix** arguments.

### Analysis and Calculus

Analysis deals with derivatives and integrals,
so numerical analysis approximates derivation and integration.
`matrico` provides a function to compute differences of consecutive columns of a matrix,
which can be used for a [finite difference](https://en.wikipedia.org/wiki/Finite_difference) approximation of a [difference quotient](https://en.wikipedia.org/wiki/Difference_quotient).
To approximate integrals by [Riemann sums](https://en.wikipedia.org/wiki/Riemann_sum), a column-wise [trapezoid rule](https://en.wikipedia.org/wiki/Trapezoidal_rule) is implemented.

* `(mx-diff A)` - Returns **matrix** of differences of consecutive columns of argument **matrix**.
* `(mx-trapz A)` - Returns column-**matrix** of sums of consecutive columns of argument **matrix**.

#### Ordinary Differential Equations

A central subject of numerical analysis is the approximation of solutions to differential equations.
`matrico` is focusing on [ordinary differential equations](https://en.wikipedia.org/wiki/Ordinary_differential_equation) (ODE), differential equations in a one dimensional variable,
as opposed to partial differential equations in multiple dimensional variables.
Ordinary differential equations are prominently used for [dynamical systems](https://en.wikipedia.org/wiki/Dynamical_system),
where the variable is (one-dimensional) time, and describes the temporal evolution of a system.
Integrators, such as [Runge-Kutta methods](https://en.wikipedia.org/wiki/Runge%E2%80%93Kutta_methods),
compute a sequence of point-wise approximations to the solution function's values, called trajectory for evolution systems.
In `matrico` trajectories are obtained by a single step methods, meaning the next (time) step approximation depends only the current approximation step.
Opposed to, for example, [MATLAB](https://www.mathworks.com/help/matlab/math/choose-an-ode-solver.html), `matrico` uses fixed step methods,
and not [adaptive time step methods](https://en.wikipedia.org/wiki/Adaptive_step_size).

The two selected solvers implemented in `matrico` are explicit Runge-Kutta method of second (nonlinear) order,
and were chosen as they feature low complexity in terms of compute and memory.
Particularly, both methods are of the ["two register" class ("2R*")](https://doi.org/10.1016/j.jcp.2009.11.006);
this means that internally an iteration requires merely two vector (column-matrix) registers,
of which one is constant for each sub-step iteration.
This is the minimum memory consumption possible.
Furthermore, both methods induce a family of method as either allow to increase the number of sub-steps or stages, to increase stability,
and hence approximate more complicated problems:
First, the family of implemented [hyperbolic Runge-Kutta methods](https://doi.org/10.1016/0378-4754(84)90056-9) is aimed at transport, advection, or non-normal problems and require a stability region around the imaginary axis.
These hyperbolic Runge-Kutta methods have the maximum hyperbolic stability limit for an explicit second order method of chosen number of stages.
Second, the family of implemented [strong stability preserving Runge-Kutta methods](https://doi.org/10.1137/07070485X) is aimed at general and parabolic problems, which require a stability region in the negative-real half-plane.
Strong stability preserving (SSP) means that if a first order explicit Euler method adheres to some constraint for the given ODE, so will this second-order method, which is optimal among second-order methods in this regard.

* `(mx-ode2-hyp num sys tim x0)` - Returns **matrix** holding trajectory solved by the hyperbolic Runge-Kutta method with **fixnum** argument many stages, **procedure** argument vector field, **pair** argument variable discretization (step, horizon) and column-**matrix** initial value.
* `(mx-ode2-ssp num sys tim x0)` - Returns **matrix** holding trajectory solved by the strong stability preserving Runge-Kutta method with **fixnum** argument many stages, **procedure** argument vector field, **pair** argument variable discretization (step, horizon) and column-**matrix** initial value.

Instead of just a vector field also a **pair** of a vector field **procedure** and an output function **procedure** can be passed, which allows to simulate input-output systems, typical in [system theory](https://en.wikipedia.org/wiki/Dynamical_systems_theory) and [control theory](https://en.wikipedia.org/wiki/Control_theory).

This sums up the highest-level functionality of `matrico`.
The next post will mob up utility and private functions for completeness,
and thereby conclude this series.