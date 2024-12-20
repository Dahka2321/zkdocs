---
weight: 10
bookFlatSection: true
title: "Feldman's Verifiable Secret Sharing"
summary: "A verifiable version of Shamir's secret sharing scheme due to Feldman."
references: ["shamir", "feldman", "craige"]
---
# Feldman's Verifiable Secret Sharing

As Feldman first introduced, Verifiable secret sharing is an extension to Shamir's secret sharing scheme. The idea is that, when generating shares of the secret $S$, the splitting party also generates a set of public values that shareholders can use to validate their shares.

For simplicity, we will use "Sherry" to refer to the party generating the shares of a secret $S$. We denote a share using a lower-case $s$, and a shareholder (or player) as $P$. We will assume a $\left(k, n\right)$ threshold scheme, meaning that $k$ shares are enough to recover $S$ and fewer do not obtain any information about $S$. In some protocols, the players may perform the reconstruction, independently or collaboratively.

## Share Generation and Verification

### Share Generation

In standard Shamir secret sharing, Sherry generates a polynomial $$ f \left( x\right) = a_{0} + a_{1} x + \cdots + a_{k-1} x^{k-1} \in \mathbb{F}\left[x\right] \enspace ,$$ where $a_{0}=S$ and $\mathbb{F}$ is a finite field of characteristic $p$, where $p$ is a suitably large prime. Shares are generated as $s_{i}=\left(x_{i},f\left(x_{i}\right)\right)$ for $i=1,\ldots,n$. Sherry then sends $s_{i}$ to $P_{i}$ over a secure channel. Shares are generated the same way in Feldman's scheme.

Feldman's scheme extends Shamir's scheme by having Sherry extend the shares to include verification values, as well as  _publicly share_ an additional set of values that each player $P_{i}$ can use to verify that their share $s_{i}$  is valid.

The public values are generated by translating them into elements of a commutative group $G$ for which the discrete logarithm problem is hard. For expository purposes, we'll start with $\z{q}$, the multiplicative group of integers modulo a large prime $q$, where $q-1$ is divisible by a large prime $r$, and a generator value $g\in\z{q}$ such that $\left|g\right|=r$.

### Verification Values

Let $A_{j}=g^{a_{j}}\pmod{q}$ for $j=0,\ldots,k-1$. These are the verification values, and Sherry publishes all of them on a public broadcast channel.

Recovering any $a_{j}$ values from the corresponding $A_{j}$ would require solving a discrete logarithm problem over $\z{q}$, which is presumed to be hard. Note that $A_{0}=g^{a_{0}}=g^{S}\pmod{q}$; solving the discrete log for $A_{0}$ would recover $S$ directly.

### Verification of Shares

To verify their share $s_{i}=\left(x_{i},f\left(x_{i}\right)\right)$, player $P_{i}$ computes:

$$
V_{i}=\prod_{j=0}^{k-1}{A_{j}^{x_{i}^{j}}} \enspace .
$$

For each $j$, we have $A_{j}=\left(g^{a_{j}}\right)^{x_{i}^{j}}=g^{a_{j}\cdot x_{i}^{j}}$, which means

$$
V_{i}=g^{a_{0}} g^{a_{1}\cdot x_{i}} g^{a_{2}\cdot x_{i}^{2}}\cdots g^{a_{k-1}\cdot x_{i}^{k-1}}=g^{a_{0} + a_{1}\cdot x_{i} + \cdots + a_{k-1}\cdot x_{i}^{k-1}}=g^{f\left(x_{i}\right)}
$$

Since $P_{i}$ knows $f\left(x_{i}\right)$  as part of their share, they can compute $V_{i}'=g^{f\left(x_{i}\right)}$ directly from the value of $f\left(x_{i}\right)$ given in $s_{i}$. If $V_{i}'=V_{i}$, then $P_{i}$ accepts $s_{i}$  as valid; otherwise, they reject $s_{i}$.

## Variations

Feldman points out that it is not necessary to use $\z{q}$ for generating the verification values. The paper discusses using elliptic curves over finite fields. Using the more familiar point multiplication notation for an elliptic curve group $G$ with a generator point $P\in G$, we have $V_{i}=\left[a_{0}\right]P+\left[a_{1}x_{i}\right]P+\cdots +\left[a_{k-1}x_{i}^{k-1}\right]P=Y_{i}$.

## Security pitfalls

 - **Forgery attack:** Craige describes a [forgery attack](https://www.jcraige.com/vss-forgery) against Feldman's technique when broadcast channels are incorrectly implemented. If an attacker can change each player's view of a single $A_{j}$ value, then it is possible to forge invalid shares for each player that will pass the verification step.
 - **Zero share attacks:** All of the pitfalls of Shamir's secret sharing scheme (e.g., generation of zero shares) still apply.
 - **Improper parameter choice:** The $p$, $q$, and $g$ values should be selected with care. While Shamir's original paper suggested using $p=2^{16}-15=65521$, this will limit the coefficients to 16 bits; recovering the $a_{j}$ values from their corresponding $A_{j}$ values becomes trivial. Further, improper selection of $q$ or $g$ can lead to several discrete log attacks.
