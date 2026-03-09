I honestly can't explain the whole process without first relying on underlying concepts and abstractions that I don't think I should gloss over, so this'll be a longer read. If you want to skim, just skip the table of contents below and keep reading.

Also I can't stand not having LaTeX so I'm hosting this on https://github.com/Avni2000/Wisco-Humanoids-Docs as markdown and I'll just convert to docx when I need to.

Our overarching goal is to make a robot walk, yes? Let's start from the bottom and slowly build up to abstractions. I imagine we'll just refactor this stuff out into a "foundational knowledge" database later, but for now they're necessary building parts.

We'll speak broadly on:
1. What are [PD-Controller](PD-Controller.md)s? <- Click on this link, start here! Then go down in order.
2. How do they connect to [Policy](../Policy.md)?
3. What does the Value Function do?
4. What's the Loss Function, and how does it use all of this?
5. Where does [Teacher-Student-Setup](Teacher-Student-Setup.md) fit in?

---

## Where We Are

Let's recap the chain so far:

$$s_t \xrightarrow{\pi_\theta} q_{\text{target}} \xrightarrow{\text{PD}} \tau \xrightarrow{\text{robot}} s_{t+1}$$

In plain English: the [Policy](../Policy.md) reads the robot's state, outputs target joint angles, the [PD-Controller](PD-Controller.md) converts those into torques, and the robot moves into a new state. Repeat 1000 (for eg.) times a second.


**To be clear** a policy is literally just $12$ numbers. That's it. The policy itself is a neural network,  same structure as in [Multilayer-perceptrons](Multilayer-perceptrons.md):

$$s_t \in \mathbb{R}^{45} \xrightarrow{W_1, b_1} \mathbb{R}^{512} \xrightarrow{f} \xrightarrow{W_2, b_2} \mathbb{R}^{512} \xrightarrow{f} \xrightarrow{W_3, b_3} \mathbb{R}^{12}$$

You run it, you get 12 numbers, you hand them to the PD controller.

Like all our weights, the policy parameters $\theta$ start random. The robot falls at the start. We need a way to judge whether any given $\theta$ is good or bad, and push it in a better direction. That's the job of the value function and the loss function.

There's one more wrinkle: all of this training happens in **simulation**, where we can feed the policy extra information that real hardware will never have, exact terrain height, contact forces, friction coefficients. We've explained this briefly in [Teacher-Student-Setup](Teacher-Student-Setup.md), call that extra information $e_t$. So the policy we're actually training sees:

$$(s_t,\ e_t) \xrightarrow{\pi_\theta} q_{\text{target}}$$

This is the **teacher** policy. It's powerful precisely because $e_t$ makes its job easier. We'll come back to what we do with it once it's trained.

---
## The Value Function

At any state, ask: **"If the teacher keeps acting under its own policy $\pi_{\text{teacher}}$, how much total reward will it accumulate from here?"**

See the proper [Value Function](Value%20Function) for this.
### TODO link to Value Function

That's the value function. It doesn't just mean "some policy." It means *this specific policy, held fixed*. The value function is always defined relative to one particular policy's behavior. $V^{\pi_{\text{teacher}}}$ doesn't mean "this is what will happen," it means **if** the teacher keeps using this exact policy forever from this state, this is the expected total reward. As training progresses and $\theta$ updates, $\pi_{\text{teacher}}$ changes, and so does $V^{\pi_{\text{teacher}}}$. They evolve together.



In plain English: $V$ is the teacher's "opinion" of how good the current situation is, based on how well it itself expects to handle things from here.

High value = the robot is in a position where it tends to accumulate a lot of reward. Low value = it's probably about to fall.

**Note why $e_t$ helps here.** If the robot is about to step onto a patch of ice, $V$ trained without $e_t$ can't see that coming, it just has to guess from joint angles and IMU readings. $V$ trained with $e_t$ (the teacher) knows the friction coefficient is 0.1 and can give a much more accurate prediction. Better $V$ = cleaner training signal.

And from [Value Function](../Value%20Function.md), you can't compute the infinite sum directly, you'd have to run the robot forever from every possible state, averaging over all randomness. So instead you **approximate** $V$ with another neural network, the critic $V_\phi$:

$$s_t \in \mathbb{R}^{45} \xrightarrow{V_\phi} \mathbb{R}^1$$

I don't honestly know anything about this, but basically same MLP structure as the policy, but outputs one scalar instead of 12. You train it by running the robot in simulation, collecting actual experience, and using observed rewards to correct the network's predictions. That's exactly what the value loss below does.

---

## The Advantage

Once you have $V$, you can ask a sharper question: was the action actually taken *better or worse* than what the policy expected?

$$A_t = R_t - V(s_t, e_t)$$

where $R_t$ is the actual return (reward collected going forward from $t$).

- $A_t > 0$: the action led to more reward than predicted — do it more
- $A_t < 0$: the action led to less reward than predicted — do it less

This is the signal that drives policy improvement. Without $V$, you only know whether an episode went well overall. With $V$, you know which specific decisions inside the episode were good or bad.

---

## The Loss Function

Training optimizes two things simultaneously:

**Policy loss**: push $\theta$ to take actions with positive advantage:

$$\mathcal{L}_{\text{policy}} = -\mathbb{E}_t\left[A_t \cdot \log \pi_\theta(a_t \mid s_t, e_t)\right]$$

Minimizing this increases the probability of actions that beat expectations and decreases the probability of actions that underperform.

**Value loss**: make $V$ an accurate predictor:

$$\mathcal{L}_{\text{value}} = \mathbb{E}_t\left[(V_\theta(s_t, e_t) - R_t)^2\right]$$

If $V$ is inaccurate, the advantage estimates are noisy, and the policy update is unreliable. So you train $V$ alongside $\pi$.

The total loss is a weighted sum of both:

$$\mathcal{L}_{\text{teacher}} = \mathcal{L}_{\text{policy}} + c \cdot \mathcal{L}_{\text{value}}$$

Backprop through $\mathcal{L}_{\text{teacher}}$, update $\theta$. Repeat across millions of simulation steps until the teacher walks.

During teacher training, two networks run simultaneously:

| Network                    | Input        | Output                                  | Job                             |
| -------------------------- | ------------ | --------------------------------------- | ------------------------------- |
| Teacher actor $\pi_\theta$ | $(s_t, e_t)$ | $q_{\text{target}} \in \mathbb{R}^{12}$ | decide what to do               |
| Critic $V_\phi$            | $(s_t, e_t)$ | scalar $\in \mathbb{R}$                 | estimate how good this state is |

Neither starts knowing anything. Both are random. They improve together.

---

## So Now You Have a Trained Teacher:

The teacher takes $(s_t, e_t)$ as input. $e_t$ doesn't exist on real hardware. You can't deploy this.

This is where the [Teacher-Student-Setup](Teacher-Student-Setup.md) comes in. You train a **student** policy that only sees $s_t$  (exactly what real sensors provide) and train it to mimic the teacher's outputs:

$$\mathcal{L}_{\text{student}} = \| \pi_{\text{student}}(s_t) - \pi_{\text{teacher}}(s_t, e_t) \|^2$$

(MSE for example. Use whatever loss you want)

The student has a policy network, same architecture, same 12 outputs, but no value function. No reward signal, no advantage. Its only loss is MSE against the teacher's outputs: it's just a function being fitted to match another function. The critic is discarded entirely at this stage.

| Network                              | Input                     | Output                                  | Job                                       |
| ------------------------------------ | ------------------------- | --------------------------------------- | ----------------------------------------- |
| Student actor $\pi_{\text{student}}$ | $s_t \in \mathbb{R}^{45}$ | $q_{\text{target}} \in \mathbb{R}^{12}$ | mimic the teacher without privileged data |
|                                      |                           |                                         |                                           |

The full pipeline:

$$\underbrace{\mathcal{L}_{\text{teacher}}}_{\text{RL in sim with privileged data}} \longrightarrow \underbrace{\mathcal{L}_{\text{student}}}_{\text{imitation on sensor-only input}} \longrightarrow \text{deploy}$$


The value function is what makes the teacher strong enough to be worth copying.