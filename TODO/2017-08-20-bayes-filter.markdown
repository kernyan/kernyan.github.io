---
author: kernyan9
comments: true
date: 2017-08-20 00:55:27+00:00
layout: post
link: http://kernyan.com/2017/08/20/bayes-filter/
slug: bayes-filter
title: Derivation of Bayes-Filter algorithm
wordpress_id: 180
categories:
- Autonomous Vehicles
---

** Why is Bayes-filter useful? **
For many autonomous robotics system, the action to take at each time period depends on its environment. For instance, a roomba (robotic vacuum cleaner) needs to know the position it is in a room before it can decide which way to move.
The more a robot knows about the environment, the better the decision it can make. However measuring the environment is often uncertain, as it can be affected by among others,



 	
  * noise in measurement data (e.g., sensors such as LIDAR/RADAR having limited accuracy),
 	
  * faulty sensors,
 	
  * faulty actuators (e.g., wheels slippage on the robot may cause actual travelled distance to be less than calculated distance, which could then affect its estimation of environment variables),
 	
  * fuzzy and dynamic environment (e.g., fog or surrounded by moving objects), and
 	
  * the robot's own actions (e.g., movements, or interaction with the environment)
To incorporate these uncertainty, one approach is to model the state variables probabilistically. The probability distribution of the state variables conditioned on historical states and measurements is known as belief. What we want is to describe the probability distribution of the state variables given measurements and controls (note: control is the term used to mean all the robot's action).

** Bayes-filter **
Before we venture further, we need to define the symbols we will be using.
Let,


$latex \displaystyle  \begin{array}{rcl}  x_{t} && \text{ be the state variables} \\ z_{t} && \text{ be the measurements obtained from sensors} \\ u_{t} && \text{ be the control/action variables the robot takes} \\ && \text{(e.g., steering at an angle for some duration)} \\ u_{1:t} && \text{ be the set of controls from time 1 to t} \end{array} &fg=000000$



We can then state our motivation as "how do we evolve an initial boundary condition of $latex {p(x_0)}&fg=000000$ to $latex {p(x_t|z_{1:t}, u_{1:t})}&fg=000000$."
Going from one time period to another, we assume the robot will execute two operations 1) perform control using its actuators (note: we consider doing nothing as a valid control operation), 2) measure its environment.
Let's illustrate these two operations from t = 0 to 2,
At t = 0,


$latex \displaystyle  p(x_0) &fg=000000$



With whatever knowledge that we have, we form an opinion about the environment by specifying $latex {p(x_0)}&fg=000000$, which could simply be a uniform distribution if we have zero knowledge.
At t = 1,
After the robot performs a control action, our belief becomes



$latex \displaystyle  p(x_1|u_1) = \int_{x_0}  p(x_1 | x_0, u_1) p(x_0) \ \mathrm{d}x_0  \ \ \ \ \ (1)&fg=000000$




Then the robot performs a measurement on its environment. Its belief will then be updated by



$latex \displaystyle  p(x_1|u_1, z_1) = \frac{p(z_1 | x_1, u_1)p(x_1|u_1)}{p(z_1|u_1)}  \ \ \ \ \ (2)&fg=000000$




At t = 2,
With the next control action performed,



$latex \displaystyle  p(x_2|z_1, u_{1:2}) = \int_{x_1}  p(x_2 | x_1, z_1, u_{1:2}) p(x_1|z_1, u_1) \ \mathrm{d}x_1  \ \ \ \ \ (3)&fg=000000$




With the next measurement update performed



$latex \displaystyle  p(x_2|u_{1:2}, z_{1:2}) = \frac{p(z_2 | x_2, u_{1:2}, z_1)p(x_2|u_{1:2}, z_1)} {p(z_2|u_{1:2}, z_1)}  \ \ \ \ \ (4)&fg=000000$




At this point, we observe that equation (2) and (4) are similar except for the time subscript. Such is also true for equation (1) and (3). This leads us to suspect that we could formulate a recursive algorithm to evolve forward our robot's belief.
We shall introduce more terms for convenience.
Let,


$latex \displaystyle  \begin{array}{rcl}  \overline{bel}(x_t) &=& p(x_t|z_{1:t-1}, u_{1:t}) \text{,} \\ bel(x_t) &=& p(x_t|z_{1:t}, u_{1:t}) \text{, and} \\ \eta &=& \text{normalizing constant such that probability expression integrates to 1} \end{array} &fg=000000$



Our posterior distribution becomes



$latex \displaystyle  bel(x_{t}) = \eta \ p(z_{t} | x_{t}, u_{1:t}, z_{1:t-1}) \ \overline{bel}(x_{t})  \ \ \ \ \ (5)&fg=000000$




Inspecting (5), we list the distributions we need



 	
  1. initial probability of environment, $latex {p(x_0)}&fg=000000$,
 	
  2. transition probability, $latex {p(x_t|x_{t-1}, u_{1:t})}&fg=000000$,
 	
  3. measurement probability, $latex {p(z_t|x_t, u_{1:t}, z_{1:t-1})}&fg=000000$
As long as we have the transition function and measurement function, we could implement an algorithm that evolves our robot's state probability distribution over time. However, our specification requires us to keep track of all historical controls and measurements (as can be seen from the subscript $latex {{1:t} }&fg=000000$ of (5). This is rather impractical as our machines would quickly run out of memory. In fact, it generally makes sense that future prediction would depend lesser on historical information the farther they are back in time. One way to capture this sentiment formally, is to state the following assumption,


$latex \displaystyle  p(x_t|x_{t-1}, x_{t-2},\ldots,x_1) = p(x_t|x_{t-1}, x_{t-2},\ldots,x_{t-m}) \qquad \text{ for } t > m &fg=000000$



With this assumption, we can now say that the prediction of the current state depends only on the past $latex {m}&fg=000000$ states. A process that obeys this property is also known as a Markov chain of order $latex {m}&fg=000000$. For our purpose, we will assume that our process is a 1st order Markov chain. We can then restate our posterior belief in a simplified recursive form as below,


$latex \displaystyle  \begin{array}{rcl}  bel(x_t) &=& p(x_t|z_{1:t}, u_{1:t}) \\ &=& \frac{p(z_t|x_t, z_{1:t-1}, u_{1:t})\ p(x_t|z_{1:t-1}, u_{1:t})} {p(z_t|z_{1:t-1}, u_{1:t})} \\ &=& \eta \ p(z_t|x_t, z_{1:t-1}, u_{1:t})\ p(x_t|z_{1:t-1}, u_{1:t}) \\ &=& \eta \ p(z_t|x_t)\int_{x_{t-1}}\ p(x_t| x_{t-1}, z_{1:t-1}, u_{1:t}) \ p(x_{t-1}|z_{1:t-1}, u_{1:t}) \ \mathrm{d}x_{t-1}\\ &=& \eta \ p(z_t|x_t)\int_{x_{t-1}} p(x_t| x_{t-1}, u_t) \ p(x_{t-1}|z_{1:t-1}, u_{1:t-1}) \ \mathrm{d}x_{t-1}\\ &=& \eta \ p(z_t|x_t)\int_{x_{t-1}}\ p(x_t| x_{t-1}, u_t) bel(x_{t-1}) \ \mathrm{d}x_{t-1}\\ &=& \eta \ p(z_t|x_t)\ \overline{bel}(x_{t-1}) \end{array} &fg=000000$



The derivation above shows that we can now state a recursive state distribution with



 	
  * initial state distribution
 	
  * measurement distribution
 	
  * state transition distribution
This is also known as the Bayes-filter. To implement the filter, one can use the pseudocode presented below


$latex \displaystyle  \begin{array}{rcl}  && \mkern-95mu \text{Input } (bel(x_{t-1}), u_t, z_t) \text{ for all } x_t \\ \overline{bel}(x_t) &=& \int_{x_{t-1}} p(x_t|x_{t-1},u_t)\ bel(x_{t-1}) \\ \indent bel(x_t) &=& \eta \ p(z_t|x_t)\ \overline{bel}(x_t) \\ && \mkern-95mu \text{return } bel(x_t) \end{array} &fg=000000$




** References **
 Thrun, Sebastian, et al. Probabilistic Robotics. MIT Press, 2010.
