# Project Notes

## Repository Structure
- `thesis/Mthesis.tex` — Master's thesis
- `thesis/noncovex regulerization.pdf` — Reference paper on nonconvex regularization for neural networks
- `Optimization/convex analysis.tex` — Convex analysis notes

## Discussion Notes (2026-02-26): Nonconvex Regularization and Sphere Equivalence

### Paper: Nonconvex Regularization (reference in thesis)

**Main formulation**: The paper optimizes a two-layer neural network with inner weights constrained to the unit sphere and a penalty phi(|c_n|) on the outer weights.

### Key Findings

#### 1. Representability vs Optimization Equivalence
- **Representability** (always holds): Any function representable with neurons in R^{d+1} can also be represented on the sphere, by p-homogeneity of the activation. This is the paper's "WLOG" claim on page 2.
- **Optimization equivalence** (only for l^p penalties): The unconstrained all-weights regularized problem is equivalent to the sphere-constrained problem with a modified penalty. Proven in Appendix F, Proposition 3, citing [32] (Neyshabur, Tomioka, Srebro, 2015).

#### 2. Nonconvex Penalties are a Modeling Choice
For the nonconvex penalties used in experiments (log penalty, l^q with q < 1), **there is no equivalence theorem** reducing an unconstrained problem to the sphere formulation. The sphere formulation is a modeling choice motivated by:
- Representational completeness
- Analogy with the l^p case
- Computational convenience (compactness)

#### 3. Proposition 3 — Generalization to n-Homogeneous Activations

##### Discussion about power p - Numerical
Proposition 3 (Appendix F) proves: the unconstrained problem with separable power-type penalty (1/alpha)|c|^alpha + (1/beta)r(omega)^beta is equivalent to the sphere problem with penalty C * |c_tilde|^k.

**For 1-homogeneous (ReLU):**
- k = alpha*beta / (alpha + beta)
- With alpha = beta = 2: k = 1 (lasso)

**For n-homogeneous activations:**
- The rescaling (c, omega) -> (c/tau^n, tau*omega) preserves the network function
- The balancing trick gives exponent on c_tilde: **k = pq / (np + q)**
- With p = q = 2: **k = 2/(n+1)**
- Important: the exponent on ||omega|| is npq/(np+q), which differs from k by a factor of n. Since c_tilde = c * ||omega||^n, the factor of n is absorbed into c_tilde.

**Common error**: Naively applying Proposition 3's formula s = 2pq/(p+q) with "effective" exponents P = np, Q = q gives npq/(np+q) — but this is the exponent on ||omega||, not on c_tilde.

##### Discussion about power p - Theoretical
- Assumption (A1) requires phi'(0) < +infinity. This excludes q-norms phi(z) = (1/q)z^q with q < 1, where phi'(0) = +infinity.
- When phi'(0) = +infinity: the zero measure (no neurons) is always a local solution, because adding any neuron with small outer weight c increases the objective (the marginal penalty cost is infinite near c = 0).
- Consequence: no approximation guarantee can be given for arbitrary local solutions, since the zero measure is a local solution but fits nothing.
- The approximation theorem (Theorem 5) works by bounding the residual via the local optimality condition: gain from best neuron <= phi'(0) * alpha. When phi'(0) is finite this gives a quantitative bound; when phi'(0) = +infinity the argument collapses.
- For q < 1 penalties, good local solutions may still exist, but gradient-based optimization must rely on appropriate initialization to avoid the zero-measure basin.

#### 4. Key References for the Sphere Equivalence
- **[32] Neyshabur, Tomioka, Srebro (2015)** — "In Search of the Real Inductive Bias" — proves the equivalence for l^p penalties via the balancing argument
- **[40] Rosset, Swirszcz, Srebro, Zhu (2007)** — l_1 regularization in infinite dimensional feature spaces
- **[2] Bach (2017)** — Breaking the curse of dimensionality with convex neural networks
- **[33] Ongie, Willett, Soudry, Srebro (2020)** — Bounded norm infinite width ReLU nets

#### 5. Unbounded Dictionary (Omega unbounded)

The paper uses Omega = S^d (compact), so C(Omega)* = M(Omega) holds cleanly via Riesz. If Omega were unbounded (e.g., Omega = R^d for unconstrained inner weights):

- C(Omega)* = M(Omega) breaks down. Replace C(Omega) with **C_0(Omega)** (continuous functions vanishing at infinity).
- The Riesz-Markov theorem still gives: C_0(Omega)* = M(Omega) (finite signed Radon measures).
- The duality pairing is the same: <phi, mu> = integral_Omega phi(omega) d mu(omega), well-defined because phi -> 0 at infinity and mu has finite total variation.
- C_b(Omega) (bounded continuous) has a larger dual — includes finitely additive measures capturing "mass at infinity" — analytically ugly and not suitable here.
- This is part of why the sphere constraint is convenient: compactness makes C(S^d) = C_0(S^d) trivially, and the duality is unambiguous. Working on R^d would require C_0(R^d) and still implicitly controls weights via finite total variation of mu.

---

## Discussion Notes (2026-03-11): Extending Finite Support to Omega = R^d

### Goal
Prove that bar{mu} (a local minimizer of J) is finitely supported when Omega = R^d for general (possibly non-homogeneous) activations.

### The Bottleneck in the Original Proof (page 24)
The paper proves finite support via compactness of Omega = S^d:
- If bar{mu} has infinitely many atoms, their weights c_n -> 0 (finite total variation)
- **Compactness of S^d** => extract convergent subsequence bar{omega}_n -> hat{omega}
- Optimality condition gives |bar{p}(hat{omega})| = alpha; construct competitor mu_N merging the tail, showing J(mu_N) < J(bar{mu}), contradiction

For Omega = R^d, atoms may escape to infinity — no convergent subsequence guaranteed. Homogeneity reduction to S^{d-1} is not available for general activations.

### Proof Plan: Decay Assumption (A_infty)

**New assumption (A_infty)**: sigma(.; omega) -> 0 in L^2(D, nu) as ||omega|| -> infinity.

#### Step 1: Atoms cannot escape to infinity

- The dual variable bar{p}(omega) = <N(bar{mu}) - y, sigma(.; omega)>_{L^2} satisfies:
  bar{p}(omega) -> 0 as ||omega|| -> infinity, by Cauchy-Schwarz and (A_infty)
- For any atom with c_n -> 0, the optimality condition gives:
  |bar{p}(bar{omega}_n)| = alpha * phi'(c_n) -> alpha * phi'(0) = alpha > 0
- If ||bar{omega}_n|| -> infinity: then bar{p}(bar{omega}_n) -> 0, contradicting the above
- **Conclusion**: all atoms are confined to a bounded set in R^d

#### Step 2: Extract convergent subsequence

Since {bar{omega}_n} is bounded in R^d, Bolzano-Weierstrass gives a convergent subsequence bar{omega}_n -> hat{omega} in R^d (no compactness of Omega needed — just local compactness of R^d plus the boundedness from Step 1).

#### Step 3: Run original argument

This step is identical to the paper's proof:
- c_n -> 0, bar{omega}_n -> hat{omega}, and continuity of bar{p} give |bar{p}(hat{omega})| = alpha
- Construct mu_N = bar{mu} - sum_{n>=N} c_n delta_{bar{omega}_n} + C_N delta_{hat{omega}} where C_N = sum_{n>=N} c_n
- Regularization: phi(C_N) <= sum_{n>=N} phi(c_n) by subadditivity of concave phi with phi(0)=0
- Loss: N(mu_N) - N(bar{mu}) = C_N sigma(.; hat{omega}) - sum_{n>=N} c_n sigma(.; bar{omega}_n) -> 0 in L^2 (by continuity of sigma and bar{omega}_n -> hat{omega})
- Together: J(mu_N) < J(bar{mu}) for large N, contradicting local optimality of bar{mu}

### What (A_infty) Rules Out

- **ReLU**: sigma(x; omega) = max(omega^T x, 0) has ||sigma(.; omega)||_{L^2} ~ ||omega|| (growing), so (A_infty) fails. For ReLU, the sphere constraint (or homogeneity reduction) is genuinely necessary.
- **Bounded activations** (sigmoid, tanh): if sigma is bounded and sigma(x; omega) -> 0 pointwise as ||omega|| -> infinity (for fixed x), then by dominated convergence (A_infty) holds if nu has finite mass.
- **Gaussian-like activations**: sigma(x; omega) = exp(-||omega - x||^2 / 2) satisfies (A_infty) with exponential decay.

### Functional-Analytic Consistency

On Omega = R^d (locally compact, non-compact), the correct dual space is C_0(R^d)* = M(R^d) (signed finite Radon measures, by Riesz-Markov). For the optimality conditions to make sense via subdifferential calculus in M(R^d), we need the "test function" sigma(.; omega) to be in C_0(R^d) as a function of omega — i.e., sigma(.; omega) -> 0 as ||omega|| -> infinity. This is exactly (A_infty) (up to L^2 vs uniform decay). So (A_infty) is not just a technical device — it is the natural condition for the duality to work on R^d.

---

### Proof Details: Why the Balancing Trick Works

For the penalty g(tau) = A * tau^P + B * tau^{-Q} (A, B > 0):
1. g -> infinity as tau -> 0+ and tau -> infinity, with exactly one critical point => it is a **global minimum** (not a maximum)
2. The minimum value <= the original (unscaled) penalty, because the original corresponds to a specific tau, and we minimize over all tau > 0
3. This does not change the problem — every (c, omega) can be uniquely reparametrized as (c_tilde/tau^n, tau*omega_tilde), and L depends only on the function, not the representation

