"PD" stands for Proportional-Derivative. It's a feedback controller. Forget robots for a second.

---

## Let's start simple. "P" only:

You have a thing at position $x$. You want it at position $x_{\text{target}}$. You apply a force proportional to how far away it is:

$$F = K_p \underbrace{(x_{\text{target}} - x)}_{\text{error}}$$

That's it. Big error -> big force. Error shrinks -> force shrinks. Error hits zero → force hits zero.

**Here's the problem:**

The thing has momentum. It accelerates toward the target, arrives with velocity, overshoots, now error flips sign, force flips direction, oscillates forever. Like a spring with no damping. The next section will address this.

---

## Add the D term

Also apply a force opposing the current velocity:

$$F = K_p(x_{\text{target}} - x) - K_d \dot{x}$$

The $-K_d \dot{x}$ term says: **if you're moving fast, brake**. It doesn't care where you are, only how fast you're moving and in what direction.

$$\dot{x} = \frac{dx}{dt}, \text{ or the derivative w.r.t time.}$$
Recall from Calc I that if $x$ is position, then $\dot{x}$ is simply velocity.

Together:

- $K_p$ pulls you toward the target
- $K_d$ slows you down as you approach

The system settles instead of oscillating.

## What's a "Feedback" Controller?

It reads the current state ($x$, $\dot{x}$) and reacts to it continuously. It's not a pre planned trajectory, it's a loop:

$$\text{measure} \rightarrow \text{compute error} \rightarrow \text{apply force} \rightarrow \text{measure again}$$

Running at a high hz for a robot joint, this loop is fast enough that the joint reliably tracks whatever target you "hand" ;) it.

---

Back to robots. This is why the [Policy](Policy.md) only needs to output $q_{\text{target}}$ for us to walk. The PD controller handles all the low-level physics of actually getting there. The policy operates at the level of "where should this joint be," not "how many newton-meters right now."

