This is given by $s^t$ - state at timestep $t$ in plain english. This would seem to imply that we have states at 
$$
T=0, 1, 2\dots t
$$
at least.

## What it actually is

It's a list of numbers describing everything the robot can currently sense. You take all those sensor readings and concatenate them into one long vector.

**Note:**

A bipedal robot has 2 legs. Each leg has:

- Hip: 3 joints (it can swing forward/back, side to side, and rotate)
	- - **"swing forward/back"** → hip flexion/extension (HFE) (pitch)
		- ![Pitchl](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ec/Aileron_pitch.gif/500px-Aileron_pitch.gif)
	- **"side to side"** → hip abduction/adduction (HAA) (roll)
		- ![Roll](https://upload.wikimedia.org/wikipedia/commons/c/cc/Aileron_roll.gif)
	- **"rotate"** → hip rotation (HR) (yaw)
		-  ![Yaw visualization](https://upload.wikimedia.org/wikipedia/commons/9/96/Aileron_yaw.gif)
 > More specifically, the DoFs for the hip include hip rotation (HR), hip flexion and extension (HFE), and hip abduction and adduction (HAA). - https://arxiv.org/html/2409.19795v2 


- Knee: 1 joint (only bends one way)

- Ankle: 2 joints (tilt forward/back, side to side)

>For more general applications, robots with a full humanoid body plan have been built. Prominent examples of humanoids such as ASIMO, the HRP series, HUBO, REEM-C, and TALOS are all similar in terms of kinematics. **The legs are composed of a three-degree-of-freedom (DoF) hip to simulate a spherical joint, one DoF for knee bending and two DoF for an ankle ball joint. As such, six actuators are enough to provide roughly the same form and functionality as a human leg.** - https://link.springer.com/article/10.1007/s43154-021-00050-9


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
This exact structure appears in published work. 

> The bipedal locomotion policy receives observations which include the proprioceptive information, locomotion command, and the last action $a_{t-1} \in \mathbb{R}^{12}$. Proprioceptive information includes joint position $\theta_t \in \mathbb{R}^{12}$ and joint velocity $\dot{\theta}_t \in \mathbb{R}^{12}$ provided by the joint encoders and projected gravity in the robot frame $g_t \in \mathbb{R}^3$ from the IMU. [_MERL TR2025-140_ ](https://www.merl.com/publications/docs/TR2025-140.pdf)

That's the input to the policy network. One flat list of 45 numbers. The network doesn't know which numbers are joint angles vs IMU readings, it just sees 45 numbers and learns what to do with them.

