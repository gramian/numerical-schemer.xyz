---
title: "Matrico Algorithms III"
date: 2023-10-05
tags: Lisp Scheme SchemeLang Chicken-Scheme matrico
---

## `matrico`'s Differential Equation Solvers

* [Part I](http://numerical-schemer.xyz/2023/07/10/matrico-algorithms-i.html)
* [Part II](https://numerical-schemer.xyz/2023/08/21/matrico-algorithms-ii.html)
* Part III

### The Significance Differential Equations

Differential equations are mathematical problems described by differentials of the solution function `x`.
Ordinary differential equations (ODEs) are the basic type of differential equations in one variable,
for example, time `t`:
```
ẋ(t) = f(t,x(t))
```
Dynamical systems, systems evolving over time defined by their derivatives, can be described by such ODEs.
Note, that this is already a special case of an ODE, particularly,
the single first-order differential is already isolated (explicit).
More generally, an ODE can be of the implicit form:
```
0 = f(t,x(t),ẋ(t),ẍ(t),…)
```
Partial differential equations (PDEs) are an even more general type of differential equations in multiple variables,
for example one spatial and one time dimension.
Numerically, PDEs are typically first reduced to ODEs (by means of discretization), which are then solved in the remaining dimension again by discretization.
This makes solvers for ODEs so important.

Generally, most numerical solvers work by [discretization](https://en.wikipedia.org/wiki/Discretization),
which means the solution function's continuous argument variable is replaced by a set of samples,
for example a regular grid.
A famous and popular class of numerical ODE solvers are [Runge-Kutta (RK) methods](https://en.wikipedia.org/wiki/Runge%E2%80%93Kutta_methods#Explicit_Runge%E2%80%93Kutta_methods).
RK methods can be categorized as implicit and explicit.
The former (implicit) methods allow the solution of [stiff systems](https://en.wikipedia.org/wiki/Stiff_equation) but require the solution of a linear system (for linear implicit systems),
or a root-finding problem (for nonlinear implicit systems), in each step;
the latter (explicit) systems compute the next discretization step only based on previous steps.
`matrico` provides only explicit RK methods, but given the QR decomposition, simple yet powerful methods
like the first-order diagonally implicit RK (DIRK) method can be implemented with `matrico`.


Before, looking at the solver algorithms, I introduce an extension to ODEs that is central to
[system theory and control theory](https://en.wikipedia.org/wiki/Control_theory): [Input-output systems](https://en.wikipedia.org/wiki/State-space_representation).
An input-output system consists of an ODE (in `x`), and an output function `y`:
```
ẋ(t) = f(t,x(t))
y(t) = g(t,x(t))
```
Note that this is a one-directional coupling, meaning `f` cannot depend on `y`.
Again, a more general case would be a differential algebraic equation system,
coupling bi-directionally differential equations and algebraic equations.
However, `matrico` supports solving not only ODEs but input-output system directly.

### `matrico`'s Generic Input-Output System Time Stepper

The explicit RK single-step solvers provided by `matrico` are implemented in two parts:
A generic `stepper` function and another function defining the specifics of the solver algorithm. 
The `stepper` handles the discretization and wraps the solver
and returns the solution trajectory as matrix,
where each column represents a discretization step's solution.
Practically, this is implemented as a tail recursion:
```scheme
(define (stepper typ sys tim x0)
  (let [(vec (if (pair? sys) (head sys) sys))
        (out (if (pair? sys) (tail sys) (lambda (x y) y)))
        (dt (head tim))
        (tf (tail tim))]
    (let rho [(step   0.0)
              (state  x0)
              (series (list (out 0.0 x0)))]
      (if (fp> step tf) (matrix-implode (reverse series))
                        (let [(next (typ vec dt step state))]
                          (rho (fp+ step dt) next (cons (out step next) series)))))))
```
Depending on whether a vector field or a pair of a vector field and an output function is given,
either an ODE system or an input-output system is approximately solved. 

### Explicit Runge-Kutta Steppers

`matrico` provides two families of explicit Runge Kutta methods,
where "explicit" refers to the direct evaluability of the next step, given the previous step.
There is a multitude of RK methods which emphasize certain solution properties.
The core property is convergence of a certain order,
where the order determines what order of polynomials can be approximated numerically exact.
However, in case of RK methods, there are degrees of freedom beyond those required for convergence.
These extra degrees of freedom can be used to optimize for other properties.
Typically, stability is a property that is optimized for.
Among the many stability concepts of RK methods, the `matrico` solvers aim to maximize
the [region of absolute stability](https://en.wikipedia.org/wiki/Stiff_equation#A-stability)
in certain ways.
This stability needs to be scalable, meaning the computational complexity and the stability region expanse
can be balanced; so, the more computational effort is invested, the larger the stability region.

Furthermore, both RK methods provided by `matrico` also feature minimal memory requirements.
This means in terms of RK methods two registers (vectors) of the problem dimension,
one as the global accumulator (accumulating over all steps),
and one as local accumulator (accumulating over a step's sub-steps).
Details about memory efficiency of RK methods can be found in "[Runge–Kutta methods with minimum storage implementations](https://doi.org/10.1016/j.jcp.2009.11.006)".

### The 2nd Order SSP Runge-Kutta Method

The first family of RK methods `matrico` provides is a second-order accurate method,
which in its 2-stage form is equivalent to the [Heun method](https://en.wikipedia.org/wiki/Heun%27s_method).
This method is 2nd order optimal strong stability preserving (SSP),
meaning their [total variation](https://en.wikipedia.org/wiki/Total_variation_diminishing) does not increase over time.
While total variation is a useful property (particularly for hyperbolic PDEs),
more important is the per-stage increase of stability in all directions but positive real (right).

```scheme
(define (mx-ode2-ssp num sys tim x0)
  (let* [(s-1 (fp (fx-1 num)))
         (ssp (lambda (vf dt tk xk)
                (let [(dt/s-1 (fp/ dt s-1))]
                  (let rho [(cur (fx-1 num))
                            (c   tk)
                            (ret xk)]
                    (if (fx=0? cur) (mx/ (mx-axpy dt (vf c ret) (mx-axpy s-1 ret xk)) (fp num))
                                    (rho (fx-1 cur) (fp+ tk dt/s-1) (mx-axpy dt/s-1 (vf c ret) ret)))))))]
    (time-stepper ssp sys tim x0)))
```
For more information about this RK solver, see the paper:
"[Highly Efficient Strong Stability-Preserving Runge-Kutta Methods with Low-Storage Implementations](https://doi.org/10.1137/07070485X)".

### The 2nd Order Hyperbolic Runge-Kutta Methods

This is a second-order accurate method, which in its 2-stage form is equivalent to the [explicit midpoint method](https://en.wikipedia.org/wiki/Midpoint_method).
The extra degrees of freedom are used for this method to maximize the region of stability in the positive imaginary direction, and is in fact optimal in this regard.
This has the benefit of increased stability for systems that result from spatial discretization of
hyperbolic PDEs, thus the name hyperbolic RK method.
The table of values used for the algorithm's coefficients are taken from "[System Order Reduction for Gas and Energy Networks](https://doi.org/10.1002/pamm.202200201)".

```scheme
(define (mx-ode2-hyp num sys tim x0)
  (let* [(coeff (case num [ (2) (list (fp% 1 2)  1.0)]
                          [ (3) (list (fp% 1 3)  (fp% 1 2)  1.0)]
                          [ (4) (list (fp% 1 4)  (fp% 2 6)  (fp% 1 2)  1.0)]
                          [ (5) (list (fp% 1 5)  (fp% 2 10) (fp% 1 3)  (fp% 1 2)   1.0)]
                          [ (6) (list (fp% 1 6)  (fp% 2 15) (fp% 1 4)  (fp% 8 24)  (fp% 1 2) 1.0)]
                          [ (7) (list (fp% 1 7)  (fp% 2 21) (fp% 1 5)  (fp% 8 35)  (fp% 1 3) (fp% 1 2)   1.0)]
                          [ (8) (list (fp% 1 8)  (fp% 2 28) (fp% 1 6)  (fp% 8 48)  (fp% 1 4) (fp% 1 3)   (fp% 1 2) 1.0)]
                          [ (9) (list (fp% 1 9)  (fp% 2 36) (fp% 1 7)  (fp% 8 63)  (fp% 1 5) (fp% 5 21)  (fp% 1 3) (fp% 1 2)   1.0)]
                          [(10) (list (fp% 1 10) (fp% 2 45) (fp% 1 8)  (fp% 8 80)  (fp% 1 6) (fp% 9 50)  (fp% 1 4) (fp% 1 3)   (fp% 1 2) 1.0)]
                          [(11) (list (fp% 1 11) (fp% 2 55) (fp% 1 9)  (fp% 8 99)  (fp% 1 7) (fp% 14 99) (fp% 1 5) (fp% 8 33)  (fp% 1 3) (fp% 1 2) 1.0)]
                          [(12) (list (fp% 1 12) (fp% 2 66) (fp% 1 10) (fp% 8 120) (fp% 1 8) (fp% 4 35)  (fp% 1 6) (fp% 14 75) (fp% 1 4) (fp% 1 3) (fp% 1 2) 1.0)]
                          [else (error 'mx-ode2-hyp "Invalid number of stages, should be in [2,12].")]))
         (hyp   (lambda (vf dt tk xk)
                  (let rho [(cur coeff)
                            (ret xk)]
                    (if (empty? cur) ret
                                     (rho (tail cur) (mx-axpy (fp* dt (head cur)) (vf (fp*+ dt (head cur) tk) ret) xk))))))]
    (time-stepper hyp sys tim x0)))
```
For more information about this method, see the paper:
"[One Step Integration Methods of Third-Fourth Order Accuracy with Large Hyperbolic Stability Limits](https://doi.org/10.1016/0378-4754(84)90056-9)"

Now interestingly, it looks like there is a pattern to these values,
but I was not yet able to figure it out.
If one would, coefficients for an arbitrary number of stages could be generated procedurally.

Lastly, I want to mention that some of the papers may be behind a paywall,
but a [Google Scholar](https://scholar.google.com) search for the titles may yield a publicly accessible version of these works.
And, if I find time, I will add the plots for the stability regions of these RK methods.

This concludes the series of blog posts about numerical algorithms in `matrico`.
Next, I will discuss how to potentially use or bundle CHICKEN.
