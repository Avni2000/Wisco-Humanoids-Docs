This is given by $s^t$ - state at timestep $t$ in plain english. This would seem to imply that we have states at $$
T=0, 1, 2\dots t
$$
at least.

## What it actually is

It's a list of numbers describing everything the robot can currently sense. You take all those sensor readings and concatenate them into one long vector.

**Note:**

A bipedal robot has 2 legs. Each leg has:

- Hip: 3 joints (it can swing forward/back, side to side, and rotate)
- Knee: 1 joint (only bends one way)
- Ankle: 2 joints (tilt forward/back, side to side)

That's 6 per leg × 2 legs = 12 joints to play with.

| Sensor            | What it measures                                                                                                                                                                                   | Size                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| $q$               | joint angles right now                                                                                                                                                                             | $\mathbb{R}^{12}$                            |
| $\dot{q}$         | joint velocities right now                                                                                                                                                                         | $\mathbb{R}^{12}$                            |
| $\omega$          | how fast the body is rotating (IMU)\*                                                                                                                                                              | $\mathbb{R}^{3}$ -- "rotation" in x,y,z axis |
| $g_{\text{proj}}$ | which way is down (IMU)\*                                                                                                                                                                          | $\mathbb{R}^{3}$ -- "down" in x,y,z axis     |
| $v_{\text{cmd}}$  | Speed you're commanding<br><br>You're telling the robot to move at some velocity. That velocity has 3 components -- forward/back, left/right, and rotation rate. So 3 numbers, considering pos/neg | $\mathbb{R}^{3}$                             |
| $a_{t-1}$         | What action you took last step. The last action was 12 target joint positions. So storing it is also 12 numbers.                                                                                   | $\mathbb{R}^{12}$                            |

>\*IMU stands for *Inertial measurement unit*

Stack all of those into one vector:

$$s_t = [q,\ \dot{q},\ \omega,\ g_{\text{proj}},\ v_{\text{cmd}},\ a_{t-1}] \in \mathbb{R}^{45}$$

That's the input to the policy network. One flat list of 45 numbers. The network doesn't know which numbers are joint angles vs IMU readings, it just sees 45 numbers and learns what to do with them.
