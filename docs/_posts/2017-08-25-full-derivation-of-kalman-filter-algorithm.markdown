---
layout: post
title:  "Full derivation of Kalman-Filter algorithm"
date:   2017-08-25 08:00:00 -0000
categories:
- Robotics
mathjax: true
---


# Table of Contents
1. [Specifying the Kalman-filter algorithm](#specifying-the-kalman-filter-algorithm)
2. [Derivation of Kalman-filter algorithm](#derivation-of-kalman-filter-algorithm)
3. [Appendix](#appendix)

### Specifying the Kalman-filter algorithm

The Bayes filter elaborated in the previous post gave us the basis of evolving a posterior state distribution across time. However, there are several issues we have to sort out before we can implement a Bayes filter for practical usage,

- the exact distributions of initial state \\( {p(x_0)} \\), measurement function \\( {p(z_t \mid x_t)} \\), and transition function \\( {p(x_t \mid x_{t-1},u_t)} \\) are often unknown, and
- even if they are known, the evaluation of product of distributions (e.g., transition density function * \\( {\overline{bel}(x_t)} \\)), and integration of distributions (e.g., transition function over range of prior state) generally requires numerical methods as there are no closed-form solutions. These increases computational requirement and could cause our robot to be inappropriate for real-time usages. 

One way to address the issues above is to make some assumptions that allows our Bayes algorithm to be more amenable. Let's assume that our state evolves according to a linear Gaussian process. This allows us to reformulate our functions as follows,

$$
\displaystyle
\begin{aligned}
\text{Transition function, } x_t &= A_t x_{t-1} + B_t u_t + \epsilon_t \quad \\
\text{Measurement function, } z_t &= C_t x_t + \delta_t 
\end{aligned}
\tag{1}
$$

where,

$$
\displaystyle
\begin{aligned}
\text{Transition error, } \epsilon_t & \sim \mathcal{N}(0, R_t) \\ 
\text{Measurement error, } \delta_t & \sim  \mathcal{N}(0, Q_t) \\
\text{Initial state distribution, } x_0 & \sim \mathcal{N}(\mu_0, \Sigma_0) 
\end{aligned}
$$

A Bayes-filter that follows Gaussian process is also known as a Kalman-filter. Our interest today is to show that in a Kalman-filter, the conditional state posterior distribution is also a Gaussion process. In other words, that the following holds,

$$
\displaystyle x_t \mid z_{1:t}, u_{1:t} \sim \mathcal{N}(\mu_t, \Sigma_t) \tag{2}
$$

Restating the Bayes algorithm under a Kalman-filter gives us,

$$
\displaystyle
\begin{aligned}
&\text{Kalman-filter algorithm: Input} \left(\mu_{t-1}, \Sigma_{t-1}, u_t, z_t\right) \\
\bar{\mu_t} &= A_t \mu_{t-1} + B_t u_t \\
\bar{\Sigma_t} &= A_t \Sigma_{t-1}{A_t}^{T} + R_t \\
K &= \bar{\Sigma_t} {C_t}^T {(Q_t + C_t \bar{\Sigma_t} {C_t}^T)}^{-1} \\
\Sigma_t &= (I - K_t C_t)\bar{\Sigma_t} \\
\mu_t &= \bar{\mu_t} + K(z_t - C_t \bar{\mu_t}) \\
& \text{return } \mu_t, \Sigma_t
\end{aligned}
$$

### Derivation of Kalman-filter algorithm

We shall now prove that the Kalman-filter algorithm results in the state posterior distribution (2) by induction. For this, we need to show that,

$$
\displaystyle 
\begin{aligned}
x_0 &\sim \mathcal{N}(\mu_0, \Sigma_0) \qquad \text{and,} \\
x_{t-1} \mid z_{1:t-1}, u_{1:t-1} &\sim \mathcal{N}(\mu_{t-1}, \Sigma_{t-1})
\end{aligned}
$$

implies,

$$
\displaystyle x_t \mid z_{1:t}, u_{1:t} \sim \mathcal{N}(\mu_t, \Sigma_t) \tag{3}
$$

The first point is true because our initial state distribution is assumed to be normal within the context of Kalman-filter. The second point will be shown as part of our derivation of the Kalman-filter algorithm.

Our goal is to express \\( {\mu_t } \\) and \\( {\Sigma_t } \\) as function of \\( {\mu_{t-1}, \Sigma_{t-1}, u_t, \text{ and } z_t } \\). Recall from the previous post that,

$$
\displaystyle
\begin{aligned}
bel(x_t) &= p(x_t \mid z_{1:t}, u_{1:t}) \\
&= \eta p(z_t \mid x_t) \ \overline{bel}(x_{t-1}) \\
&= \eta p(z_t \mid x_t) \int_{x_{t-1}} p(x_t \mid x_{t-1}, u_t) \ p(x_{t-1} \mid z_{1:t-1}, u_{1:t-1}) \ \mathrm{d}x_{t-1}
\end{aligned}
$$

We first show that the integrand above evaluates to a normal distribution. From (1), we know that

$$
\displaystyle
x_t \mid x_{t-1}, u_t \sim \mathcal{N}(A_t x_{t-1}+B_t u_t, A\Sigma_{t-1}A^{T}+R_t)
$$

Now, we can express \\( {\overline{bel}(x_t) } \\) as,

$$
\begin{aligned}
\overline{bel}(x_t) &= \int_{x_{t-1}}p(x_t|x_{t-1},u_t)\ p(x_{t-1}|z_{1:t-1},u_{1:t-1}) \mathrm{d}x_{t-1} \\
&= \eta \int_{x_{t-1}} \text{exp } \left\{ - \frac{1}{2} {(x_t - A_t x_{t-1} - B_t u_t)}^T {R_t}^{-1} (x_t - A_t x_{t-1} - B_t u_t) \right\} \\
& \quad \text{exp }\left\{ - \frac{1}{2} {(x_{t-1} - \mu_{t-1})}^T {\Sigma_{t-1}}^{-1} (x_{t-1} - \mu_{t-1}) \right\} \mathrm{d}x_{t-1}
\end{aligned}
\tag{4}
$$


\\( {\eta } \\) is a normalizing factor that collects all constants not expressed, such that our probability density function still integrates to 1.

We want to rearrange the terms in the exponential in (4) in such a way that allows us to integrate over \\( {x_{t-1}} \\). (For convenience, let's denote the terms in the exponential in (4) as \\( {-L_t} \\)). Our strategy is to attempt to decompose \\( {L_t } \\) into two functions, one with the terms \\( {x_{t-1}} \\), and one without, which we shall label as \\( {L_t(x_{t-1}, x_t) \text{ and } L_t(x_t)} \\) respectively, and
identify a form for \\( {L_t(x_{t-1}, x_t)} \\) such that the integral over \\( {x_{t-1}} \\) evaluates to a constant, which we can then subsume into the normalizing constant \\( {\eta } \\). 

To do the above, we observe that the function \\( {L_t } \\) is, quadratic in \\( {x_{t-1} } \\), and exponential over the quadratic.

This suggests that if we can collect the terms with \\( {x_{t-1} } \\) and rearrange it such that it matches a normal probability density, then the integral would evaluate to 1 (or a constant which then gets collected together with \\( {\eta } \\)).

We can construct a normal distribution from an exponential quadratic (see [Appendix](#appendix)). By applying first and second order differentiation to \\( {L_t(x_{t-1}, x_t) } \\), we get,

$$
\displaystyle
\begin{aligned}
\frac{\delta L_t(x_{t-1})}{\delta x_{t-1}} &= - {A_t}^T {R_t}^{-1} (x_t-A_t x_{t-1} -B_t u_t) + {\Sigma_{t-1}}^{-1}(x_{t-1}-\mu_{t-1}) \\
\frac{\delta^2 L_t(x_{t-1})}{\delta {x_{t-1}}^2} &= {A_t}^T {R_t}^{-1} A_t + {\Sigma_{t-1}}^{-1} \\
&=: {\Psi_t}^{-1}
\end{aligned}
\tag{5}
$$

Setting the first derivative to 0 and solving for \\( {x_{t-1} } \\) gives us (after some algebraic manipulation),

$$
\displaystyle x_{t-1} = \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}] \tag{6}
$$

Thus using the \\( {x_{t-1} } \\) we solved for, and \\( {\Psi_t } \\) as the mean and variance of a Gaussian pdf, we have

$$
\displaystyle
\begin{aligned}
L_t(x_{t-1},x_t) &= \frac{1}{2} {(x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}])}^T {\Psi_t}^{-1} (x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}]) \end{aligned}
$$

With this, let us find \\( {L_t(x_t) } \\) by subtracting \\( {L_t(x_{t-1}, x_t) } \\) from \\( {L_t } \\)

$$
\displaystyle
\begin{aligned}
L_t(x_t) &= L_t - L_t(x_{t-1}, x_t) \\
&= \frac{1}{2} {(x_t - A_t x_{t-1} - B_t u_t)}^T {R_t}^{-1} (x_t - A_t x_{t-1} - B_t u_t)\\
&+ \ \frac{1}{2}{(x_{t-1}-\mu_{t-1})}^T{\Sigma_{t-1}}^{-1}(x_{t-1}-\mu_{t-1})\\
&- \ \frac{1}{2} { ( x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}] ) }^T {\Psi_t}^{-1} ( x_{t-1} - \Psi_t[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1}] )
\end{aligned}
\tag{7}
$$

Substituting \\( {\Psi_t } \\) from (5) into (7), and after more algebraic manipulation, it turns out that all terms with \\( {x_{t-1} } \\) cancels out and we are left with

$$
\displaystyle
\begin{aligned}
L_t(x_t) &= \frac{1}{2}{(x_t - B_t u_t)}^T {R_t}^{-1}(x_t - B_t u_t) \\
&+\ \frac{1}{2}{\mu_{t-1}}^T {\Sigma_{t-1}}^{-1}\mu_{t-1} \\
&-\ \frac{1}{2}{ [{A_t}^T{R_t}^{-1}(x_t-B_t u_t) +{\Sigma_{t-1}}^{-1}\mu_{t-1} ]}^T {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1}) }^{-1} [{A_t}^T{R_t}^{-1}(x_t-B_t u_t) +{\Sigma_{t-1}}^{-1}\mu_{t-1}]
\end{aligned}
$$

The forms of \\( {L_t(x_{t-1}, x_t)} \\) and \\( {L_t(x_t)} \\) that we have now tells us two things, that \\( {L_t(x_{t-1}, x_t) } \\) indeed has a Gaussian distribution form which would result in \\( {\int \textnormal{exp } \{-L_t(x_{t-1}, x_t)\}\ \mathrm{d} x_{t-1} = \eta } \\), and
\\( {L_t(x_t)} \\) indeed does not contain any terms with \\( {x_{t-1} } \\)

With the decomposition of \\( {L_t } \\) we have now, we can now continue developing equation (4).

$$
\displaystyle
\begin{aligned}
\overline{bel}(x_t) &= \int_{x_{t-1}}p(x_t|x_{t-1}, u_t)p(x_{t-1}|z_{1:t-1}, u_{1:t-1}) \ \mathrm{d}x_{t-1} \\
&= \eta \int_{x_{t-1}} \text{exp }\{ - \frac{1}{2} {(x_t - A_t x_{t-1} - B_t u_t)}^T {R_t}^{-1} (x_t - A_t x_{t-1} - B_t u_t)\} \text{ exp } \{ -\frac{1}{2} {(x_{t-1} - \mu_{t-1})}^T {\Sigma_{t-1}}^{-1} (x_{t-1} - \mu_{t-1})\} \ \mathrm{d}x_{t-1} \\
&= \eta \ \text{exp } \{-L_t(x_t)\} \int_{x_{t-1}} \text{exp } \{ -L_t(x_{t-1}, x_t)\} \ \mathrm{d}x_{t-1} \\
&= \eta \ \text{exp } \{-L_t(x_t)\}
\end{aligned}
\tag{8}
$$

We notice from (7) that \\( {L_t(x_t)} \\) is also exponential quadratic in the terms \\( {x_t} \\), as such, we again apply the first and second order differentiation technique to construct a normal distribution (see [Appendix](#appendix)). We get,

$$
\displaystyle
\begin{aligned}
\frac{\delta L_t(x_t)}{\delta x_t} &= {R_t}^{-1}(x_t - B_t u_t) \\
&\quad - {R_t}^{-1} A_t 
\left( {A_t}^T {R_t}^{-1} A_t + {\Sigma_{t-1}}^{-1} \right)^{-1} \\
&\quad \cdot \left( {A_t}^T {R_t}^{-1}(x_t - B_t u_t) + {\Sigma_{t-1}}^{-1} \mu_{t-1} \right) \\\\
\frac{\delta^2 L_t(x_t)}{\delta x_t^2} &= {R_t}^{-1} - {R_t}^{-1} A_t 
\left( {A_t}^T {R_t}^{-1} A_t + {\Sigma_{t-1}}^{-1} \right)^{-1} 
{A_t}^T {R_t}^{-1}
\end{aligned}
\tag{9}
$$


Using inversion lemma (see Appendix), we can rewrite (9) as,

$$
\displaystyle
\begin{aligned}
\frac{\delta^2 L_t(x_t)}{\delta {x_t}^2} &= {R_t}^{-1} - {R_t}^{-1}A_t{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{A_t}^T{R_t}^{-1} \\
&= {(R_t+A_t\Sigma_{t-1}{A_t}^T)}^{-1} \\
&=: {\bar{\Sigma}}_t^{-1}
\end{aligned}
\tag{10}
$$

The intuitive interpretation of (10) is that the variance of our normal distribution, $$ {\bar{\Sigma}}_t $$ is the variance of the transition process (1), which is the sum of the error in transition process (1), \\( {R_t } \\), and the variance of the prior period's belief, \\( {\overline{bel}(x_{t-1})} \\) evolved through the linear process \\( {A_t } \\) 

Next, we solve for \\( {x_t } \\) in (9),

$$
\displaystyle
\begin{array}{rcl}
0 &=& {R_t}^{-1}(x_t-B_t u_t) \\
&-& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1}\mu_{t-1}] \\
&\iff& {R_t}^{-1}(x_t-B_t u_t) \\
&=& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}[{A_t}^T {R_t}^{-1}(x_t-B_t u_t) + {\Sigma_{t-1}}^{-1}\mu_{t-1}] \\
&\iff& [{R_t}^{-1} - {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{A_t}^T {R_t}^{-1}](x_t-B_t u_t) \\
&=& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\
&\iff& {(R_t+A_t\Sigma_{t-1}{A_t}^T)}^{-1}(x_t-B_t u_t) \\
&=& {R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \text{ by inversion lemma} \\
&\iff& (x_t-B_t u_t) \\
&=& (R_t+A_t\Sigma_{t-1}{A_t}^T){R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + (R_t+A_t\Sigma_{t-1}{A_t}^T){R_t}^{-1} A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + (I+A_t\Sigma_{t-1}{A_t}^T{R_t}^{-1}) A_t {({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}^{-1}{\Sigma_{t-1}}^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + (I+A_t\Sigma_{t-1}{A_t}^T{R_t}^{-1}) A_t [{\Sigma_{t-1}}{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}]^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + (A_t+A_t\Sigma_{t-1}{A_t}^T{R_t}^{-1}A_t) [{\Sigma_{t-1}}{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}]^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + A_t(I+\Sigma_{t-1}{A_t}^T{R_t}^{-1}A_t) [{\Sigma_{t-1}}{({A_t}^T{R_t}^{-1}A_t + {\Sigma_{t-1}}^{-1})}]^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + A_t(I+\Sigma_{t-1}{A_t}^T{R_t}^{-1}A_t) {(I+{\Sigma_{t-1}}{A_t}^T{R_t}^{-1}A_t)}^{-1}\mu_{t-1} \\
&\iff& x_t = B_t u_t + A_t\mu_{t-1} \\
&\iff& x_t = A_t\mu_{t-1} + B_t u_t
\end{array}
$$

Thus, we obtain the mean of the normal distribution as,

$$
\displaystyle x_t = A_t\mu_{t-1} + B_t u_t
\tag{11}
$$

With (11) as the mean, and (10) as the variance parameter of the normal distribution of (8), we can now say that,

$$
\displaystyle
\overline{bel}(x_t) \sim \mathcal{N}(A_t\mu_{t-1} + B_t u_t, A_t\Sigma_{t-1}{A_t}^T + R_t) =: \mathcal{N}(\bar{\mu}_t, \bar{\Sigma}_t)
\tag{12}
$$

Now the last remaining step we have to do is to combine the transition distribution with the measurement distribution,

$$
\displaystyle
bel(x_t) \\ = p(x_t|z_{1:t}, u_{1:t})
\\ = \eta \ p(z_t|x_t)\ \overline{bel}(x_{t-1})
\tag{13}
$$

Since we now know that both $$ {p(z_t \mid x_t)} $$ and \\( {\overline{bel}(x_{t-1}) } \\) are normal distribution with parameters (1) and (12) respectively, we can write (13) as,

$$
\displaystyle
\begin{array}{rcl}
bel(x_t) &=& p(x_t \mid z_{1:t}, u_{1:t}) \\
&=& \eta \ p(z_t \mid x_t)\ \overline{bel}(x_{t-1}) \\
&=& \eta \text{ exp } \{-\frac{1}{2}{(z_t - C_t x_t)}^T{Q_t}^{-1}(z_t - C_t x_t) \} \\
&& \negmedspace \text{ exp } \{-\frac{1}{2}{ (x_t - \bar{\mu}_t)}^T{\bar{\Sigma}_t}^{-1}(x_t - \bar{\mu}_t) \}
\end{array}
\tag{14}
$$

Let us collect the exponential terms in (14) as \\( {J_t} \\), such that,

$$
\displaystyle
bel(x_t) = \eta \text{ exp } \{-J_t\}
$$

Again we notice that (14) is exponential quadratic in $$ {x_t} $$, which means we can apply first and second order differentiation technique to $$ {J_t } $$ to determine the mean and variance parameters of its distribution, which we also know at this point is Gaussian,

$$
\displaystyle
\begin{array}{rcl}
\frac{\delta J_t}{\delta {x_t}} &=& -{C_t}^T{Q_t}^{-1}(z_t - C_t x_t) + {\bar{\Sigma}_t}^{-1}(x_t-\bar{\mu}_t) \\
\frac{\delta^2 J_t}{\delta {x_t}^2} &=& {C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1} \\
&=:& {\Sigma_t}^{-1}
\end{array}
\tag{15}
$$

Setting (15) to zero, and substituing $$ {\mu_t } $$ for $$ {x_t } $$,

$$
\displaystyle
\begin{array}{rcl}
0 &=& -{C_t}^T{Q_t}^{-1}(z_t - C_t x_t) + {\bar{\Sigma}_t}^{-1}(x_t-\bar{\mu}_t) \\
0 &=& -{C_t}^T{Q_t}^{-1}(z_t - C_t \mu_t) + {\bar{\Sigma}_t}^{-1}(\mu_t-\bar{\mu}_t) \\
&\iff& {C_t}^T{Q_t}^{-1}(z_t - C_t \mu_t) = {\bar{\Sigma}_t}^{-1}(\mu_t-\bar{\mu}_t) \\
&\iff& {C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t = ({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})\mu_t \\
&\iff& \mu_t = {({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})}^{-1}({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t)
\end{array}
\tag{16}
$$

We now have a complete specification of \\( {bel(x_t) } \\) as a normal distribution with mean, and variance as specified in (16) and (15), which proved (3). Our proof by induction of (2) is now complete.

Kalman Gain

We could have used (16) and (15) in our Kalman-filter algorithm as below, and call it a day.

$$
\displaystyle
\begin{array}{rcl}
\mu_t &=& {({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})}^{-1}({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t) \\
\Sigma_t &=& ({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})^{-1}
\end{array}
\tag{17}
$$

However another representation that in more commonly used exists. It also is more efficient by allowing us to avoid having to evaluate double inverses. By introducing a Kalman gain term, we can express (17) as,

$$
\displaystyle
\begin{array}{rcl}
\mu_t &=& \bar{\mu}_t+K(z_t-C_t \bar{\mu}_t) \\
\Sigma_t &=& (I - K_t C_t)\bar{\Sigma}_t
\end{array}
\tag{18}
$$

To go from the variance formulation from (17) to (18), we write

$$
\displaystyle
\begin{array}{rcl}
\Sigma_t \ &=& {(C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t)}^{-1} \\
&=& \bar{\Sigma}_t - \bar{\Sigma}_t {C_t}^T (Q_t + C_t \bar{\Sigma}_t {C_t}^T)^{-1}C_t \bar{\Sigma}_t \\
&=& (I - \bar{\Sigma}_t {C_t}^T [Q_t + C_t \bar{\Sigma}_t {C_t}^T]^{-1}C_t )\bar{\Sigma}_t \\
&=& (I - K_t C_t )\overline{\Sigma}_t
\end{array}
\tag{19}
$$

Where we define the Kalman gain $$ {K_t } $$ as

$$
\displaystyle
K_t = \bar{\Sigma}_t {C_t}^T [Q_t + C_t \bar{\Sigma}_t {C_t}^T]^{-1}
\tag{20}
$$

Next, we show how we can get the mean formulation from (17) to (18),

$$
\displaystyle
\begin{array}{rcl}
\mu_t &=& {({C_t}^T{Q_t}^{-1}C_t + {\bar{\Sigma}_t}^{-1})}^{-1}({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t) \\
&=& (I-K_t C_t)\bar{\Sigma}_t ({C_t}^T{Q_t}^{-1}z_t + {\bar{\Sigma}_t}^{-1}\bar{\mu}_t) \\
&=& \bar{\Sigma}_t {C_t}^T{Q_t}^{-1}z_t - K_t C_t \bar{\Sigma}_t {C_t}^T{Q_t}^{-1}z_t + (I-K_t C_t)\bar{\mu}_t \\
&=& K_t (Q_t + C_t \bar{\Sigma}_t {C_t}^T) {Q_t}^{-1}z_t - K_t C_t \bar{\Sigma}_t {C_t}^T{Q_t}^{-1}z_t + (I-K_t C_t)\bar{\mu}_t \\
&=& K_t (I + C_t \bar{\Sigma}_t {C_t}^T {Q_t}^{-1} - C_t \bar{\Sigma}_t {C_t}^T{Q_t}^{-1})z_t + (I-K_t C_t)\bar{\mu}_t \\
&=& K_t z_t + (I-K_t C_t)\bar{\mu}_t \\
&=& \bar{\mu}_t + K_t (z_t - C_t \bar{\mu}_t)
\end{array}
\tag{21}
$$

Finally, combining (12), (19), (20), and (21), we obtain the Kalman-filter algorithm.

### Appendix

#### Constructing Normal distribution

For a normal probability density function $$ {\mathcal{N}(\mu, \Sigma) } $$, it's pdf has the form

$$
\displaystyle f(x) = \frac{1}{\sqrt{2\pi\Sigma}} \text{ exp }\{- \frac{1}{2} {(x - \mu)}^T {\Sigma}^{-1} (x - \mu)\}
$$

Let the expression in the exponential be denoted as $$ {L(x)} $$, then the parameters of the normal distribution can be obtained as,

$$
{\mu = x \textnormal{ solved for with } \frac{\delta L(x)}{\delta x}} \quad \text{set to 0, and}
$$

$$
{\Sigma^{-1} = \frac{\delta^2 L(x)}{\delta x^2} }
$$ 

#### Inversion Lemma

The inversion lemma (or also known as Woodbury matrix identity) states that for invertible matrices $$ {A, U, C, V } $$ of the correct size, the following holds

$$
\displaystyle
(A+UCV)^{-1} = A^{-1} - A^{-1}U(C^{-1}+VA^{-1}U)^{-1}VA^{-1}
$$

References

Thrun, Sebastian, et al. Probabilistic Robotics. MIT Press, 2010.

