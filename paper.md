---
title: 'opty: Software for trajectory optimization and parameter identification using direct collocation'
tags:
  - optimal control
  - trajectory optimization
  - parameter identification
  - direct collocation
  - nonlinear programming
  - symbolic computation
authors:
 - name: Jason K. Moore
   orcid: 0000-0002-8698-6143
   affiliation: 1
 - name: Antonie van den Bogert
   orcid: 0000-0002-3791-3749
   affiliation: 2
affiliations:
 - name: University of California, Davis
   index: 1
 - name: Cleveland State University
   index: 2
date: 04 June 2017
bibliography: paper.bib
---

# Summary

opty is a tool for describing and solving trajectory optimization and parameter
identification problems based on symbolic descriptions of ordinary differential
equations that describe a dynamical system. The motivation for its development
resides in the need to solve optimal control problems of biomechanical systems.
The target audience is engineers and scientists interested in solving nonlinear
optimal control and parameter identification problems with minimal
computational overhead.

A user of opty is responsible for specifying the system's dynamics (the ODEs),
the cost or fitness function, bounds on the solution, and the initial guess for
the solution. opty uses this problem specification to derive the constraints
needed to solve the optimization problem using the direct collocation method
[@Betts2010]. This method maps the problem to a non-linear programming problem
and the result is then solved numerically with an interior point optimizer,
IPOPT [@Watcher2006] which is wrapped by cyipopt [@Cyipopt2017] for use in
Python. The software allows the user to describe the dynamical system of
interest at a high level in symbolic form without needing to concern themselves
with the numerical computation details. This is made possible by utilizing
SymPy [@Meurer2017] and Cython [@Behnel2011] for code generation and
just-in-time compilation to obtain wrap optimized C functions that are
accessible in Python.

Direct collocation methods have been especially successful in the field of
human movement [@Ackermann2010, @vandenBogert2012] because those systems are
highly nonlinear, dynamically stiff, and unstable with open loop control.
Typically, closed-source tools were used for multibody dynamics (e.g. SD/Fast,
Autolev), for optimization (SNOPT), and for the programming environment
(Matlab). Recently, promising work has been done with the Opensim/Simbody
dynamics engine [@LeeUmberger2016, @LinPandy2017], but this requires that
Jacobian matrices are approximated by finite differences. In contrast, opty
provides symbolic differentiation which makes the code faster and prevents poor
convergence when the severe nonlinearity causes finite differences to be
inaccurate. Furthermore, opty allows the system to be formulated using an
implicit differential equation, which often results in far simpler equations
and better numerical conditioning. The first application of opty was in the
identification of feedback control parameters for human standing
[@MooreTGCS2015]. It should be noted that opty can use any dynamic system model
and is not limited to human movement.

Presently, opty only implements first order (backward Euler) and second order
(midpoint Euler) approximations of the dynamics. Higher accuracy and/or larger
time steps can be achieved with higher order polynomials [@PattersonRao2014],
and opty could be extended towards such capabilities. In our experience,
however, the low order discretizations provide more robust convergence to the
globally optimal trajectory in a nonlinear system when a good initial guess is
not available [@Zarei2016].

There are existing software packages that can solve optimal control problems
and have have similarities to opty. Below, is a feature comparison:

```
+========+=====================+============+================+=============================+======+========================+==========+=================+=================================================================================================+
|        |                     |            |                |                             |      |                        | Implicit |                 |                                                                                                 |
| Name   | Citation            | Language   | License        | Derivatives                 | DAEs |  Discretization        | dynamics | Solvers         | Project Website                                                                                 |
+========+=====================+============+================+=============================+======+========================+==========+=================+=================================================================================================+
| Casadi | [@Andersson2013]    | - C++      | LGPL           | Automatic differentiation   | Yes  | None                   | Yes      | - IPOPT, WORHP  | `Casadi Website <https://github.com/casadi/casadi/wiki>`_                                       |
|        |                     | - Python   |                |                             |      |                        |          | - SNOPT, KNITRO |                                                                                                 |
|        |                     | - Octave   |                |                             |      |                        |          |                 |                                                                                                 |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| DIDO   | [@Ross2002]         | Matlab     | Commercial     | Analytic                    | No   | Pseudospectral         | Yes      | built-in        | `DIDO Website <http://www.elissarglobal.com/industry/products/software-3/>`_                    |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| DIRCOL | [@vonStryk1993]     | Fortran    | Non-commercial | Finite differences          | Yes  | Piecewise linear/cubic | Yes      | NPSOL, SNOPT    | `DIRCOL Website <http://www.sim.informatik.tu-darmstadt.de/en/res/sw/dircol/>`_                 |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| DYNOPT |                     |            |                |                             |      |                        |          |                 |                                                                                                 |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| GESOP  |                     |            |                |                             |      |                        |          |                 |                                                                                                 |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| GPOPS  | [@PattersonRao2014] | Matlab     | Commercial     | Automatic differentiation   | No   | Pseudospectral         | No       | SNOPT, IPOPT    | `GPOPS Website <http://www.gpops2.com/>`_                                                       |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| opty   |                     | Python     | BSD 2-Clause   | Analytic                    | Yes  | Euler, Midpoint        | Yes      | IPOPT           | `opty Documentation <http://opty.readthedocs.io>`_                                              |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| PROPT  | [@Rutquist2010]     | Matlab     | Commercial     | Analytic                    | Yes  | Pseudospectral         | Yes      | SNOPT, KNITRO   | `TOMDYN Website <http://tomdyn.com/index.html>`_                                                |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| PSOPT  | [@Becerra2010]      | C++        | GPL            | - Automatic differentiation | Yes  | Pseudospectral, RK     | No       | IPOPT, SNOPT    | `PSOPT Website <http://www.psopt.org/>`_                                                        |
|        |                     |            |                | - Sparse finite differences |      |                        |          |                 |                                                                                                 |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
| SOCS   | [@Betts2010]        | Fortran    | Commercial     | Finite differences          | Yes  | Euler, RK, & others    | Yes      | built-in        | `SOCS Documentation <http://www.boeing.com/assets/pdf/phantom/socs/docs/SOCS_Users_Guide.pdf>`_ |
+--------+---------------------+------------+----------------+-----------------------------+------+------------------------+----------+-----------------+-------------------------------------------------------------------------------------------------+
```

Each of these software packages offer a different combination of attributes and
features that make it useful for different problems. opty is the only package
that has a liberal open source license, following precedent set by other core
scientific Python packages. This allows anyone to use and modify the code
without having to share the source of their application. opty also is the only
package, open source or not, that allows (in fact forces) the user to describe
their problem via a high level symbolic mathematical description using the API
of a widely used computer algebra system. This relieves the user from having to
translate the much simpler continuous problem definition into a discretize NLP
problem. opty leverages the popular Scientific Python core tools like NumPy,
SymPy, and matplotlib allowing users to include opty code into Python programs.
set of tools. Lastly, the numeric code generated by opty to evaluate the NLP
constraints is highly optimized and provides extremely efficient [parallel]
evaluation of the contraints. This becomes very valuable for high dimensional
dynamics. opty currently does not offer a wide range of discretization methods
nor support for solvers other than IPOPT, but those could relatively easily be
added based on user need.

# References
