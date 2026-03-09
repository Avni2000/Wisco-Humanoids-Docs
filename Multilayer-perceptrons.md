## A Single Linear Layer

Start simple. You have a vector of numbers, your input $x \in \mathbb{R}^n$. You want to transform it into a different vector $y \in \mathbb{R}^m$. All a model really is, is the transformation from $x$ to $y$.  

This is akin to the equation $$
y=mx+b
$$
This is also a model. Given an input $x$, it'll try and match it to the point of best fit $y$. But what if our $x$ is really complex? Imagine it's a picture of a plant and we want to classify what kind of plant it is. The picture could be in a really high resolution, so $x$ might actually be a matrix, not just a scalar.

A linear layer does exactly this:

$$y = Wx + b$$

- $W \in \mathbb{R}^{m \times n}$ is a **weight matrix**. Each row is a set of coefficients that computes one output number as a weighted sum of all input numbers.
- $b \in \mathbb{R}^m$ is a **bias vector**. It shifts each output up or down by a fixed amount.

That's it. A linear layer is just a learned weighted sum. Every output depends on every input, with learned weights controlling how much.


---

## The curse of linearity:

Stack two linear layers:

$$y = W_2(W_1 x + b_1) + b_2 = \underbrace{(W_2 W_1)}_{\tilde{W}} x + \underbrace{(W_2 b_1 + b_2)}_{\tilde{b}}$$

That collapses to a single linear layer $\tilde{W}x + \tilde{b}$. No matter how many linear layers you stack, you still just have one linear map. You've gained nothing.

### Adding nonlinearity:

Between every two linear layers, apply a nonlinear function $f$ elementwise. A common choice is **ELU**:

$$f(z) = \begin{cases} z & z > 0 \\ e^z - 1 & z \leq 0 \end{cases}$$

Positive inputs pass through unchanged. Negative inputs get squashed toward $-1$ instead of blowing up and changing the model drastically.

Now the composition no longer collapses. Each layer can extract genuinely different structure from the data. 

There is a reason deep learning is called deep learning. It allows us to **stack** layers on top of each other. Stack enough of these and the network can approximate arbitrarily complex functions, including the mapping from "robot sensor readings" to "good joint targets."
