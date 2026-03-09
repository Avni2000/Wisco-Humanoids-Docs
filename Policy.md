> This'll be its own tab/doc (I think Robert's working on it rn) but I wanted to learn a bit about it for this topic

## So we need $q_{\text{target}}$ for our [PD-Controller](PD-Controller.md),  now what?

Something has to produce it. That something is the **policy network**.

---
## The Policy Network

If you're unfamiliar with how neural networks work, read [Multilayer-perceptrons](Multilayer-perceptrons.md) first — the rest of this assumes you know what $W$, $b$, and $f$ are.

See [State-at-t](State-at-t.md) for what the robot can sense. Those readings concatenate into $s_t \in \mathbb{R}^{45}$.

We run that through three layers:

$$s_t \in \mathbb{R}^{45} \xrightarrow{W_1, b_1} h_1 \in \mathbb{R}^{512} \xrightarrow{f} \xrightarrow{W_2, b_2} h_2 \in \mathbb{R}^{512} \xrightarrow{f} \xrightarrow{W_3, b_3} a_t \in \mathbb{R}^{12}$$

Here's what's actually happening at each step and why.

**Step 1: $h_1 = f(W_1 s_t + b_1)$: 45 → 512**

The 45 input numbers are raw sensor readings. They don't directly tell you anything useful on their own -- "joint 3 is at 0.4 radians" means nothing without knowing what the other joints are doing, how fast the body is rotating, what command you're trying to follow.

The answer you need depends on *relationships* between those inputs. Things like: "is my torso leaning left while my left foot is in the air?" or "am I moving forward but being commanded to stop?" Those aren't single sensor readings — they're combinations of many.

You can't compute those relationships in 45-dimensional space. There isn't room. So you project up to 512 dimensions, a much larger working space, where the network can start detecting combinations and conjunctions across the raw inputs. Think of it as scratch paper. The 512 numbers in $h_1$ aren't sensor readings anymore; they're intermediate computations the network learned to find useful.

**Step 2: $h_2 = f(W_2 h_1 + b_2)$: 512 → 512**

One pass isn't enough. $h_1$ found low-level combinations ("left leg is behind right leg, torso tilting forward"). $h_2$ builds on those to find higher-level structure ("I'm mid-stride on the left leg and need to start pushing off"). Each layer composes the previous layer's features into more abstract ones.

This is the literal meaning of "deep" in deep learning.  depth = layers of abstraction stacked on each other.

**Step 3: $a_t = W_3 h_2 + b_3$: 512 → 12, no activation**

Now compress all of that processed information back down to 12 numbers: one target joint angle per joint. No activation function on this final layer — activations like ELU squash values, but joint angles are real numbers with no natural bounds. You want the raw linear output.

Those 12 numbers are $q_{\text{target}}$. Hand them to the PD controller. That's the full forward pass.

**A note on weights:**

> You might be wondering how exactly $a_{t}$ is just pure joint values. Let's go back all the way to [Multilayer-perceptrons](Multilayer-perceptrons.md), where the idea is basically, let's randomly initialize the weights for this function, throw a loss function on it, run it against real world i/o (in the case of supervised learning), and watch the weights get **optimized**. So we NEVER EVER touch weights, we ALWAYS let them figure themselves out. The idea with $a_{t}$ is that the robot will fail so much that at some point, it'll align itself well enough such that $a_{t}$ is not only the right value of joint values, but it's a joint value at all.

---

## What $\theta$ Is

Every $W$ and $b$ across all three layers, flattened into one long vector:

$$\theta = [\,\text{vec}(W_1),\ b_1,\ \text{vec}(W_2),\ b_2,\ \text{vec}(W_3),\ b_3\,]$$

Count the parameters: $45 \times 512 + 512 + 512 \times 512 + 512 + 512 \times 12 + 12 = 290{,}828$ numbers.

Recall every single one starts random. The policy outputs garbage. The robot falls immediately.

Training the robot means finding the values of $\theta$ that make those 290k numbers produce actions that lead to walking.

---

## The Question Training Has to Answer

You have a robot. It's falling. You have a network with 290k parameters. How do you know which parameters to change, and by how much, to make it walk?

You need some way to *score* how good the robot's behavior is, and then propagate that score back through $\theta$ to nudge it in the right direction.

That's where at LAST the [Value Function Loss Function](Value%20Function%20Loss%20Function/Value%20Function%20Loss%20Function.md) comes in.


---
**A note on Teacher-Student Setups**
The
  - Teacher takes $(s_t, e_t)$, the 45 sensor readings plus privileged sim data
  - Student takes just $s_t$, the 45 sensor readings, same as what [Policy](Policy.md) describes

  So when you read [Policy](Policy.md)'s forward pass, that is the student. The teacher is the same
  network with a wider first layer to accommodate $e_t$.

---
