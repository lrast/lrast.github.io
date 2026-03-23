---
layout: post
title: "Inverse Reinforcement Learning"
date: 2026-03-20
tags: [computational models, review, draft outline]
---

A _reinforcement learning_ problem is composed of a set of states, a set of actions, a state-action dependent transition function $p(s' \| s, a)$, and a state- (or state-action-) dependent reward function $R(s)$.
The objective of the agent in such a problem is to find a _policy_ $\pi(s)$, determining the actions that it should take in any given state in order to optimize the expected future reward

$$ V^{\pi}(s) = R(s) + \mathbb{E}_{p, \pi} \left[ \sum_i \gamma^i R(s_i) \right]. $$

This can be a discounted ($\gamma < 1$) infinite-time reward or a finite-time reward, discounted or not. 
Applying dynamic programming, we can write this value function in recursive form, as the Bellman Equation:

$$ V^{\pi}(s) = R(s) + \gamma \mathbb{E}_{a \sim \pi(s)} \left[ \mathbb{E}_{s' \sim p(s' | s, a)}[ V^{\pi}(s') ]  \right] $$

And as a state-action dependent value, the $Q$ function:

$$ Q^\pi(s, a) = R(s) + \gamma \mathbb{E}_{s' \sim p(s' | s, a)}[ V^{\pi}(s') ], $$

which can be used to determine the optimal policy by $ \hat \pi (s) = \textrm{argmax}_a Q^\pi(s, a) $



The _inverse reinforcement learning_ (IRL) problem assumes that we observe an 'expert' agent that has fully-learned an optimal policy under some unknown reward function $R(s)$. 
Our objective is to learn about the reward function that this agent is optimized for.
Assuming that there are $n$ states and $k$ actions, the main question for this write-up will be the sample-complexity in $n$ (and $k$): how does the number of data-points required to learn the reward function scale with the size of the states and actions.
We will be particularly interested in whether it is possible to achieve $\log n$ scaling, implying that the amount of data required scales reasonably with the _dimension_ of the state space.


There are multiple possible objectives and multiple types of observations that can be used for inverse reinforcement learning, each of which leads to different sample complexity limits.
First, the observations could be observations of individual sample trajectories produced by the expert, or observations of the policy function of the expert.
As we will see, policy observations are significantly more powerful, since they allow us to observe all states at once, even those which are rarely or never visited by an expert, saving us a factor of $O(n)$.
In terms of objectives, we could be aiming to determine the reward function $R(s)$ itself (to within $\epsilon$), or simply to learn a policy that performs as well as the expert (to within $\epsilon$).
Policy learning will be significantly easier, because it only requires characterizing the aspects of the reward that are relevant to the optimal policy, again saving $O(n)$ samples.
Finally, there are several different types of experiments that we could run: passive observation, active learning by probing particular states under fixed transitions, active learning by probing the agent under controllable state-state transition dynamics, and active learning by adding additional rewards to states of our choosing.
We will address these cases in turn below.



### Reward identification, trajectory observations

This is the best studied case, and also the hardest. 
Work along this line started with [^1] which noted that the Bellman equation and argmax actions combine to give a set of linear constraints on the possible reward functions

$$ (P_{a^*} - P_a)(I - \gamma P_{a^*})^{-1} R \ge 0 \quad \forall a \ne a^* $$

Where $P_a$ is state-state transition matrices given the action $a$, $P_{a^*}$ is the state-state transition matrix under the observed optimal action (at each state), and $R$ is the unknown reward function.
Note that the inequality applies element-wise, and that these equations must hold for all actions $a$ besides the observed optimal action.


These constraints usually leave a large subset of possible reward functions (to include constant reward functions) that all explain the observations.
This remains the case even when we take into account the arbitrary scaling and zero-point of the reward function. 
The original paper [^1] resolved this degeneracy by finding sparse rewards that give the greatest value difference between observed actions and next-best actions, resulting in a linear-programming problem.


Sample complexity results for this problem were derived by [^3], which again asked for sparse rewards, but required that the fit results are strictly better than zero reward (so the inequality above is strict), allowing them to treat the problem as a one-class L1-regularized SVM.
They asked how many sample are required to ensure (with high probability) that rewards fit this way, using empirical transition matrices, are still viable solutions under the ground-truth transition matrices, finding $O(n^2 \log nk)$ samples are required (or $O(d^2 \log nk)$ if the transitions are known to be d-sparse).
Note that, in addition to trajectory demonstrations, this approach requires query access to state-action outputs, in order to find the $P_a$ matrices, which would never be sampled otherwise.
The polynomial sample complexity comes from two places: 1) estimation of the stochastic transition matrices requiring $n \log n$ samples, 2) accurate estimation of the matrix inverse requiring an additional factor of $n$ samples.


Later work by the same authors [^4] showed that a similar lower bound on sample complexity $O(n \log n)$, as did a set-based analysis, focused on minimizing distances between sets of feasible reward functions[^2].
A continuous space analysis [^11] also found $O(n^2 \log n)$ sample requirement, with $n$ the number of basis functions.
Notably, the $n \log n$ bound agrees with the sample complexity bound for _forward_ reinforcement learning [^9], which is similarly limited by exploration of the state-action transition landscape.
In a sense, this result provides hope for applications of these types of approaches to high-dimensional problems, given the success of reinforcement learning in games and similar high-dimensional settings.

If, instead of _knowing_ the state-state transition function, we have the ability to _control_ it, we can do slightly better by following the experimental approach of [^8], applied to trajectory observations.
This approach re-scales rewards to $\[0, 1\]$ to remove representational degeneracy in the reward function, and defines $\epsilon$-closeness in terms of (infinity-norm) closeness of the values of the rewards.
Identification experiments become algorithmic probes of rewards, using specially designed environments, as follows:
1) Identify the maximally rewarded state (reward=1, by definition) by allowing the agent to transition from a start state to any other state, where it will stay.
This takes constant time.
2) Identify the minimum reward state (reward=0) by iterating through all states, keeping a running-minimum: in a pair-wise comparison, the expert never chooses to transition to this state.
This requires a single loop through the states, costing $O(n)$ samples.
3) Determine the rewards for all other states by iterating through states, allowing the expert to choose between staying the given state, and a 'gamble' that will randomly fall into the minimum or maximum state.
The reward at the state in question is determined (to within $\epsilon$) by binary search on the probability of the maximum-reward vs minimum-reward state.
This takes $O(n \log(1/\epsilon))$ samples.
Thus, the overall approach take $O(n)$ samples.
By controlling the state transitions, this approach allows us to circumvent learning the transition function at all, and instead learn the reward for each state individually. 
Trajectory measurements mean that we must still perform an experiment for each state in turn.



### Policy identification, trajectory observations

This case is also called 'apprenticeship learning' or 'imitation learning'.
Rather than finding a representation of the reward function itself, we instead focus on learning a policy that imitates the expert demonstrations.
The learned policy will be considered $\epsilon$-close to the expert if its expected total reward is within $\epsilon$ of the expected total reward of the expert policy.

This was problem first addressed in the IRL context by [^5], which solved the problem by alternating between solving forward reinforcement learning for candidate reward functions, and using these solutions to find new 'maximum margin' reward functions, that produce the strongest advantage for observed trajectories over the computed RL solutions.
They show that the number of samples required scales as $O(n \log n)$.
Following work[^6] re-phrased the steps in this iteration as moves in a two-player game.
Applying tools for approximate game-playing they achieve sample complexity $O(\log n)$ (which they note also applies to the previous methods).
Thus, policy identification is feasible by passive learning, even in high-dimensional state spaces.
Note that these methods require _solving_ the forward RL problem multiple times for different reward functions, which can be computationally-intensive, however, they are data-efficient.


The key idea here is that, rather than thinking about the rewards at different states, we can instead focus on the _expected reward_ that each state will contribute _under the expert policy_.
Treating these expected reward values as points in $n$-dimensional space, the estimation problem reduces to an $n$-dimensional mean estimation problem, which requires samples that scales as $\log n$ (bounding each estimate individually and using the Hoeffding inequality and union bound).
Essentially, policy estimation allows us to step around the hard problems of reward function degeneracy and unobservability and get measurements for multiple states from single observations (unlike transition function selection, above).
We only care about value _under_ the expert policy which we can estimate directly from the observable expert occupancy, thus making the problem tractable in high dimensions.


On important caveat, however, is that these results assumed that the state-state transition functions are _known_, so that they could focus on learning about the policy. 
_Learning_ the state-state transition functions requires $O(n^3k \log nk)$ passive trajectory observations [^6] to keep the same error bounds on the learned policy.
It is likely that, as above, access to direct state-action probing can reduce this complexity to $O(nk \log n)$, the limit for learning stochastic matrices, but the difficulty still remains.
Thus, similar to reward identification, the limiting step in policy learning is the state-state transition function, not the reward related functions.
The key distinction is that, given this, learning only requires logarithmic, rather than polynomial, samples.


### Reward identification, policy observations 

In this case, we assume that, rather than sampling single expert trajectories, the expert reports it's whole policy: the action that it would perform in every state.
Evidently, this is significantly more information than individual trajectories, and is equivalent to $O(n)$ probes of an expert oracle at specified states.
While this is a fairly strong assumption, we could imagine similar types of data collected from observations of heterogeneous populations or high-dimensional datasets properly viewed.


In this case, active learning approaches are better studied, so we start with them.
Active learning methods for this case were developed in [^8], using controllable state transitions.
These methods proceed identically to the equivalent case with trajectory observations: experiments first identify the minimum- and maximum- reward states, then determine the rewards of the remaining states by having the expert choose whether or not to 'gamble' by choosing an action that will randomly lead to either the maximum- or minimum- reward state.
Binary search on the probability of the maximum vs minimum allows the rewards of other states to be found (to within epsilon) as weighted sums of the maximum and minimum reward values.
(The exact values are arbitrary from the perspective of expert behavior in any environment.)
The three steps in this approach require samples that scale as follows.
Finding the maximum state requires $O(1)$ samples: the expert will always choose the maximum in all-to-all connections.
Finding the minimum state requires $O(\log n)$ samples: states are tested in pairs for which is larger, with the returned policy cutting the number of candidate minimum states in half each iteration.
Finding the rewards of the other states requires $O(\log 1 / \epsilon)$: policy observations means that all states return their choices of whether to gamble or stay in a single experiment.
Thus, the sample complexity of this active learning method is $O(\log n)$.
If we also have the ability to add additional rewards to states of our choice [^7], this reduces further to $O(1)$ because we no longer need to determine the maximum and minimum rewards by comparing states, and can proceed directly to the binary search.
However, the same works [^8] also show that, in the more realistic case, with active selection from a limited number of environments (without control over rewards), the number of 'excess' experiments to achieve as good of results as purpose-designed environments scales $O(n)$.
Thus, this logarithmic scale is reliant on being able to design specific environments to experiment with.


In the less studied realm of passive learning with policy observations, [^7] considers the question of generalization errors on a sequence of tasks, with changing environment and rewards.
The learner sees both the environment and a set of changing rewards, while some intrinsic rewards remain latent, which the learner needs to find based on expert feedback when the learner's policy is sufficiently bad compared to the expert.
The work derives a lower bound on the number of demonstrations needed, when faced with an adversarial environment, of $O(n^2 \log n)$.
Notably, this bound is _unchanged_ when policy observations are replaced by trajectory observations for this specific task.
In a continuous-space analysis, with $n$ basis functions, the analysis of [^12] finds complexity results $O(n)$ for policy observations, $O(n \log n)$ for trajectory observations, and (of course) $O(n \log n)$ probes of the environment.


These works reveal a somewhat unexpected picture of policy observations: when the learner is able to conduct expressive experiments, the ability to observe all elements of the policy simultaneously is very powerful, essentially saving a factor of $n$ experiments.
However, the additional power is not sufficient to qualitatively improve sample complexity for all passive learning approaches, and even when it does, it may only be by a factor of $\log n$.
Thus, policy observations are very helpful when data collection involves investigating states individually, less so when this is not the case.


### Policy identification, policy observations

This case is trivial, requiring $O(1)$ samples in the state size: we observe exactly what we are trying to fit.


### Hybrid approaches:

One interesting work, that doesn't neatly fit into any of these categories is [^10], which used a setting quite similar to reward identification with trajectory observations, but measured goodness of fit based on policy: how well learned reward functions induce high-performing (under the ground-truth reward) policies in other transition matrix environments.
They assumed that the target transition matrices are known, and that the learner has query access to the expert policy and the resulting transitions, and showed that the sample requirement scales as $O(nk)$.
Essentially, querying the expert a constant number of times per state.
This approach was extended by [^13] to cases where the transition matrices are unknown, but the learner gets to specify an exploration policy, used to make action decisions during exploration, while observing expert actions. 
The sample complexity remains $O(nk)$.
Along similar lines, the work [^8] discussed above, use a series of fully-characterized environments, as probes on the reward, again finding linear scaling with the size of state space (at least for active selection of environments).
This objective represents an interesting starting point for applications, in particular, moving toward the case where transition matrices are unknown, but sample-able or perturb-able.
This direction also dovetails nicely with the inverse optimization problems.



## Summary:
Overall, the IRL problem is bound by a couple of limiting steps.
First, learning the state-action transition matrices requires $O(nk \log n)$ steps, even with full active learning by query access.
This is a limiting factor for any approach that uses the transition probability distributions, unless it knows them beforehand.
Second, for reward function learning with individual state observations, there is a limiting $O(n)$ complexity because the reward function can vary arbitrarily with state, so full characterization requires the measuring at each state independently.

Putting aside learning about the environment, passive observations require $O(n \log n)$ samples for reward identification with trajectory observation, $O(n)$ samples for reward identification with policy observations, and $O(\log n)$ samples for policy identification.
Both of these alternative formulations remove one step of inference, either trajectories -> policy, for policy observations, or policy -> reward, for policy identification.
Because policies can be viewed as empirical averages of trajectories, removing this inference step saves significantly less samples than changing the target does.
However, the savings can be leveraged by active learning: reward function learning requires $O(n)$ samples for trajectory observations, and $O(\log n)$ samples or even $O(1)$ samples for policy observations under active learning, depending on the type of experiments that can be performed.

From the perspective of practical characterization in high-dimensional spaces the key point is that characterizing transition dynamics will always be expensive, potentially limiting, regardless of the computations that our observed agent performs.
In this view, the reward function (or it's proxies) has the advantage that it is a _more parsimonious_ description of the system than the transition dynamics, particularly if it can be fit more efficiently.
Thus methods that focus on conditions with _unknown_ or controllable transition dynamics are of major importance.
The methods surveyed here are unable to break the polynomial barrier without access to policy observations, chiefly because of the requirement to characterize the reward function at all states.
To my eye, however, the door remains cracked open for game based approaches (as in policy learning) with reward function generalization objectives, under unknown transition functions to provide a feasible, but reasonable approach.


## References:

[^1]: (Ng & Ressel, 2000) Algorithms for Inverse Reinforcement Learning

[^2]:(Lazzati, Metelli & Restelli, 2023) On the Sample Complexity of Inverse Reinforcement Learning

[^3]: (Komanduru and Honorio, 2019) On the Correctness and Sample Complexity of Inverse Reinforcement Learning

[^4]: (Komanduru and Honorio, 2021) A Lower Bound for the Sample Complexity of Inverse Reinforcement Learning 

[^5]: (Abbeel & Ng, 2004) Apprenticeship Learning via Inverse Reinforcement Learning

[^6]: (Syed & Schapire, 2007) A Game-Theoretic Approach to Apprenticeship Learning

[^7]: (Amin, Jiang & Singh 2017) Repeated Inverse Reinforcement Learning

[^8]: (Amin & Singh 2016) Towards Resolving Unidentifiability in Inverse Reinforcement Learning

[^9]: (Azar, Munos & Kappen 2012) On the Sample Complexity of Reinforcement Learning with a  Generative Model

[^10]: (Metelli, et al. 2021) Provably Efficient Learning of Transferable Rewards

[^11]: (Dexter, Bello & Honorio, 2021) Inverse Reinforcement Learning in a Continuous State Space with Formal Guarantees

[^12]: (Kamoutsi et al, 2024) Randomized algorithms and PAC bounds for inverse reinforcement learning in continuous spaces

[^13]: (Lindner, Krause & Ramponi 2023) Active Exploration for Inverse Reinforcement Learning
