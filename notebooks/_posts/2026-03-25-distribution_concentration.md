---
layout: post
title: "Theory Note: Three Notions of Distribution Concentration"
date: 2026-03-30
tags: [computational models, theory]
---

In this note, I will discuss three notions of distribution concentration, or distribution similarity.
While they capture different ideas, we will see that they are mathematically equivalent.
They are:

1. $f$-divergences between distributions
2. concentration of Fisher information in parameter space
3. reward functions during policy imitation

We also note two further notions of distribution concentration 
<ol start="4">
  <li>regression loss functions</li>
  <li>earth mover distance </li>
</ol>

Which we will see _only_ overlap with the $f$-divergence related notions at specific individual points.
In both cases, the limitation is a result of the fact that these measures care about distances in the underlying space, while $f$-divergence related notions are only sensitive to probability density ratios.

### $f$-divergences

The $f$-divergences are a family of divergence measures between probability distributions.
Each member is defined by a convex function $f$, giving the divergence measure

$$ D_f(p || q) = \int_X f \left( \frac{p(x)}{q(x)} \right) q(x) dx $$

where $p$ and $q$ are probability distributions over a space $x \in X$, and ${q(x) = 0}$ only where ${p(x) = 0}$. 
Note that that this includes, among others, forward and reverse KL divergences and total-variation distance.
However, there are many alternative divergence measures that are not included in this class as well.

$f$-divergences are a theory construct, producing a general class of divergences that are useful for questions about statistical inference and information theory.
$f$-divergences satisfy several intuitive properties that we might expect of divergence measures, including the data-processing inequality, decreases under coarse-graining, and convergence and increases in their entropy measures under Markov process dynamics (H-theorems).



### Fisher information concentration

The Fisher information is a measure of how sensitive an estimator is to changes in the quantity that it is trying to estimate.
Let $s$ be a quantity of interest, that we are trying to estimate based on observations of $r$.
The Fisher information is given by

$$ I(s) = \mathbb{E}_{p(r|s)} \left[ \left( \frac{d}{ds}  \log p(r|s)   \right)^2 \right] $$


This captures how much the distribution of $r$ values changes when $s$ changes, providing a limit on the variance of estimations of $s$.
Large Fisher information means that $r$ changes a lot when $s$ changes, implying that low variance estimation is possible, vice versa, small Fisher information means small changes in $r$ with $s$, meaning that estimations of $s$ must have high variance.

If we are designing a system with limited representation capacity to capture the value of $s$, we may not want the same sensitivity everywhere, instead preferring to make our most sensitive estimates where they matter most, or where they are used most often.
This idea leads to the optimization problem

$$ \max_{I(s)} \int f \left(\sqrt{I(s)} \right) p(s) ds $$

Here $f$ is a _concave_ function which determines how much our estimator favors concentrated versus diffuse Fisher information (i.e. large FI somewhere vs medium FI everywhere).
Note that we are maximizing over Fisher information curves, which will generally require a capacity constraint (e.g. on total Fisher information) to prevent divergence.
The Fisher information in the integral is written inside a square-root for mathematical convenience (see also Fisher information metric), but this can be absorbed into the arbitrary function $f$.

If we imagine that our system contains an internal estimate $\hat s(s)$ of the target value (this can be entirely implicit), we can rephrase the objective in terms of that value

$$ \int f \left(\sqrt{I(\hat s)} \frac{p(\hat s)}{p(s)} \right) p(s) ds $$

Noting that our concept of the internal estimate can have arbitrary scaling, we rescale to units that we can actually measure, units of the Fisher information of the estimate with respect to the observable $r$: $d \tilde s = \sqrt{I(\hat s)} d \hat s$
This gives

$$ \int f \left(\frac{p(\tilde s)}{p(s)} \right) p(s) ds, $$

Swapping from maximization to minimization and ($f \to -f$) convex to concave, we get an $f$-divergence minimization problem between the environmental distribution of $s$ and the distribution of estimates $\tilde s$.

Thus, we see that the question of how the sensitivity of an unknown estimator varies with the target value is equivalent to a question of $f$-divergence between the distributions of inputs and estimates.
In this context, the function $f$ corresponds to a measure of how much the Fisher information should be concentrated.
This style of question often arises in the context of neuroscience, when characterizing internal representations of quantities in the environment.



### Imitation learning


Recent work [^1] highlights an interesting connection between $f$-divergences and imitation learning in inverse reinforcement learning (see also [my survey of IRL complexity results]({% post_url /reviews/2026-03-20-inverse_RL %})).
In this case, we start from the $f$ divergence

$$ D_f(p || q) = \int_X f \left( \frac{p(x)}{q(x)} \right) q(x) dx $$

We can think about $p$ and $q$ as _policies_, distributions of actions given states.
Here $p$ is the _imitator_ policy, whose goal is to be as close as possible to $q$, the _expert_ policy, with closeness measured by the $f$-divergence.
Rewriting the $f$-divergence using a variational expression[^2] gives:

$$ D_f(p || q) = \max_{T(x)} \mathbb{E}_{p}[T(x)] - \mathbb{E}_{q}[ f^* (T(x))] $$

Where $T$ is an arbitrary statistic, and $f^*$ is the convex conjugate of $f$, defined as:

$$ f^* (t) = \max_y ty - f(y) $$

Intuitively, $f(y)$ is the 'cost' of mismatch between imitator and expert densities.
If we think about $t$ as the 'price' paid per degree of mismatch, then the convex conjugate $f^* (t)$ represents the maximum 'profit' available from deviating in exchange for payment.
Note that this transform works in both ways ($f$ is convex) and the identity can be derived by applying the reverse transform to $f$ in the divergence expression.

The price analogy is helpful in understanding, so we'll expand on it.
Based on the identity, $f$-divergence minimization

$$ \min_p  D_f(p || q) = \int_X f \left( \frac{p(x)}{q(x)} \right) q(x) dx $$

can be rewritten as

$$ \min_{p(x)} \max_{T(x)} \mathbb{E}_{p}[T(x)] - \mathbb{E}_{q}[ f^* (T(x))] $$

taking the form of a minimax game.
Following an idea that is often used in the generative adversarial networks, we consider the two functions that are optimized to be actions of two players.
First, the _imitator_ specifies $p(x)$, with the goal of making it close to $q(x)$.
Second, the _discriminator_ attempts to distinguish between $p(x)$ and $q(x)$, which they do by specifying a payout, $T(x)$, that they are paid whenever a sample at $x$ comes from the imitation distribution.

Assume that half of samples come from the imitator, and half of then come nature.
The expected payment to the the imitator is $\mathbb{E}_{p}[T(x)]$, the first term in the minimax expression.
However, this payment alone is insufficient to incentivize the discriminator to perform well, as it could send $T(x) \to \infty$ everywhere.
So, we must have the discriminator pay for its bet $T(x)$, by having it pay the imitator whenever a sample at $x$ comes from the natural distribution.
From the equation, we can see that this downside should be $f^* (T(x))$, equal to the profit that the imitator takes under the price $T(x)$. 
To argue that this is the 'fair market price' of a bet $T(x)$, consider again the form of $f^*$

$$ f^* (T) = \max_{\frac{p}{q}} T \frac{p}{q} - f \left( \frac{p}{q} \right). $$

This is equal to the income that the discriminator can expect to make from the bet (first term) minus a 'discount' based on how different densities actually are from one another (second term).
Note that large $f$ values are bad for the imitator, but good for the discriminator, because they give it a larger discount on its bets.
We charge the imitator the largest price consistent with this discounted benefit across possible density ratios.


In this viewpoint, at the equilibrium, $T(x)$ are the 'shadow prices': they are the amount that the imitator pays for disagreement with the expert.
Notably, convex duality gives

$$ T(x) = f' \left( \frac{p(x)}{q(x)} \right) $$

The increase in loss from increasing disagreement.
Returning to the reinforcement learning context, the paper[^1] makes the identification

$$ R(s,a) = - T(s, a) $$


Recall that $p$ and $q$ are interpreted as policies $q(x) = \pi_E(a|s)$ is the _observed_ expert policy, $p(x)= \pi_I(a|s)$ is a learned imitator policy.
In other words, the the (implicit) reward for a state-action pair is exactly the amount of incentive that we must give the imitator to disagree more with the expert policy.


This, in turn, gives us a novel interpretation for the function $f$ in the $f$-divergence.
Noting again that $f$ is convex, we see that $f'$ must be monotonic.
Thus, $f$ gives the relationship between rewards and concentration of distributions.
Thinking about it in reverse, if we observe concentration of distributions that is consistent with some $f$-divergence, then we know that $f$ (through its derivative) sets the scale on the underlying rewards.
That is to say, while it does not change which locations are better rewarded than others, changing $f$ changes _how much better_ some rewards are than others, and how this scales with overall reward value.


### Not equivalent: Loss functions during regression

Changing the loss function during regression is a natural idea for modulating how 'concentrated' estimates are.
With loss function $l(y_1,y_2)$, the regression problem takes the form

$$ \min_{h(x)} \mathbb{E}_{p(x,y)} [ l(y, h(x)) ] \approx \min_{h(x)} \sum_{x_i, y_i} l(y_i, h(x_i))$$

The loss is usually thought of as following from the log-likelihood of the noise model for regression, in the max likelihood setting

$$ = \max_{\textrm{model}} \mathbb{E}_{p(x,y)} [ \log p_{\textrm{model}}(x, y)) ] $$

Where the loss is the negative log-likelihood of data under the model.
This, in turn, follows from KL-divergence minimization between empirical $p(x,y)$ and model $p_{\textrm{model}}(x, y))$ probabilities

$$ = \min_{\textrm{model}} D_{KL}(p(x, y) || p_{\textrm{model}}(y, x)) ] $$

Note that the KL divergence also includes an entropy term, but it is purely dependent on the data distribution, and therefore does not impact fit models.

The question that naturally follows is whether this idea can extend to other $f$-divergences? Further, is there any overlap between exploring the space of loss functions and exploring the space of $f$-divergences?
The answer on both fronts appears to be _no_.
The KL $\to$ loss argument relies on the ability to 'split' the divergence into an empirical expectation of functions of the model distribution _alone_ plus further individual terms

$$ f(p/q) q = q l(p) + C_1(q) + C_2(p) $$

or

$$ f(p/q) q = p l(q) + C_1(q) + C_2(p) $$

This _only_ works when $f$ is of the form $\log x$ or $x \log x$ respectively, corresponding to forward- and reverse- KL divergences.
This is formalized by the Bregman divergences, which have the form

$$D_B(p, q) = F(p) + F(q) - \langle \nabla F(q), p - q \rangle$$

where $F$ is a functional, say an integral, $\nabla F$ can be the local derivative inside the integral, and the inner-product is also an integral.
Note that the only 'cross-term' is the expectation from the inner product, as we would desire.
For probability densities, the families of $f$-divergences and Bregman divergences overlap only at the forward- and reverse- KL divergences[^3].

Thus, we have seen that exploring the space of regression loss functions is completely independent from exploring the space of $f$-divergence measures.
These different measures capture different features of the fits: loss functions determine how close empirical and predicted values of the dependent variable $y$ are, while divergence measures determine the similarity of the empirical and predicted distributions in probability density space.
To put a fine point on this, for any $f$-divergence measure, we can construct a family of distributions that solve the $f$-divergence minimization problem, under constraints on the sufficient statistics

$$ \min_{p(x)} D_f(p(x) || h(x) ) \textrm{ such that } \mathbb{E}_{p(x)}[T(x)] = T_0 $$

Where $h(x)$ is the 'base measure' that our estimation distribution should be close to, and $T(x)$ are the 'sufficient statistics' that we want to match to their empirical values, $T_0$. 
This is analogous to KL-divergence minimization, or entropy maximization, which produces the exponential family of distributions.
Each $f$-divergence produces an analogous family:

$$ p(x | \theta ) = h(x) \exp_f (\theta^T T(x) - A(\theta)) $$

Where $\exp_f(.) = f'^{-1}(x)$ is a monotonic function, playing the same role as the exponential in the exponential family, and $A(\theta)$ is the normalization factor (and corresponds to the Legendre dual of the optimal $f$-divergence).
In a classical exponential family, changing the statistics $T(x)$ has the same effect as changing the loss function.
That remains the case here, regardless of the underlying divergence. 
Thus, this form highlights that $f$-divergences and loss functions impact different parts of the estimated distributions.
Loss functions impact distance in the space of the data, which $f$-divergence impacts concentration in the space of probability densities.

 
### Not equivalent: earth-mover distance vs $f$-divergence

There are interesting parallels between variational expressions for $f$-divergences and the frequently studied earth-mover distance between probability distributions, which I noticed while writing this note.
The earth-mover distance is defined as:

$$ EMD(p, q) = \min_{\Pi(x,y)} \int_{X,Y} c(x,y) \Pi(x, y) dx dy $$

under the constraint that

$$ \int_Y \Pi( x,y) dy = p(x), \quad \int_X \Pi(x,y) dx = q(y) $$

That is, we find the joint distribution $\Pi(x,y)$, with marginals $p(x)$ and $q(y)$ that minimizes the average of the cost $c(x,y) \ge 0$ for moving mass from $x$ to $y$.
Intuitively, this is cost of the best possible 'plan' for moving probability density from $p(x)$ to $q(y)$.

The earth-mover distance is equivalent to the dual form (see Kantorovich duality):

$$ EMD(p, q) = \max_{T_1, T_2} \mathbb{E}_{y \sim q(y)}[T_2(y)] - \mathbb{E}_{x \sim p(x)} [T_1(x)] $$

with the constraint

$$ T_2(y) - T_1(x) \le c(x,y) $$

Intuitively, this formulation is equivalent to 'selling' mass at $x$ for a price $T_1(x)$ and 'buying' the mass at $y$ for price $T_2(y)$.
The constraint ensures that the prices paid render this approach at least as expensive as transporting the mass directly.

Meanwhile, $f$-divergences are equivalent to the variational form:

$$ D_f(p || q) = \max_T \mathbb{E}_{x \sim p(x)} [T(x)] - \mathbb{E}_{y \sim q(y)}[ f^*(T(y))] $$

Other than the ordering of terms, there is a strong similarity between these expressions: both evaluate the difference between related two related statistics on the two different distributions.
Earth mover statistics are related by cost-based duality

$$ T_1(x) = \max_y T_2(y) - c(x,y) $$

$f$-divergence statistics are related by convex duality

$$ f^*(p) = \max xp - f(x) $$

The key distinction is that the convex duality acts on the outputs of the statistic $T$ _uniformly across inputs_ , while the cost duality acts differently at different inputs.
This is not surprising given the definitions: the $f$-divergence depends only on the probability density ratio, while earth mover is sensitive to distances in the input space. 
I believe that the two families intersect at the total variation distance, and only there, but will refrain from going deeper here.


## References:

[^1]: (Wulfmeier et al, 2024) [Imitating Language via Scalable Inverse Reinforcement Learning](https://proceedings.neurips.cc/paper_files/paper/2024/file/a5036c166e44b731f214f41813364d01-Paper-Conference.pdf)

[^2]: (Nguyen, Wainwright, and Jordan, 2007) Estimating divergence functionals and the likelihood ratio by penalized convex risk minimization

[^3]: (Amari, 2009) alpha-Divergence Is Unique, Belonging to Both f-Divergence and Bregman Divergence Classes
