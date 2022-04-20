---
title: Metric Space
date: 2022-04-20 08:34:18
tags: Algorithm
---

Metric Space, Cover, Pack and Lipschitz Continuity.

<!--more-->

## Metric Space

In mathematics, a metric space is a nonempty set together with a metric on the set. The metric is a function that defines a concept of *distance* between any two members of the set, which are usually called points. 

The metric satisfies a few simple properties:

- the distance from $A$ to $B$ is zero if and only if A and B are the same point.

- the distance between two points are positive.

- the distance from $A$ to $B$ is the same as the distance from $B$ to $A$, and

- the distance from $A$ to $B$ is less than or equal to the distance from A to B via any third point C.

## Covering Number and Packing Number

In mathematics, a covering number is the number of spherical balls of a given size needed to completely cover a given space, with possible overlaps. 

Two related concepts are the packing number, the number of disjoint balls that fit in a space, and the metric entropy, the number of points that fit in a space when constrained to lie at some fixed minimum distance apart.

Let $(Z, d)$ be a metric space, let $K$ be a subset of $Z$, and let r be a positive real number. Let $B_r(x)$ denote the ball of radius $r$ centered at $x$. A subset $C$ of $Z$ is an $r-external$ covering of $K$ if:

$$K \subseteq \cup _{x\in C} B_r(x)$$

In other words, for every $y \in K$ there exists $x \in C$ such that $d(x,y) \leqslant r$.

If furthermore $C$ is a subset of $K$, then it is *an r-internal covering*.

The *external covering number* of $K$, denoted $N\_{r}^{\text{ext}}(K)$, is the minimum cardinality of any external covering of K. The *internal covering number*, denoted $N\_{r}^{\text{int}}(K)$, is the minimum cardinality of **any** internal covering.

A subset $P$ of $K$ is a packing if $P \subseteq K$ and the set $\{B\_{r}(x)\}\_{x\in P}$ is pairwise disjoint. The packing number of $K$, denoted $N\_{r}^{\text{pack}}(K)$, is the maximum cardinality of **any** packing of $K$.

A subset $S$ of $K$ is r-separated if each pair of points $x$ and $y$ in $S$ satisfies $d(x, y) \geqslant r$. The metric entropy of $K$, denoted $N_{r}^{\text{met}}(K)$, is the maximum cardinality of **any** r-separated subset of $K$.

Covering number and packing number will be written as $N(Z, d, r), M(Z, d, r), $ respectively.

Metric entropy also equals to $log N(Z, d, r)$.

![](/img/Metric-Space/coveringAndPacking.png)

## Lipschitz Continuity

In mathematical analysis, Lipschitz continuity, named after German mathematician Rudolf Lipschitz, is a strong form of uniform continuity for functions. Intuitively, a Lipschitz continuous function is limited in how fast it can change: there exists a real number such that, for every pair of points on the graph of this function, the absolute value of the slope of the line connecting them is not greater than this real number; the smallest such bound is called the Lipschitz constant of the function (or modulus of uniform continuity). For instance, every function that has bounded first derivatives is Lipschitz continuous.

> We have following chain of strict inclusions for functions 
> over **a closed and bounded non-trivial interval** of the **real line** (So strong!)
> 
> Continuously Differentiable $\subset$ Lipschitz Continuous  
> $\subset$ $\alpha-$ Holder Continuous $\subset$ Uniformly Continuous = Continuous

Lipschitz Continuity Defintion:

Given two metric spaces $(X, dX)$ and $(Y, dY)$, where $dX$ denotes the metric on the set $X$ and $dY$ is the metric on set $Y$, a function $f : X \rightarrow Y$ is called Lipschitz continuous if there exists a real constant $K \geqslant 0$ such that, for all $x_1$ and $x_2$ in $X$,

$$dY(f(x_1), f(x_2)) \leqslant KdX(x_1, x_2)$$

