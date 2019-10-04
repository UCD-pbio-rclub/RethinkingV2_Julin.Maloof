---
title: "Chapter 9"
author: "Julin N Maloof"
date: "9/26/2019"
output: 
  html_document: 
    keep_md: yes
---



## Video Notes

* Bayesian is about the posterior but doesn't care how you get there.  It is not about MCMC, that is just one tool to get the posterior.
* King Markov
  - Metropolois Archipelago
  - Must visit each island in proportion to population density.
  - Flip a coin to choose proposed island on left or right. "proposal"
  - census population of proposal island and current island
  - move to proposal with probability of prop  pop / current pop
  - repeat.  this ensures visiting each island in proportion to its population in the long run.
  - why would you do this?  useful if you don't know the distribution of population sizes.  Or in this case the distribution of posterior probabilities.  Allow sampling from unknown posterior distribution.
* Metropolis algorithm
  - will converge in the long run
  - as long as proposals are symmetric
  - not very efficient
  - Useful to draw samples from posterior distribution
  - Island: parameter values
  - Population size: proportional to posterior probability
  - works for any numbers of dimensions (parameters); continuous or discrete
  - Markov chain: history doesn't matter, probability only depends on where you are.
* Why MCMC?
  - can't write integrated posterior, or can't use it
  - multilevel models, networks, phylogenies, spatial models are are hard to get integrated
  - optimization (e.g. quap) not a good strategy in high dimensions -- must have full distributions
  - MCMC is not fancy.  It is old and essential.
* Many MCMC strategies
  - Metropolis-Hastings (MH): More general
  - Gibbs sampling (GS): Efficient version of MH
  - Metropolis and Givvs are "guess and check" strategies.  so quality proposals are essential.  If making dumb proposals then you don't move, and don't visit potentially important parts of the distribution.
  - Hamiltonian Monte Carlo (HMC) fundamentally different, does not guess and check.
* 

## Problems

### 8E1

8E1. Which of the following is a requirement of the simple Metropolis algorithm?
(1) The parameters must bed iscrete.
(2) The likelihood function must be Gaussian.
(3) The proposal distribution must be symmetric.

3

### 8E2

By using conjugate priors (Whatever those are), allowing more efficient proposals.

