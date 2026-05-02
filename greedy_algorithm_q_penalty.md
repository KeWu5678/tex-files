# Greedy Algorithm for |c|^q Penalty (q < 1)

## The Core Difficulty

In Pieper's Algorithm 1, the insertion criterion is: insert a neuron at omega if |p(omega)| > alpha. This works because:
- Theorem 4 (Pieper) / Theorem 5.3 (thesis) says: if |p(omega)| < alpha for all omega outside the support, then mu is a local minimizer
- So the algorithm keeps inserting until no violation exists — at termination you have a local minimizer

For phi(c) = (1/q)|c|^q with q < 1, this breaks completely:
1. phi'(0) = +infinity, so perturbing with u = delta_omega gives the vacuous bound |p(omega)| <= alpha * phi'(0) = +infinity
2. The zero measure is always a local minimizer (Remark 5(3) in the thesis)
3. You cannot detect "where to insert" by checking |p| > alpha because there is no finite threshold

## The Key Idea: Finite-Step Insertion

The standard greedy step asks: "does adding an infinitesimal neuron at omega decrease J?" — this is what |p(omega)| > alpha captures.

For |c|^q, infinitesimal insertion always increases J (the penalty barrier is infinite at c = 0). But a finite insertion can jump over the barrier. So we ask instead:

> "Does adding a neuron at omega with optimally chosen weight c* decrease J?"

When you insert a neuron at omega with weight c into the current network, the change in J is (using the quadratic structure of L):

```
Delta J(c; omega) = c * p(omega) + (1/2) c^2 * S(omega)^2 + (alpha/q)|c|^q
```

where S(omega)^2 = ||sigma(.; omega)||_H^2 and p(omega) is the dual variable (equation (7) in the thesis).

The insertion criterion becomes: there exists c such that Delta J(c; omega) < 0.

## Analysis of the 1D Subproblem

For c > 0 and p(omega) < 0 (the case where a positive weight helps), set |p| = -p(omega) and define:

```
h(c) = c * S^2 + alpha * c^{q-1}
```

The stationarity condition Delta J'(c) = 0 becomes h(c) = |p|.

Since q < 1, the term c^{q-1} -> +infinity as c -> 0+ and the term c * S^2 -> +infinity as c -> +infinity. So h has a unique minimum at:

```
c_* = (alpha(1-q) / S^2)^{1/(2-q)}
```

and h is convex on (0, +infinity). The structure of Delta J is:

- Delta J(0) = 0
- Delta J increases near 0 (the penalty barrier)
- If |p| is large enough: Delta J has a local max at some c_1, then decreases to a local min at c_2 > c_1
- If Delta J(c_2) < 0, insertion is profitable, and c_2 is the optimal insertion weight

Computing Delta J at the critical point c_2 (where h(c_2) = |p|):

```
Delta J(c_2) = -(1/2) c_2^2 * S^2 + alpha(1-q)/q * c_2^q
```

This is negative iff:

```
c_2 > c_crit := (2*alpha*(1-q) / (q * S^2))^{1/(2-q)}
```

The corresponding threshold on |p| is:

```
p_crit(omega) = h(c_crit) = c_crit * S^2 + alpha * c_crit^{q-1}
```

**Insertion criterion**: insert at omega with weight c_2 iff |p(omega)| > p_crit(omega).

## Consistency Check (q -> 1)

As q -> 1: c_crit -> 0, and h(c_crit) -> alpha. So the threshold recovers the l^1 criterion |p| > alpha as q -> 1.

## Important Features

- c_2 > 0 is bounded away from zero: inserted neurons start with a finite weight, "jumping" over the penalty barrier. This is consistent with Corollary 5.1.1 (minimum atom size).
- p_crit depends on omega through S(omega). On the sphere (Omega = S^d), S is bounded, so p_crit is bounded.
- p_crit > alpha for q < 1: it's harder to justify inserting neurons with the |c|^q penalty (higher bar to clear), which is natural since the penalty has a stronger barrier near zero.

## The Algorithm

```
Algorithm: Iterative node insertion for |c|^q penalty (q < 1)

Input: data {(x_k, V_k, grad V_k)}, alpha > 0, q in (0,1),
       initial network (omega^(0), c^(0)) of width N(0)
       (recommended: warm-start from l^1 solution)

1: for t = 0, 1, ..., T do
2:    Compute dual variable p_t(omega) via equation (7)
3:    [Phase 1: Finite-step insertion]
4:    Sample N_trial random nodes omega in Omega
5:    For each candidate omega, solve the 1D problem:
         c*(omega) = argmin_{c} Delta J(c; omega)
      (splits into c > 0 and c < 0 branches, each a scalar equation)
6:    Insert nodes where Delta J(c*(omega); omega) < -tol
      with initial outer weight c*(omega)
7:    [Phase 2: Local training]
8:    Optimize (omega, c) jointly on J_omega(c) via gradient descent
      (phi'(|c|) = |c|^{q-1} is bounded since |c| >= c_min > 0)
9:    [Phase 3: Pruning]
10:   Remove neurons with |c_n| < c_prune
      (natural threshold: c_prune ~ c_min from Corollary 5.1.1)
11: end for
```

## Differences from Pieper's Algorithm 1

| | l^1 penalty (Pieper) | \|c\|^q penalty (q < 1) |
|---|---|---|
| Insertion criterion | \|p(omega)\| > alpha | Delta J(c*(omega); omega) < 0 |
| Initial weight of inserted neuron | infinitesimal | finite c* > c_min > 0 |
| Pruning threshold | ad hoc | natural: c_prune ~ c_min from Corollary 5.1.1 |
| Zero measure | not a local min | always a local min (must warm-start) |
| Termination guarantee | \|p\| <= alpha everywhere | \|p\| <= p_crit everywhere |

## Termination Criterion as a Local Optimality Condition

> **Proposition (Local optimality for |c|^q)**: Let phi(z) = (1/q)z^q (q < 1). Let mu_bar = sum c_n delta_{omega_n} be finitely supported such that:
> 1. c_bar is a local minimum of J_{omega_bar}
> 2. For all omega not in {omega_1, ..., omega_N} and all c in R: Delta J(c; omega) >= 0
>
> Then mu_bar is a local minimizer of J.

Condition (2) is equivalent to: |p(omega)| <= p_crit(omega) for all omega outside the support.

The proof follows the same structure as Theorem 5.3 — you decompose a perturbation mu into atoms near the support (handled by condition 1) and distant perturbations (handled by condition 2, using the finite-step analysis instead of |p| <= alpha).

## Approximation Guarantee

At termination, |p(omega)| <= p_crit(omega) for all omega. Using this in the fidelity estimate (analogous to Theorem 5.1 / Pieper's Theorem 5):

The key calculation uses <p, mu_bar> = -alpha * sum |c_n|^q <= 0 (from the optimality conditions at atoms), and:

```
-<p, mu_f> <= sup|p| * ||mu_f||_M <= sup p_crit * ||f||_{W(D)}
```

This gives:

```
||N[mu_bar] - f||_H^2 <= sup_{omega} p_crit(omega) * ||f||_{W(D)} + ||y - f||_H^2
```

where sup p_crit scales as alpha^{1/(2-q)} (since 1/(2-q) < 1, this is a weaker bound than the l^1 case, but still -> 0 as alpha -> 0).

## Practical Recommendation

Two-stage approach:
1. **Stage 1**: Solve with phi(z) = z (l^1 penalty) using the existing greedy algorithm (Section 6.2) to get a good initial network
2. **Stage 2**: Switch to |c|^q penalty. The initial weights are nonzero (above the barrier), so gradient-based optimization works. Then apply the insertion/pruning loop with the finite-step criterion.

This avoids the zero-measure basin entirely.
