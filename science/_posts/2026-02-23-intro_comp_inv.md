---
layout: post
title: "The sample complexity of computational inverse problems: introduction"
date: 2026-02-23
tags: [computational models]
---

The goal of a __computational inverse problem__ is to describe the behavior of an observed system in terms of a computational problem that the system is solving.
In other words, we assume that our observations about a system can be captured by some computational objective, and aim to describe the system in terms of that computation.
This requires that we develop methods to fit specific features of computational models based on observations of how a system behaves.
Motivations for this approach include imitation learning, adaptation, understanding economic behavior and animal behavior, machine learning interpretability, and functional or physiological modeling of biological systems.


Generally, such problems are quite hard because the universe of computations that an observed system could be solving is large and intricately structured.
As such, fitting computational models can require large amounts of data.
Here, we focus on results about the _sample complexity of learning_: how much data is required for fits.
This is analogous to computational complexity, except that, rather than time or space, we ask how much _data_ is required to _fit_ a model of a system.
Assume that we observe the behavior of a computational system with a certain size, and fit a model based on those observations.
Since the data is random, we can never be guaranteed that the fit is good, so instead we ask that it is 'likely enough' that the fit is 'close enough' to the ground truth (known as Probably Approximately Correct, or PAC). 
We then ask how the number of samples required scales with the likelihood, the closeness, and the size of the observed system.

With an eye toward applications in functional understanding of biology and machine learning systems, we will focus primarily on the sample-number scaling with _size_.
As with other complexity categorizations, scaling laws can be conceptually divided into 1) very easy problems: logarithmic scaling, where each additional sample multiplies the size of systems that can be fit well, 2) feasible problems: polynomial scaling, where sample requirements scale roughly one-to-one with the size of the system, and 3) infeasible problems: exponential scaling, where a small increase in the size of the systems necessitates several times more samples. 
With regard to our applications, the goal is to understand what types of systems we _can and cannot_ expect to be able to fit, so we focus on the feasible / infeasible distinction.
However, the systems that we are dealing with are often high-dimensional, so sample requirement scaling with _dimension_ is a critical concern.
The number of states is exponential in the dimensionality, so polynomial scaling with dimensionality requires logarithmic scaling with number of states.

It is important to emphasize that sample complexity of learning is a function of _the objects that we are trying to learn about_, not the model or methods that we are using to fit the data.
This can be unintuitive for people who are used to thinking about different models and learning approaches, but it again parallels more traditional computational complexity results about the difficulty of problems rather than algorithms.
Sample complexity answers the question of how easy a system is to learn about, independent of learning approach.
It lower-bounds all possible learning algorithms we could try.
If the particular system that we are studying actually falls into a category that is difficult to learn about, then it will _require_ a (potentially impractically) large amount of data to learn the features of that system, regardless of the model that is used to fit it.
Models that do not take this into account can, of course, always be fit to the data at hand, but they will inevitably miss the core of the features that make learning difficult.

In the reviews that follow we will consider three well-studied categories of problems that describe 'function' or 'computation' or 'intent': finite automata, optimization, and reinforcement learning.
__Finite automata__ describe an observed system as performing computations according to the rules of a finite state machine. 
Starting from some initial internal state, the machine observes a series of inputs, which cause the internal state to transition to a new state, depending on the input and the current state of the machine.
It then produces an output depending on the final state.
The _learning problem_ is to learn the initial state, the state/input-dependent transition matrix, and the state-dependent outputs.
__Optimization problems__ describe the behavior of observed systems as the solution of an optimization: there is a objective that describes 'how good' a behavior is, and constraints on the behaviors that can be exhibited, and the system performs the behavior that is best according to these objectives and constraints.
A classical example of this approach is action in physics.
The _learning problem_ is to learn the objective and constraint functions that explain observed behavior.
__Reinforcement learning__ problems act like something of a combination of these two.
The observed system is allowed to choose actions that move it between different states (think locations in the world).
It chooses actions in such a way as to maximize a state (or state-action) dependent reward function.
The _learning problem_ is to determine the reward function that an observed actor is optimal for.
Because we are assuming that the observed agent is already performing optimally relative to the (unknown) reward function, better names for the learning problem could be 'inverse planning' or 'inverse dynamic programming', but it is known as 'inverse reinforcement learning' in the literature.
I will survey each of these problems in turn in future posts.


## Related
- Fitting Finite Automata 
- Inverse Optimization Problems
- Inverse Reinforcement Learning

