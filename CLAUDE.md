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

### Proof Details: Why the Balancing Trick Works

For the penalty g(tau) = A * tau^P + B * tau^{-Q} (A, B > 0):
1. g -> infinity as tau -> 0+ and tau -> infinity, with exactly one critical point => it is a **global minimum** (not a maximum)
2. The minimum value <= the original (unscaled) penalty, because the original corresponds to a specific tau, and we minimize over all tau > 0
3. This does not change the problem — every (c, omega) can be uniquely reparametrized as (c_tilde/tau^n, tau*omega_tilde), and L depends only on the function, not the representation
