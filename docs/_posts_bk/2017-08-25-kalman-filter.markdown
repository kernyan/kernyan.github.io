---
author: kernyan9
comments: true
date: 2017-08-25 04:36:54+00:00
layout: post
link: http://kernyan.com/2017/08/25/kalman-filter/
slug: kalman-filter
title: Full derivation of Kalman-Filter algorithm
wordpress_id: 676
categories:
- Autonomous Vehicles
---












** Specifying the Kalman-Filter algorithm **









The Bayes filter elaborated in the previous post gave us the basis of evolving a posterior state distribution across time. However, there are several issues we have to sort out before we can implement a Bayes filter for practical usage,







  * the exact distributions of initial state $latex {p(x_0)}&fg=000000$, measurement function $latex {p(z_t|x_t)}&fg=000000$, and transition function $latex {p(x_t|x_{t-1},u_t)}&fg=000000$ are often unknown, and 
  * even if they are known, the evaluation of product of distributions (e.g., transition density function * $latex {\overline{bel}(x_t)}&fg=000000$), and integration of distributions (e.g., transition function over range of prior state) generally requires numerical methods as there are no closed-form solutions. These increases computational requirement and could cause our robot to be inappropriate for real-time usages. 






One way to address the issues above is to make some assumptions that allows our Bayes algorithm to be more amenable. Let's assume that our state evolves according to a linear Gaussian process. This allows us to reformulate our functions as follows,





$latex \displaystyle \begin{array}{rcl}  \text{Transition function, } x_t & = & A_t x_{t-1} + B_t u_t + \epsilon_t  \\ \text{Measurement function, } z_t & = & C_t x_t + \delta_t \end{array}\ \ \ \ \ (1)&fg=000000$






where,





$latex \displaystyle  \begin{array}{rcl}  \text{Transition error, } \epsilon_t & \sim & \mathcal{N}(0, R_t) \\ \text{Measurement error, } \delta_t & \sim & \mathcal{N}(0, Q_t) \\ \text{Initial state distribution, } x_0 & \sim & \mathcal{N}(\mu_0, \Sigma_0) \end{array} &fg=000000$






A Bayes-filter that follows Gaussian process is also known as a Kalman-filter. Our interest today is to show that in a Kalman-filter, the conditional state posterior distribution is also a Gaussion process. In other words, that the following holds,





$latex \displaystyle  x_t|z_{1:t}, u_{1:t} \sim \mathcal{N}(\mu_t, \Sigma_t)  \ \ \ \ \ (2)&fg=000000$






Restating the Bayes algorithm under a Kalman-filter gives us,





$latex \displaystyle  \begin{array}{rcl}  && \mkern-60mu \text{Kalman-filter algorithm: Input} \left(\mu_{t-1}, \Sigma_{t-1}, u_t, z_t\right) \\ \bar{\mu_t} & = & A_t \mu_{t-1} + B_t u_t \\ \bar{\Sigma_t} & = & A_t \Sigma_{t-1}{A_t}^{T} + R_t \\ K & = & \bar{\Sigma_t} {C_t}^T {(Q_t + C_t \bar{\Sigma_t} {C_t}^T)}^{-1} \\ \Sigma_t & = & (I - K_t C_t)\bar{\Sigma_t} \\ \mu_t & = & \bar{\mu_t} + K(z_t - C_t \bar{\mu_t}) \\ && \mkern-60mu \text{return } \mu_t, \Sigma_t \end{array} &fg=000000$












** Derivation of Kalman-filter algorithm **









We shall now prove that the Kalman-filter algorithm results in the state posterior distribution (2) by induction. For this, we need to show that,





$latex \displaystyle  \begin{array}{rcl}  x_0 & \sim & \mathcal{N}(\mu_0, \Sigma_0) \qquad \text{and,} \\ x_{t-1}|z_{1:t-1}, u_{1:t-1} & \sim & \mathcal{N}(\mu_{t-1}, \Sigma_{t-1}) \end{array} &fg=000000$






implies, 





$latex \displaystyle  x_t|z_{1:t}, u_{1:t} \sim \mathcal{N}(\mu_t, \Sigma_t)  \ \ \ \ \ (3)&fg=000000$






The first point is true because our initial state distribution is assumed to be normal within the context of Kalman-filter. The second point will be shown as part of our derivation of the Kalman-filter algorithm.



Our goal is to express $latex {\mu_t }&fg=000000$ and $latex {\Sigma_t }&fg=000000$ as function of $latex {\mu_{t-1}, \Sigma_{t-1}, u_t, \text{ and } z_t }&fg=000000$. Recall from the previous post that,





$latex \displaystyle  \begin{array}{rcl}  bel(x_t) & = & p(x_t|z_{1:t}, u_{1:t}) \\ & = & \eta p(z_t|x_t) \ \overline{bel}(x_{t-1}) \\ & = & \eta p(z_t|x_t) \int_{x_{t-1}} p(x_t| x_{t-1}, u_t) \ p(x_{t-1}|z_{1:t-1}, u_{1:t-1}) \ \mathrm{d}x_{t-1} \end{array} &fg=000000$






We first show that the integrand above evaluates to a normal distribution. From (1), we know that 

$latex \displaystyle  x_t | x_{t-1}, u_t \sim \mathcal{N}(A_t x_{t-1}+B_t u_t, A\Sigma_{t-1}A^{T}+R_t) &fg=000000$


 Now, we can express $latex {\overline{bel}(x_t) }&fg=000000$ as,





$latex \displaystyle \begin{array}{rcl}  \overline{bel}(x_t) & = & \int_{x_{t-1}}p(x_t|x_{t-1},u_t)\ p(x_{t-1}|z_{1:t-1},u_{1:t-1}) \mathrm{d}x_{t-1} \nonumber \\ & = & \eta \int_{x_{t-1}} \text{exp } \{ - \frac{1}{2} {(x_t - A_t x_{t-1} - B_t u_t)}^T {R_t}^{-1} (x_t - A_t x_{t-1} - B_t u_t) \} \nonumber \\ && \quad \qquad \text{exp }\{ - \frac{1}{2} {(x_{t-1} - \mu_{t-1})}^T {\Sigma_{t-1}}^{-1} (x_{t-1} - \mu_{t-1}) \} \mathrm{d}x_{t-1}  \end{array}\ \ \ \ \ (4)&fg=000000$






$latex {\eta }&fg=000000$ is a normalizing factor that collects all constants not expressed, such that our probability density function still integrates to 1.



We want to rearrange the terms in the exponential in (4) in such a way that allows us to integrate over $latex {x_{t-1}}&fg=000000$. (For convenience, let's denote the terms in the exponential in (4) as $latex {-L_t}&fg=000000$. Our strategy is to attempt to,







  1. decompose $latex {L_t }&fg=000000$ into two functions, one with the terms $latex {x_{t-1}}&fg=000000$, and one without, which we shall label as $latex {L_t(x_{t-1}, x_t) \text{ and } L_t(x_t)}&fg=000000$ respectively, and 
  2. identify a form for $latex {L_t(x_{t-1}, x_t)}&fg=000000$ such that the integral over $latex {x_{t-1}}&fg=000000$ evaluates to a constant, which we can then subsume into the normalizing constant $latex {\eta }&fg=000000$. 






To do the above, we observe that the function $latex {L_t }&fg=000000$ is,







  1. quadratic in $latex {x_{t-1} }&fg=000000$, and 
  2. exponential over the quadratic 






This suggests that if we can collect the terms with $latex {x_{t-1} }&fg=000000$ and rearrange it such that it matches a normal probability density, then the integral would evaluate to 1 (or a constant which then gets collected together with $latex {\eta }&fg=000000$).



We can construct a normal distribution from an exponential quadratic (see Appendix). By applying first and second order differentiation to $latex {L_t(x_{t-1}, x_t) }&fg=000000$, we get,





$latex \displaystyle \begin{array}{rcl}  \frac{\delta L_t(x_{t-1})}{\delta x_{t-1}} & = & - {A_t}^T {R_t}^{-1} (x_t-A_t x_{t-1} -B_t u_t) + {\Sigma_{t-1}}^{-1}(x_{t-1}-\mu_{t-1}) \\ \frac{\delta^2 L_t(x_{t-1})}{\delta {x_{t-1}}^2} & = & {A_t}^T {R_t}^{-1} A_t + {\Sigma_{t-1}}^{-1} \\ &=:& {\Psi_t}^{-1}  \end{array}\ \ \ \ \ (5)&fg=000000$






Setting the first derivative to 0 and solving for $latex {x_{t-1} }&fg=000000$ gives us (after some algebraic manipulation),





$latex \displaystyle  x_{t-1} = \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}] \ \ \ \ \ (6)&fg=000000$






Thus using the $latex {x_{t-1} }&fg=000000$ we solved for, and $latex {\Psi_t }&fg=000000$ as the mean and variance of a Gaussian pdf, we have





$latex \displaystyle  \begin{array}{rcl}  L_t(x_{t-1},x_t) & = & \frac{1}{2} {(x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}])}^T {\Psi_t}^{-1} \\ && \negmedspace (x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}]) \end{array} &fg=000000$






With this, let us find $latex {L_t(x_t) }&fg=000000$ by subtracting $latex {L_t(x_{t-1}, x_t) }&fg=000000$ from $latex {L_t }&fg=000000$





$latex \displaystyle \begin{array}{rcl}  L_t(x_t) & = & L_t - L_t(x_{t-1}, x_t) \\ & = & \frac{1}{2} {(x_t - A_t x_{t-1} - B_t u_t)}^T {R_t}^{-1} (x_t - A_t x_{t-1} - B_t u_t)\\ && \negmedspace + \ \frac{1}{2}{(x_{t-1}-\mu_{t-1})}^T{\Sigma_{t-1}}^{-1}(x_{t-1}-\mu_{t-1})\\ && \negmedspace - \ \frac{1}{2} { ( x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}] ) }^T {\Psi_t}^{-1} \\ && \negmedspace ( x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}] )  \end{array}\ \ \ \ \ (7)&fg=000000$






Substituting $latex {\Psi_t }&fg=000000$ from (5) into (7), and after more algebraic manipulation, it turns out that all terms with $latex {x_{t-1} }&fg=000000$ cancels out and we are left with





$latex \displaystyle  \begin{array}{rcl}  L_t(x_t) & = & \frac{1}{2}{(x_t - B_t u_t)}^T {R_t}^{-1}(x_t - B_t u_t) \\ && \negmedspace +\ \frac{1}{2}{\mu_{t-1}}^T {\Sigma_{t-1}}^{-1}\mu_{t-1} \\ && \negmedspace -\ \frac{1}{2}{ [{A_t}^T{R_t}^{-1}(x_t-B_t u_t) +{\Sigma_{t-1}}^{-1}\mu_{t-1} ]}^T {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1}) }^{-1} \\ && \negmedspace [{A_t}^T{R_t}^{-1}(x_t-B_t u_t) +{\Sigma_{t-1}}^{-1}\mu_{t-1}] \end{array} &fg=000000$






The forms of $latex {L_t(x_{t-1}, x_t)}&fg=000000$ and $latex {L_t(x_t)}&fg=000000$ that we have now tells us two things, that







  1. $latex {L_t(x_{t-1}, x_t) }&fg=000000$ indeed has a Gaussian distribution form which would result in $latex {\int \textnormal{exp } \{-L_t(x_{t-1}, x_t)\}\ \mathrm{d} x_{t-1} = \eta }&fg=000000$, and 
  2. $latex {L_t(x_t)}&fg=000000$ indeed does not contain any terms with $latex {x_{t-1} }&fg=000000$ 






With the decomposition of $latex {L_t }&fg=000000$ we have now, we can now continue developing equation (4).





$latex \displaystyle \begin{array}{rcl}  \overline{bel}(x_t) & = & \int_{x_{t-1}}p(x_t|x_{t-1}, u_t)p(x_{t-1}|z_{1:t-1}, u_{1:t-1}) \ \mathrm{d}x_{t-1} \\ & = & \eta \int_{x_{t-1}} \text{exp }\{ - \frac{1}{2} {(x_t - A_t x_{t-1} - B_t u_t)}^T {R_t}^{-1} (x_t - A_t x_{t-1} - B_t u_t)\} \\ && \negmedspace \text{exp } \{ -\frac{1}{2} {(x_{t-1} - \mu_{t-1})}^T {\Sigma_{t-1}}^{-1} (x_{t-1} - \mu_{t-1})\} \ \mathrm{d}x_{t-1} \\ & = & \eta \ \text{exp } \{-L_t(x_t)\} \int_{x_{t-1}} \text{exp } \{ -L_t(x_{t-1}, x_t)\} \ \mathrm{d}x_{t-1} \\ & = & \eta \ \text{exp } \{-L_t(x_t)\}  \end{array}\ \ \ \ \ (8)&fg=000000$






We notice from (7) that $latex {L_t(x_t)}&fg=000000$ is also exponential quadratic in the terms $latex {x_t}&fg=000000$, as such, we again apply the first and second order differentiation technique to construct a normal distribution (see Appendix). We get,





$latex \displaystyle \begin{array}{rcl}  \frac{\delta L_t(x_t)}{\delta {x_t}} & = & {R_t}^{-1}(x_t-B_t u_t) - {R_t}^{-1} A_t {({A_t}^T {R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1} \\ && \negmedspace [{A_t}^T {R_t}^{-1}(x_t-B_t u_t)+{\Sigma_{t-1}}^{-1}\mu_{t-1}] \\ \frac{\delta^2 L_t(x_t)}{\delta {x_t}^2} & = & {R_t}^{-1} - {R_t}^{-1}A_t{({A_t}^T{R_t}^{-1}A_t+{\Sigma_{t-1}}^{-1})}^{-1}{A_t}^T{R_t}^{-1}  \end{array}\ \ \ \ \ (9)&fg=000000$






Using inversion lemma (see Appendix), we can rewrite (9) as,





$latex \displaystyle \begin{array}{rcl}  \frac{\delta^2 L_t(x_t)}{\delta {x_t}^2} & = & {R_t}^{-1} - {R_t}^{-1}A_t{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{A_t}^T{R_t}^{-1} \\ & = & {(R_t+A_t\Sigma_{t-1}{A_t}^T)}^{-1} \\ & =: & {{\bar{\Sigma}}_t}^{-1}  \end{array}\ \ \ \ \ (10)&fg=000000$






The intuitive interpretation of (10) is that the variance of our normal distribution, $latex {{\bar{\Sigma}}_t}&fg=000000$ is the variance of the transition process (1), which is the sum of







  * the error in transition process (1), $latex {R_t }&fg=000000$, and 
  * the variance of the prior period's belief, $latex {\overline{bel}(x_{t-1})}&fg=000000$ evolved through the linear process $latex {A_t }&fg=000000$ 






Next, we solve for $latex {x_t }&fg=000000$ in (9),





$latex \displaystyle  \begin{array}{rcl}  0 &=& {R_t}^{-1}(x_t-B_t u_t) \\ &-& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1}\mu_{t-1}] \\ &\iff& {R_t}^{-1}(x_t-B_t u_t) \\ &=& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1}\mu_{t-1}] \\ &\iff& [{R_t}^{-1} - {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{A_t}^T {R_t}^{-1}](x_t-B_t u_t) \\ &=& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\ &\iff& {(R_t+A_t\Sigma_{t-1}{A_t}^T)}^{-1}(x_t-B_t u_t) \\ &=& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \text{ by inversion lemma} \\ &\iff& (x_t-B_t u_t) \\ &=& (R_t+A_t\Sigma_{t-1}{A_t}^T){R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + (R_t+A_t\Sigma_{t-1}{A_t}^T){R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + (I+A_t\Sigma_{t-1}{A_t}^T{R_t}^{-1}) A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + (I+A_t\Sigma_{t-1}{A_t}^T{R_t}^{-1}) A_t [{\Sigma_{t-1}}{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}]^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + (A_t+A_t\Sigma_{t-1}{A_t}^T{R_t}^{-1}A_t) [{\Sigma_{t-1}}{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}]^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + A_t(I+\Sigma_{t-1}{A_t}^T{R_t}^{-1}A_t) [{\Sigma_{t-1}}{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}]^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + A_t(I+\Sigma_{t-1}{A_t}^T{R_t}^{-1}A_t) {(I+{\Sigma_{t-1}}{A_t}^T{R_t}^{-1}A_t)}^{-1}\mu_{t-1} \\ &\iff& x_t = B_t u_t + A_t\mu_{t-1} \\ &\iff& x_t = A_t\mu_{t-1} + B_t u_t \end{array} &fg=000000$






Thus, we obtain the mean of the normal distribution as,





$latex \displaystyle  x_t = A_t\mu_{t-1} + B_t u_t  \ \ \ \ \ (11)&fg=000000$






With (11) as the mean, and (10) as the variance parameter of the normal distribution of (8), we can now say that,





$latex \displaystyle  \overline{bel}(x_t) \sim \mathcal{N}(A_t\mu_{t-1} + B_t u_t, A_t\Sigma_{t-1}{A_t}^T + R_t) =: \mathcal{N}(\bar{\mu}_t, \bar{\Sigma}_t)  \ \ \ \ \ (12)&fg=000000$






Now the last remaining step we have to do is to combine the transition distribution with the measurement distribution,





$latex \displaystyle  bel(x_t) \\ = p(x_t|z_{1:t}, u_{1:t}) \\ = \eta \ p(z_t|x_t)\ \overline{bel}(x_{t-1})  \ \ \ \ \ (13)&fg=000000$






Since we now know that both $latex {p(z_t|x_t)}&fg=000000$ and $latex {\overline{bel}(x_{t-1}) }&fg=000000$ are normal distribution with parameters (1) and (12) respectively, we can write (13) as,





$latex \displaystyle \begin{array}{rcl}  bel(x_t) &=& p(x_t|z_{1:t}, u_{1:t}) \\ &=& \eta \ p(z_t|x_t)\ \overline{bel}(x_{t-1}) \\ &=& \eta \text{ exp } \{-\frac{1}{2}{(z_t - C_t x_t)}^T{Q_t}^{-1}(z_t - C_t x_t) \} \\ && \negmedspace \text{ exp } \{-\frac{1}{2}{ (x_t - \bar{\mu}_t)}^T{\bar{\Sigma}_t}^{-1}(x_t - \bar{\mu}_t) \}  \end{array}\ \ \ \ \ (14)&fg=000000$






Let us collect the exponential terms in (14) as $latex {J_t}&fg=000000$, such that, 

$latex \displaystyle  bel(x_t) = \eta \text{ exp } \{-J_t\} &fg=000000$






Again we notice that (14) is exponential quadratic in $latex {x_t}&fg=000000$, which means we can apply first and second order differentiation technique to $latex {J_t }&fg=000000$ to determine the mean and variance parameters of its distribution, which we also know at this point is Gaussian,





$latex \displaystyle \begin{array}{rcl}  \frac{\delta J_t}{\delta {x_t}} &=& -{C_t}^T{Q_t}^{-1}(z_t - C_t x_t) + {\bar{\Sigma}_t}^{-1}(x_t-\bar{\mu}_t) \\ \frac{\delta^2 J_t}{\delta {x_t}^2} &=& {C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1} \\ &=:& {\Sigma_t}^{-1}  \end{array}\ \ \ \ \ (15)&fg=000000$






Setting (15) to zero, and substituing $latex {\mu_t }&fg=000000$ for $latex {x_t }&fg=000000$,





$latex \displaystyle \begin{array}{rcl}  0 &=& -{C_t}^T{Q_t}^{-1}(z_t - C_t x_t) + {\bar{\Sigma}_t}^{-1}(x_t-\bar{\mu}_t) \\ 0 &=& -{C_t}^T{Q_t}^{-1}(z_t - C_t \mu_t) + {\bar{\Sigma}_t}^{-1}(\mu_t-\bar{\mu}_t) \\ &\iff& {C_t}^T{Q_t}^{-1}(z_t - C_t \mu_t) = {\bar{\Sigma}_t}^{-1}(\mu_t-\bar{\mu}_t) \\ &\iff& {C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t = ({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})\mu_t \\ &\iff& \mu_t = {({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})}^{-1}({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t)  \end{array}\ \ \ \ \ (16)&fg=000000$






We now have a complete specification of $latex {bel(x_t) }&fg=000000$ as a normal distribution with mean, and variance as specified in (16) and (15), which proved (3). Our proof by induction of (2) is now complete.









** Kalman Gain **









We could have used (16) and (15) in our Kalman-filter algorithm as below, and call it a day.





$latex \displaystyle \begin{array}{rcl}  \mu_t &=& {({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})}^{-1}({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t) \\ \Sigma_t &=& ({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})^{-1}  \end{array}\ \ \ \ \ (17)&fg=000000$






However another representation that in more commonly used exists. It also is more efficient by allowing us to avoid having to evaluate double inverses. By introducing a Kalman gain term, we can express (17) as,





$latex \displaystyle \begin{array}{rcl}  \mu_t &=& \bar{\mu}_t+K(z_t-C_t \bar{\mu}_t) \\ \Sigma_t &=& (I - K_t C_t)\bar{\Sigma}_t  \end{array}\ \ \ \ \ (18)&fg=000000$






To go from the variance formulation from (17) to (18), we write





$latex \displaystyle \begin{array}{rcl}  \Sigma_t \ &=& {(C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t)}^{-1} \\ &=& \bar{\Sigma}_t - \bar{\Sigma}_t {C_t}^T (Q_t + C_t \bar{\Sigma}_t {C_t}^T)^{-1}C_t \bar{\Sigma}_t \\ &=& (I - \bar{\Sigma}_t {C_t}^T [Q_t + C_t \bar{\Sigma}_t {C_t}^T]^{-1}C_t )\bar{\Sigma}_t \\ &=& (I - K_t C_t )\overline{\Sigma}_t  \end{array}\ \ \ \ \ (19)&fg=000000$






Where we define the Kalman gain $latex {K_t }&fg=000000$ as





$latex \displaystyle  K_t = \bar{\Sigma}_t {C_t}^T [Q_t + C_t \bar{\Sigma}_t {C_t}^T]^{-1}  \ \ \ \ \ (20)&fg=000000$






Next, we show how we can get the mean formulation from (17) to (18),





$latex \displaystyle \begin{array}{rcl}  \mu_t &=& {({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})}^{-1}({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t) \\ &=& (I-K_t C_t)\bar{\Sigma}_t ({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t) \\ &=& \bar{\Sigma}_t {C_t}^T{Q_t}^{-1}z_t - K_t C_t \bar{\Sigma}_t {C_t}^T{Q_t}^{-1}z_t + (I-K_t C_t)\bar{\mu}_t \\ &=& K_t (Q_t + C_t \bar{\Sigma}_t {C_t}^T) {Q_t}^{-1}z_t - K_t C_t \bar{\Sigma}_t {C_t}^T{Q_t}^{-1}z_t + (I-K_t C_t)\bar{\mu}_t \\ &=& K_t (I + C_t \bar{\Sigma}_t {C_t}^T {Q_t}^{-1} - C_t \bar{\Sigma}_t {C_t}^T{Q_t}^{-1})z_t + (I-K_t C_t)\bar{\mu}_t \\ &=& K_t z_t + (I-K_t C_t)\bar{\mu}_t \\ &=& \bar{\mu}_t + K_t (z_t - C_t \bar{\mu}_t)  \end{array}\ \ \ \ \ (21)&fg=000000$






Finally, combining (12), (19), (20), and (21), we obtain the Kalman-filter algorithm.









** Appendix **















** Constructing Normal distribution **









For a normal probability density function $latex {\mathcal{N}(\mu, \Sigma) }&fg=000000$, it's pdf has the form





$latex \displaystyle  f(x) = \frac{1}{\sqrt{2\pi\Sigma}} \text{ exp }\{- \frac{1}{2} {(x - \mu)}^T {\Sigma}^{-1} (x - \mu)\} &fg=000000$






Let the expression in the exponential be denoted as $latex {L(x)}&fg=000000$, then the parameters of the normal distribution can be obtained as,







  * $latex {\mu = x \textnormal{ solved for with } \frac{\delta L(x)}{\delta x}}&fg=000000$ set to 0, and 
  * $latex {\Sigma^{-1} = \frac{\delta^2 L(x)}{\delta x^2} }&fg=000000$ 












** Inversion Lemma **









The inversion lemma (or also known as Woodbury matrix identity) states that for invertible matrices $latex {A, U, C, V }&fg=000000$ of the correct size, the following holds





$latex \displaystyle  (A+UCV)^{-1} = A^{-1} - A^{-1}U(C^{-1}+VA^{-1}U)^{-1}VA^{-1} &fg=000000$












** References **









 Thrun, Sebastian, et al. Probabilistic Robotics. MIT Press, 2010.




