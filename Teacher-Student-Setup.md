## The Problem

In simulation you can cheat. You have access to things like exact terrain height, ground friction, contact forces, information the real robot's sensors will never give you. A policy trained on that privileged information learns much faster and behaves better.

But you can't deploy it. The real robot doesn't have that data.

---

## The Fix

Train two policies:

**Teacher**: runs in simulation with full privileged state $(s_t, e_t)$. Trained with reinforcement learning — it receives a reward signal at every timestep, estimates future reward with a value function, computes advantages, and updates its policy via a loss function. The full mechanism is in [Value Function Loss Function](Value%20Function%20Loss%20Function/Value%20Function%20Loss%20Function.md). The privileged data $e_t$ makes the value function more accurate, which makes the advantage estimates cleaner, which makes training faster and more stable.

**Student**: only sees what real sensors can provide ($s_t$ from [State-at-t](State-at-t.md)). Trained to mimic the teacher's *outputs*, not to maximize reward from scratch.

The student's loss is simple: match what the teacher would do given the same situation.

For example:
$$\mathcal{L}_{\text{student}} = \| \pi_{\text{student}}(s_t) - \pi_{\text{teacher}}(s_t, e_t) \|^2 $$

where $e_t$ is hand-waved away privileged "extra" info the teacher sees and the student doesn't.

This is the MSE, mean squared error, which is exactly what it sounds like

---

Once the student matches the teacher well enough, you throw the teacher away and deploy the student.

The student never needed to see the privileged data directly, it absorbed that knowledge indirectly through the teacher's behavior.
