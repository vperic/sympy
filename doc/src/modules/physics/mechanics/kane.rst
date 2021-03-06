==================================
Kane's Method in Physics/Mechanics
==================================

:mod:`mechanics` has been written for use with Kane's method of forming
equations of motion [Kane1985]_. This document will describe Kane's Method
as used in this module, but not how the equations are actually derived.

Structure of Equations
======================

In :mod:`mechanics` we are assuming there are 5 basic sets of equations needed
to describe a system. They are: holonomic constraints, non-holonomic
constraints, kinematic differential equations, dynamic equations, and
differentiated non-holonomic equations.

.. math::
  \mathbf{f_h}(q, t) &= 0\\
  \mathbf{k_{nh}}(q, t) u + \mathbf{f_{nh}}(q, t) &= 0\\
  \mathbf{k_{k\dot{q}}}(q, t) \dot{q} + \mathbf{k_{ku}}(q, t) u +
  \mathbf{f_k}(q, t) &= 0\\
  \mathbf{k_d}(q, t) \dot{u} + \mathbf{f_d}(q, \dot{q}, u, t) &= 0\\
  \mathbf{k_{dnh}}(q, t) \dot{u} + \mathbf{f_{dnh}}(q, \dot{q}, u, t) &= 0\\

In :mod:`mechanics` holonomic constraints are only used for the linearization
process; it is assumed that they will be too complicated to solve for the
dependent coordinate(s).  If you are able to easily solve a holonomic
constraint, you should consider redefining your problem in terms of a smaller
set of coordinates.

Kane's method forms two expressions, :math:`F_r` and :math:`F_r^*`, whose sum
is zero. In this module, these expressions are rearranged into the following
form:

 :math:`\mathbf{M}(q, t) \dot{u} = \mathbf{f}(q, \dot{q}, u, t)`

For a non-holonomic system with "o" total speeds and "m" motion constraints, we
will get o - m equations. The mass-matrix/forcing equations are then augmented
in the following fashion:

.. math::
  \mathbf{M}(q, t) &= \begin{bmatrix} \mathbf{k_d}(q, t) \\
  \mathbf{k_{dnh}}(q, t) \end{bmatrix}\\
  \mathbf{_{(forcing)}}(q, \dot{q}, u, t) &= \begin{bmatrix}
  - \mathbf{f_d}(q, \dot{q}, u, t) \\ - \mathbf{f_{dnh}}(q, \dot{q}, u, t)
  \end{bmatrix}\\


Kane's Method in Physics/Mechanics
==================================

Formulation of the equations of motion in :mod:`mechanics` starts with creation
of a ``Kane`` object. Upon initialization of the ``Kane`` object, an inertial
reference frame needs to be supplied. ::

  >>> from sympy.physics.mechanics import *
  >>> N = ReferenceFrame('N')
  >>> KM = Kane(N)

Next is the specification of coordinates and speeds for Kane's Method.

  >>> q1, q2, u1, u2 = dynamicsymbols('q1 q2 u1 u2')
  >>> q1d, q2d, u1d, u2d = dynamicsymbols('q1 q2 u1 u2', 1)
  >>> KM.coords([q1, q2])
  >>> KM.speeds([u1, u2])

It is also important to supply the order of coordinates and speeds properly if
there are dependent coordinates and speeds. They must be supplied after
independent coordinates and speeds or as a keyword argument; this is shown
later: ::

  >>> q1, q2, q3, q4 = dynamicsymbols('q1 q2 q3 q4')
  >>> u1, u2, u3, u4 = dynamicsymbols('u1 u2 u3 u4')
  >>> # Here we will assume q2 is dependent, and u2 and u3 are dependent
  >>> # We need the constraint equations to enter them though
  >>> KM.coords([q1, q3, q4])
  >>> KM.speeds([u1, u4])

Additionally, if there are auxiliary speeds, they need to be identified here.
See the examples for more information on this. In this example u4 is the
auxiliary speed. ::

  >>> u1, u2, u3, u4 = dynamicsymbols('u1 u2 u3 u4')
  >>> KM.speeds([u1, u2, u3], u_auxiliary=[u4])

Kinematic differential equations must also be supplied; there are to be
provided as a list of expressions which are each equal to zero. A trivial
example follows: ::

  >>> KM.coords([q1, q2])
  >>> kd = [q1d - u1, q2d - u2]
  >>> KM.kindiffeq(kd)

A dictionary returning the solved :math:`\dot{q}`'s can also be solved for: ::

  >>> mechanics_printing()
  >>> KM.kindiffdict()
  {q1': u1, q2': u2}

Turning on ``mechanics_printing()`` makes the expressions significantly
shorter and is recommended.

If there are non-holonomic constraints, dependent speeds need to be specified
(and so do dependent coordinates, but they only come into play when linearizing
the system). The dependent speeds need to be specified in the same order as
they were earlier. The constraints need to be supplied in a list of expressions
which are equal to zero, trivial motion and configuration constraints are shown
below: ::

  >>> N = ReferenceFrame('N')
  >>> KM = Kane(N)
  >>> q1, q2, q3, q4 = dynamicsymbols('q1 q2 q3 q4')
  >>> u1, u2, u3, u4 = dynamicsymbols('u1 u2 u3 u4')
  >>> #Here we will assume q2 is dependent, and u2 and u3 are dependent
  >>> speed_cons = [u2 - u1, u3 - u1 - u4]
  >>> coord_cons = [q2 - q1]
  >>> KM.coords([q1, q3, q4], qdep=[q2], coneqs=coord_cons)
  >>> KM.speeds([u1, u4], udep=[u2, u3], coneqs=speed_cons)

The final step in forming the equations of motion is supplying a list of
bodies and particles, and a list of 2-tuples of the form ``(Point, Vector)``
or ``(ReferenceFrame, Vector)`` to represent applied forces and torques.

  >>> N = ReferenceFrame('N')
  >>> q, u = dynamicsymbols('q u')
  >>> qd, ud = dynamicsymbols('q u', 1)
  >>> P = Point('P')
  >>> P.set_vel(N, u * N.x)
  >>> Pa = Particle('Pa', P, 5)
  >>> BL = [Pa]
  >>> FL = [(P, 7 * N.x)]
  >>> KM = Kane(N)
  >>> KM.coords([q])
  >>> KM.speeds([u])
  >>> KM.kindiffeq([qd - u])
  >>> (fr, frstar) = KM.kanes_equations(FL, BL)
  >>> KM.mass_matrix
  [-5]
  >>> KM.forcing
  [-7]

When there are motion constraints, the mass matrix is augmented by the
:math:`k_{dnh}(q, t)` matrix, and the forcing vector by the :math:`f_{dnh}(q,
\dot{q}, u, t)` vector.

There are also the "full" mass matrix and "full" forcing vector terms, these
include the kinematic differential equations; the mass matrix is of size (n +
o) x (n + o), or square and the size of all coordinates and speeds. ::

  >>> KM.mass_matrix_full
  [1,  0]
  [0, -5]
  >>> KM.forcing_full
  [ u]
  [-7]

The forcing vector can be linearized as well; its Jacobian is taken only with
respect to the independent coordinates and speeds. The linearized forcing
vector is of size (n + o) x (n - l + o - m), where l is the number of
configuration constraints and m is the number of motion constraints. Two
matrices are returned; the first is an "A" matrix, or the Jacobian with respect
to the independent states, the second is a "B" matrix, or the Jacobian with
respect to 'forces'; this can be an empty matrix if there are no 'forces'.
Forces here are undefined functions of time (dynamic symbols); they are only
allowed to be in the forcing vector and their derivatives are not allowed to be
present. If dynamic symbols appear in the mass matrix or kinematic differential
equations, an error with be raised. ::

  >>> KM.linearize()[0]
  [0, 1]
  [0, 0]

Exploration of the provided examples is encouraged in order to gain more
understanding of the ``Kane`` object.
