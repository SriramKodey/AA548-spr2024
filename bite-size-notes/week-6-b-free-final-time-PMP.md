# Free Final Time with Pontryagin's Minimum Principle

## Introduction
The problem of free final time refers to leaving the terminal time as a optimizable variable in an optimal control problem. This topic is well-understood for linear systems where analytical solutions are commonly available [1], but requires numerical methods to handle the increasing complexity of nonlinear dynamics and/or higher-order linear systems.

Free final time is an important topic in aerospace engineering, as fuel costs are inextricably tied to time of flight and many astronautical missions rely on a window of opportunity. Having the flexibility of directly optimizing time also aids in the robustness of any system that might experience changing dynamical situations commonly encountered in aerospace.

The method explored in particular is Pontryagin's Minimum Principle (PMP), which is computationally-efficient and can provide insights into sensitivity of the solution, but lacks global optimality guarantees unlike the more popular Hamilton-Jacobi-Bellman-Isaacs (HJBI) equation.

## Scope

These notes will focus on introducing free final time on a simple linear system with theory and code, explain difficulties with implementation on nonlinear and higher-order linear systems, and provide resources to explore the subject further.

Learning objectives include...
* Introducing an extension to function theory: the functional.
* Understanding the similarities of extremizing functions of a functional with finding critical points.
* Determining necessary conditions for optimality with the Hamiltonian under Pontryagin's Minimum Principle.
* Constructing a free final time problem of a simple linear system.

The reader is expected to have prior knowledge of undergraduate-level controls and topics such as basic optimization and the Lagrangian.

## Prerequisites

### Nomenclature
<details close>
<summary> <em> (open to view)</em> </summary>
<br>
<div markdown="1">

| Term            | Definition                                      |
|-----------------|-------------------------------------------------|
| $a$ | acceleration |
| $H$ | Hamiltonian |
| $J$ | cost function/functional |
| $L$ | Lagrangian |
| $t$ | time |
| $T$ | terminal time |
| $u$ | control |
| $v$ | velocity |
| $x$ | position in $x$-direction |
| $x_g$ | goal position in $x$-direction |
| $y$ | position in $y$-direction |
| $\theta$ | angle |
| $\lambda$ | co-state |
| $\omega$ | angular velocity |
| $\Psi$ | terminal cost |

</div>
</details>

### Motivation
Why do we need new math to solve free final time?

For free final time problem, terminal time $T$ is no longer fixed. Thus when finding an optimal solution, the state $z$ and control $u$ are no longer the optimization variables in a cost function, but rather $z(t)$, $u(t)$ are optimization *functions* and $T$ is an optimization variable in a cost *functional*.

Put another way: rather than finding extrema (minima/maxima) values for a function, we instead find extrema functions, functions that minimize or maximize a functional.

### Functionals
Functionals are mappings from a set of functions to the set of real numbers. Intuitively, one can think of them as "functions of functions", as functions are mappings from a set of numbers to another set of numbers. The calculus of variations uses small changes of functions and functionals, known as variations, to determine the extrema of functionals.

The purpose behind using the calculus of variations for free final time is to find a extrema function that minimizes/maximizes a functional. One of these similarities is setting the first variation to equal zero to find an extrema of a functional, analogous to the first derivative test that finds the critical points by setting the derivative of a function equal to zero.

Our optimization cost is then represented by a cost functional $J[z(t),u(t),t]$ rather than a cost function $J\big(z(t),u(t),t\big)$. More precisely,

$$\underbrace{J[z(t),u(t),t]}\_{\text{cost functional}} = \int_0^T \underbrace{L\big(z(t), u(t), t\big)}\_{\text{cost-to-go}} dt + \underbrace{\Psi(T)}\_{\text{terminal cost}}$$

where $L$ is the Lagrangian, $\Psi$ is a terminal cost function, and $T$ is the terminal time.

#### Sufficient conditions for optimality

If control is not an optimization function, the Euler-Lagrange equation from calculus of variations is a necessary condition for optimality of a functional.

However, if control is also being optimized, Pontryagin's Minimum Principle gives necessary conditions for minimization of a functional, with the condtions being derived from variational calculus. If transversality conditions are included and the Hamiltonian w.r.t. the state variables is strictly convex, that set of conditions is also sufficient. [2]

The convexity of the Hamiltonian w.r.t. the state variables can also be checked by a second partial derivative w.r.t. the state of the Hamiltonian, also known as the Hessian.

### Pontryagin's Minimum Principle (PMP)

Let the cost functional be

$$\underbrace{J[z(t),u(t),t]}\_{\text{cost functional}} = \int_0^T \underbrace{L\big(z(t), u(t), t\big)}\_{\text{cost-to-go}} dt +  \underbrace{\Psi(T)}\_{\text{terminal cost}}$$
as defined above.

The constraints from the system dynamics are added to the Lagrangian $L$ with time-varying Lagrange multipliers $\lambda(t)$  whose elements are called the co-states of the system.

We can then construct the Hamiltonian $H$:

$$
\begin{equation}
H\big(z(t), u(t), \lambda(t), t\big) = \lambda(t)^\top \cdot \dot{z} + L\big(z(t), u(t), t\big).
\end{equation}
$$

Pontryagin's minimum principle states that the optimal parameters must minimize the Hamiltonian $H$ for all time $t \in [0, T]$ and all feasible controls $u$ so that

$$
H(z^\*(t),u^\*(t),\lambda^\*(t),t) \leq H(z(t),u(t),\lambda(t),t).
$$


$$
\begin{align}
-\dot{\lambda}(t)
&= \nabla_z H(z^\*(t), u^\*(t), \lambda^\*(t), t) \\ 
&= \lambda(t)^\top \cdot \nabla_z \dot{z}(z^\*(t), u^\*(t)) + \nabla_z L(z^\*(t), u^\*(t)) 
\end{align}
$$



The above equations are necessary for optimality with fixed final time, this last condition is for free final time.

$$
\begin{equation}
\nabla_T \Psi(z(T)) + H(T) = 0
\end{equation}
$$

### Transversality Conditions

For free final time, the transversality conditions state that at the optimal terminal time $T$, the Hamiltonian $H$ should satisfy
$$H\big(z(T),u(T),\lambda(T),T\big) = 0.$$ This implies that no further performance can be achieved by changing $T$.

The co-state conditon ensures that the variations in the state at the terminal time are linked to the changes in the terminal cost.

$$
\begin{equation}
\lambda(T)^\top = \nabla_z \Psi(z(T))
\end{equation}
$$

## Example: 1D Motion

Now let's look at a standard optimal control problem, but this time with terminal time being an optimization variable.

Let's revisit a common example: a 1D system in motion. We seek to reach a goal state and minimize both effort and time doing so.
* state variable $z$
  * positions $x$
  * velocity $v$
* control input $u$
  * acceleration $a$
* state variable $z_g$
  * goal positions $x_g$
  * goal velocity $v_g$
* co-state $\lambda$
* Lagrange multiplier $\mu$
* maximum acceleration $a_{\text{max}}$

Then the state and dynamics vectors, still functions of time, are:

$$
z = \begin{bmatrix}
  x \\
  v \\
\end{bmatrix}, \qquad
\text{and}
\qquad \dot{z} = \begin{bmatrix}
  v \\
  a \\
\end{bmatrix}
$$

Let the Lagrangian $L(z(t), u(t), t\big) = \|x-x_g\|^2 + v^2$ and the terminal cost be $\Psi(T) = \|z(T) - z_g\|^2$. Include a constraint on the control: $||a||\_2 \le a_{\text{max}}$.

The cost functional can be written as:

$$
\begin{align}
J[z(t),u(t),t]
&= \int\_0^T L\big(z(t), u(t), t\big) dt + \Psi(T)\\
&= \int\_0^T \|x-x_g\|^2 + v^2 dt + \|z(T) - z\_g\|^2
\end{align}
$$

Augment the Lagrangian with the control constraint.

$$L(z(t), u(t), t\big) = \|x-x_g\|^2 + v^2 + \theta^2 + \mu (a_{\text{max}} - ||a||\_2) $$

Then the Hamiltonian augments the state dynamics to the Lagrangian.

$$H(z(t), u(t), t\big) = \lambda(t)^\top \cdot \dot{z} + \|x-x_g\|^2 + v^2 + \mu (a_{\text{max}} - a) $$

Taking the Hessian of the Hamiltonian reveals that it is positive definite, thus the Hamiltonian is convex and can result in a globally optimal answer.

$$
\frac{\partial H}{\partial z} = \begin{bmatrix} 2 & 0 \\
0 & 2 \end{bmatrix} \succ 0
$$

By Pontryagin’s minimum principle and transversality conditions, we must satisfy these conditions to be optimal.

$$
H(z^\*(t), u^\*(t), \lambda^\*(t), t) \leq H(z(t), u(t), \lambda(t), t) \text{  for  } t \in [0,T]
$$

$$
-\dot{\lambda}(t) = \lambda(t)^\top \cdot \nabla_z \dot{z}(z^\*(t), u^\*(t)) + \nabla_z L(z^\*(t), u^\*(t)) 
$$

$$
\lambda(T)^\top = \nabla_z \Psi(z(T))
$$

$$
\nabla_T \Psi(z(T)) + H(T) = 0
$$

$$H\big(z(T),u(T),\lambda(T),T\big) = 0$$

This can then be solved using shooting methods.

## Conclusion

In these notes, we've hopefully provided a base understanding of free final time as solved with Pontryagin’s Minimum Principle. With these notes, it's hoped that the reader gains a more intuitive understanding of free final time.

To continue exploring this topic, references that were used in these notes are provided in the next section.

## References
[1] Donald E. Kirk, Optimal Control Theory: An Introduction. Dover Publications, 2012, Pages 107-309.
[2] Morton I Kamien, Nancy L Schwartz, Sufficient conditions in optimal control theory, Journal of Economic Theory, Volume 3, Issue 2,
1971, Pages 207-214, ISSN 0022-0531, https://doi.org/10.1016/0022-0531(71)90018-4.
[3] David G. Hull, Optimal Control Theory for Applications, Springer, 2003, Pages 221-246.

