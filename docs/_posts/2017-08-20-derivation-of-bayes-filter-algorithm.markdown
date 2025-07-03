---
layout: post
title:  "Bayes Filter: Probabilistic State Estimation for Robots"
date:   2017-08-20 08:00:00 -0000
categories:
- Robotics
mathjax: true
---

# Table of Contents
1. [Why is Bayes-filter useful?](#why-is-bayes-filter-useful)
2. [Bayes-filter](#bayes-filter)

### Why is Bayes-filter useful?  
For many autonomous robotics system, the action to take at each time period depends on its environment. For instance, a roomba (robotic vacuum cleaner) needs to know the position it is in a room before it can decide which way to move.  
The more a robot knows about the environment, the better the decision it can make. However measuring the environment is often uncertain, as it can be affected by among others,

- noise in measurement data (e.g., sensors such as LIDAR/RADAR having limited accuracy),  
- faulty sensors,  
- faulty actuators (e.g., wheels slippage on the robot may cause actual travelled distance to be less than calculated distance, which could then affect its estimation of environment variables),  
- fuzzy and dynamic environment (e.g., fog or surrounded by moving objects), and  
- the robot's own actions (e.g., movements, or interaction with the environment)

To incorporate these uncertainty, one approach is to model the state variables probabilistically. The probability distribution of the state variables conditioned on historical states and measurements is known as *belief*. What we want is to describe the probability distribution of the state variables given measurements and controls (note: *control* is the term used to mean all the robot's actions).

### Bayes-filter  
Before we venture further, we need to define the symbols we will be using.  
Let,

$$
\begin{aligned}
x_{t} &\quad \text{ be the state variables} \\
z_{t} &\quad \text{ be the measurements obtained from sensors} \\
u_{t} &\quad \text{ be the control/action variables the robot takes} \\
&\quad \text{(e.g., steering at an angle for some duration)} \\
u_{1:t} &\quad \text{ be the set of controls from time 1 to t}
\end{aligned}
$$

We can then state our motivation as “how do we evolve an initial boundary condition of \\( p(x_0) \\) to \\( p(x_t \mid z_{1:t}, u_{1:t}) \\).”  
Going from one time period to another, we assume the robot will execute two operations  
1) perform control using its actuators (note: we consider doing nothing as a valid control operation),  
2) measure its environment.  

Let’s illustrate these two operations from *t = 0* to *2*.  
At *t = 0*,

\\[ \displaystyle p(x_0) \\]

With whatever knowledge that we have, we form an opinion about the environment by specifying \\( p(x_0) \\), which could simply be a uniform distribution if we have zero knowledge.  

At *t = 1*, after the robot performs a control action, our belief becomes  

\\[
\displaystyle
p(x_1 \mid u_1) = \int_{x_0} p(x_1 \mid x_0, u_1)\,p(x_0)\,\mathrm{d}x_0 \qquad (1)
\\]

Then the robot performs a measurement on its environment. Its belief will then be updated by  

\\[
\displaystyle
p(x_1 \mid u_1, z_1) = \frac{p(z_1 \mid x_1, u_1)\,p(x_1 \mid u_1)}{p(z_1 \mid u_1)} \qquad (2)
\\]

At *t = 2*, with the next control action performed,  

\\[
\displaystyle
p(x_2 \mid z_1, u_{1:2}) = \int_{x_1} p(x_2 \mid x_1, z_1, u_{1:2})\,p(x_1 \mid z_1, u_1)\,\mathrm{d}x_1 \qquad (3)
\\]

With the next measurement update performed  

\\[
\displaystyle
p(x_2 \mid u_{1:2}, z_{1:2}) = \frac{p(z_2 \mid x_2, u_{1:2}, z_1)\,p(x_2 \mid u_{1:2}, z_1)}{p(z_2 \mid u_{1:2}, z_1)} \qquad (4)
\\]

At this point, we observe that equations (2) and (4) are similar except for the time subscript; the same holds for equations (1) and (3). This suggests we can formulate a recursive algorithm to evolve the robot’s belief forward in time.  

We introduce more terms for convenience.  
Let,

$$
\displaystyle
\begin{array}{rcl}
\overline{bel}(x_t) &=& p(x_t \mid z_{1:t-1}, u_{1:t}), \\\\
bel(x_t) &=& p(x_t \mid z_{1:t}, u_{1:t}), \\\\
\eta &=& \text{normalizing constant such that the probability integrates to 1.}
\end{array}
$$

Our posterior distribution becomes  

\\[
\displaystyle
bel(x_t) \;=\; \eta\,p(z_t \mid x_t, u_{1:t}, z_{1:t-1})\,\overline{bel}(x_t) \qquad (5)
\\]

From (5), the distributions we need are:

* initial probability of environment, \\( p(x_0) \\)  
* transition probability, \\( p(x_t \mid x_{t-1}, u_{1:t}) \\)  
* measurement probability, \\( p(z_t \mid x_t, u_{1:t}, z_{1:t-1}) \\)

As long as we have the transition function and measurement function, we could implement an algorithm that evolves our robot's state probability distribution over time. However, our specification requires us to keep track of all historical controls and measurements (as can be seen from the subscript \\( \_{1:t} \\) of (5)). This is rather impractical as our machines would quickly run out of memory. In fact, it generally makes sense that future prediction would depend lesser on historical information the farther they are back in time.

One way to capture this sentiment formally, is to state the following assumption,

\\[
\displaystyle
p(x_t \mid x_{t-1}, x_{t-2}, \dots, x_1) \;=\; p(x_t \mid x_{t-1}, x_{t-2}, \dots, x_{t-m}) \quad \text{for } t > m.
\\]

Under this assumption, we can now say that the prediction of the current state depends only on the past *m* states. A process that obeys this property is also known as a Markov chain of order *m*. For our purpose, we will assume that our process is a 1st order Markov chain. We can then restate our posterior belief in a simplified recursive form as below,


$$
\displaystyle
\begin{array}{rl}
bel(x_t) &= p(x_t \mid z_{1:t}, u_{1:t}) \\\\
        &= \frac{p(z_t \mid x_t, z_{1:t-1}, u_{1:t}) \, p(x_t \mid z_{1:t-1}, u_{1:t})}{p(z_t \mid z_{1:t-1}, u_{1:t})} \\\\
        &= \eta \, p(z_t \mid x_t, z_{1:t-1}, u_{1:t}) \, p(x_t \mid z_{1:t-1}, u_{1:t}) \\\\
        &= \eta \, p(z_t \mid x_t) \int_{x_{t-1}} p(x_t \mid x_{t-1}, z_{1:t-1}, u_{1:t}) \, p(x_{t-1} \mid z_{1:t-1}, u_{1:t}) \, dx_{t-1} \\\\
        &= \eta \, p(z_t \mid x_t) \int_{x_{t-1}} p(x_t \mid x_{t-1}, u_t) \, p(x_{t-1} \mid z_{1:t-1}, u_{1:t-1}) \, dx_{t-1} \\\\
        &= \eta \, p(z_t \mid x_t) \int_{x_{t-1}} p(x_t \mid x_{t-1}, u_t) \, bel(x_{t-1}) \, dx_{t-1} \\\\
        &= \eta \, p(z_t \mid x_t) \, \overline{bel}(x_t)
\end{array}
$$


Thus a Bayes-filter needs only:

* initial state distribution  
* measurement distribution  
* state-transition distribution  

Pseudocode for the filter:

$$
\displaystyle
\begin{array}{l}
\textbf{Input: } \bigl(bel(x_{t-1}),\,u_t,\,z_t\bigr) \text{ for all } x_t \\\\
\overline{bel}(x_t) \;=\; \int_{x_{t-1}} p(x_t \mid x_{t-1},u_t)\,bel(x_{t-1}) \\\\
bel(x_t) \;=\; \eta\,p(z_t \mid x_t)\,\overline{bel}(x_t) \\\\
\textbf{return } bel(x_t)
\end{array}
$$

References  
Thrun, Sebastian, et al. *Probabilistic Robotics*. MIT Press, 2010.
