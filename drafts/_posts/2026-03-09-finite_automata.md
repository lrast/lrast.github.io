---
layout: post
title: "Fitting Finite Automata"
date: 2026-03-09
tags: [computational models, review, draft outline]
---

Finite state machines (finite automata) provide a simple model of computation: the machine starts from some initial state, observes a series of inputs, which cause it to transition to different states depending on the input and the current state, and then outputs a value, dependent on its final state.
As a well-studied model of simple computations, it is not surprising that finite state machines are one of the earliest and best-studied computational inverse problems.
They also provide one of the cleanest sample complexity results. 
Learning large (many state) state machines is not feasible for passive learners, which attempt to fit bulk-collected data: such learners require a number of data samples that is exponential in the size of the machine. 
Active learners, on the other hand, which can request information about specific points, _are_ able to learn large state machines, with the number of samples growing quadratically with the number of states in the machine.

To state the learning problem formally, assume that we have observations of the behavior of a machine $\mathcal{M}$, which either accepts or rejects strings, $ s $.
The learning problem is to fit a model $\tilde{\mathcal{M}}$, based on these observations, that agrees with $\mathcal{M}$.
This could be _exact agreement_, on all possible inputs, but a more practical agreement metric is provided by the PAC learning approach [^1].
Assume that train and test samples are both drawn from a distribution $p(s)$.
We ask that, with probability at least $(1-\delta)$, the learned model $\tilde{\mathcal{M}}$ agrees with $\mathcal{M}$ at least $(1-\epsilon)$ fraction of the time on whether to accept or reject.
Thus,

$$ P(d(\mathcal{M}, \tilde{\mathcal{M}}) \le \epsilon) \ge 1- \delta $$

Note that the distance $d$ is the disagreement between the models, and the probability distribution is taken over both train and test samples.
Such a learning problem is feasible if the number of datapoints required scales polynomially with  $\epsilon$, $\delta$, and the number of states in $\mathcal{M}$.


For an overview of PAC learning as a whole, as well as the specific results about finite state machines see [^1]. 
Here, I will sketch the key important results about learning finite state machines (FSMs). 
First, passive learning of finite state machines is difficult.
This was show in [^2], which showed that passive learning of FSMs would produce methods that are able to break RSA cryptography, a problem that is thought to be hard.
That is, passive FSM learning reduces to a cryptographically hard problem.
On the other hand, the foundational work of Angluin [^3] showed, that finite state machines _can_ be learned using active learning, with polynomially many samples. 
This work used two types of queries: 'membership queries', which allow the model to get the correct classification of a specific sequence, and 'equivalence queries', which check whether the learned model is identically correct, and if not produce a counter-example.
From the perspective of an experimenter, membership queries map well to experiments that we could imagine running, but equivalence queries do not.
However, this work used equivalence queries to achieve exact agreement.
If we loosen to PAC agreement, then we can simulate equivalence queries by testing the model on a passively collected dataset, scaling the number of samples to achieve the desired accuracy, called _stochastic equivalence_.
Thus, FSMs can be efficiently learned to PAC agreement by active learning with membership queries only.
Interesting to note, this approach does not work in reverse [^4] (turning PAC to exact learning by adding equivalence queries), and does not work for other classes of problems [^5].

The core hardness of the finite automaton learning problem arises from the fact that the accept / reject decision occurs only at the end of the sequence, with no intermediate information being seen.
This means that the machine can 'hide' combinatorial complexity in the internal transitions that is not visible to random input probes.
Active learning allows us to perform more specific probes of the internal state dynamics.
For this reason, the above results do not extend to models with in-built stochasticity and (optionally) random outputs, like hidden Markov models, and their direct analogs, called 'Probabilistic Deterministic Finite Automata' (because they randomly transition between single states).
Stochasticity interrupts the models' ability to hide complexity, but at the same time makes the overall computations done more difficult.
This is show in[^6]: the _sample complexity_ of learning such models is polynomial in the size, but the _time complexity_ of the _computations_ to recover the model parameters is exponential in the size of the alphabet (under $ P \ne NP$).
Later work showed that, enforcing 'distinguishability' of states (so their transition distributions are 'different enough') allows computational complexity to be polynomial as well, including in distinguishability.
See [^7] for an overview of these ideas.

Note that these results do not involve active learning, which is not longer necessary in the stochastic case.
In fact, the HMM model does not usually include inputs.
For Markov inputs, these are simple to include in a PAC setting by extending the state to include inputs whose transitions are set by the environment distribution.
These are directions to look into in the literature: are we able to achieve improvements in the size scaling with active learning approaches?
What about with modifications to the HMM itself?





## References:

[^1]: (Michael J. Kearns, Umesh Vazirani, 1994) An Introduction to Computational Learning Theory

[^2]: (M Kearns, L Valiant, 1994) Cryptographic limitations on learning boolean formulae and finite automata

[^3]: (Angluin, 1987) Learning Regular Sets from Queries and Counterexamples

[^4]: (Angluin, 1988) Queries and Concept Learning

[^5]: (Angluin, Kharitonov 1991) When Won’t Membership Queries Help?

[^6]: (Abe and Warmuth, 1992) On the Computational Complexity of Approximating Distributions by Probabilistic Automata

[^7]: (Balle, Castro, Gavaldà 2012) Learning probabilistic automata: A study in state distinguishability

