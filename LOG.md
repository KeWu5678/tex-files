
### Kurnish paper
However, there's a subtle structural echo. The Kunisch paper shows that controlling the C^2 norm of the approximating family (bound (2.28): max_i ||nabla^2 xi(theta^i)||) is what preserves
semiconcavity. In your setting, the penalty phi controls the measure (weights), not the curvature of the basis functions directly. But both are mechanisms for controlling the "complexity" of the
 approximation to preserve structural properties of the target.

You said:
The semiconcave value function v has gradient discontinuities along the active set boundaries — these are the structural features that Kunisch's Section 3 carefully analyzes. The Hessian of v can blow up
to -infinity at these boundaries (kinks).

My question: 
The discontinuity happens at points where the active set switches, right? which is the discontinuity of the approximating function, not the value function v?

Your answer:
v is also discontinuous in the gradient. The approximation mimic this pattern

You said:
The one concern: the Hessian of the network output has jump discontinuities along hyperplanes {x: a_n . x + b_n = 0}. This is actually interesting — it means ReLU^2 produces piecewise C^2 outputs, which structurally mirrors the active set boundaries in Kunisch's framework. This is exactly the "C^2 but not too smooth" property you were arguing for.


You said:
Section 3 shows that the smooth approximation v_{n,eps} handles this via the probability weights p_{n,j,eps}. As eps -> 0, the probability concentrates on the true active index (Proposition 3.1, eq. 3.13), and the gradient converges to the correct discontinuous one (Corollary 3.1).

For optimal control (Remark 3.1), these boundaries are where the optimal feedback law switches — different optimal trajectories meet. The HJB equation F(x, nabla v) = 0 holds, but the gradient (= optimal control) is discontinuous across these interfaces.

### Liu
You said:
Kunisch requires C^2, not more. ReLU^3 is exactly C^2, not C^3. This is the "just smooth enough" property you identified — the third derivative is discontinuous, so the network Hessian can have
derivative jumps that mirror the active set boundaries. ReLU^4 would be C^3, which over-smooths.

The jump continuity at the boundary of the active set is the property of the approximating function, which is constructed with the finite min, not weighted sum like NN. Your argument that the approximating function has jumps in the gradient doesn't justify we should have discontinuity in the Hessian in the activation functions, right? or the actual value function also has discontinuous gradient in general?


You said: 
Homogeneity cost. ReLU^k is k-homogeneous. The balancing trick exponent k_pen = pq/(kp + q) decreases with k. At k = 3 with p = q = 2 you get k_pen = 1/2, which is already aggressive. At k = 4 you'd get k_pen = 2/5, even more problematic for the optimization (your log penalty helps here, but the landscape still gets harder).

My question: 
Why is it a problem for the optimization?
Your answer: 
the usual nonconvex problem and exploding at 0. 

You said:
proposing: Cubic B-spline activation

A piecewise cubic polynomial, C^2 by construction, with compact support. For example, the cardinal B-spline B_4(z) supported on [0, 4]. Pros:
- Exactly C^2, not smoother — sharp curvature transitions
- Compact support means sigma(.; omega) -> 0 as ||omega|| -> infinity, so (A_infty) holds automatically — your finite support proof extends to Omega = R^d without the sphere constraint
- Localized influence: each neuron affects only a bounded region, which is natural for piecewise structure of the value function

You said:
proposing: Matern 5/2 activation (my top recommendation): sigma(z) = (1 + sqrt(5)|z| + (5/3)z^2) * exp(-sqrt(5)|z|): 
1. Exactly C^2, not C^3 — the third derivative has a jump. This is the "just C^2" property you want.
2. Exponential decay => (A_infty) holds strongly
3. Well-studied in kernel methods (the Matern RKHS with nu = 5/2 is equivalent to H^{5/2}, which aligns nicely with your Sobolev approximation goals)
4. Not homogeneous, but you don't need that
5. Easy to implement, numerically stable (no saturation like tanh, no blowup like ReLU^k)
Why Matern 5/2 over the others: The compact-support options (B-spline, Wendland) introduce hard boundaries where neurons "turn off," which can create flat regions in the dual variable p_t(omega) that make the greedy search harder.

My question: Why does expnential decay matters?
Your answer: just because the gradient is not 0 in the domain unlike b-spline. 
My question: Why not nu = 4/2?
Your answer: something about Bessel basis

You said:
Softplus (pragmatic C^infinity choice): sigma = torch.nn.Softplus():  ()
C^infinity, but doesn't saturate — grows linearly like ReLU. The reason tanh failed in your numerics is likely saturation (vanishing gradients for large inputs), not just being C^infinity. Softplus avoids
this entirely. If the C^2-vs-C^infinity distinction turns out to matter less in practice than avoiding saturation, softplus is a safe baseline to compare against.