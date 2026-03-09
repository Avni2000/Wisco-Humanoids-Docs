> FYI this is 100% AI generated, literally have barely read it. Don't put all your trust in it, but it's a really good way to keep everything in your head. 


**The three networks:**

|Name|Notation|Input|Output|Trained by|
|---|---|---|---|---|
|Teacher policy|$\pi_\theta$|$s_t, e_t$|12 joint angles|RL (policy loss)|
|Critic|$V_\phi$|$s_t, e_t$|1 scalar|RL (value loss)|
|Student policy|$\pi_\text{student}$|$s_t$ only|12 joint angles|Imitation (MSE against teacher)|

The critic is discarded after teacher training. Only the student gets deployed.

---

**The variables:**

|Symbol|What it is|
|---|---|
|$s_t$|Robot's sensor state at time $t$ — 45 numbers|
|$e_t$|Privileged sim-only info — friction, terrain, contact forces|
|$q_\text{target}$|Target joint angles output by a policy — 12 numbers|
|$\tau$|Torques — what the PD controller converts $q_\text{target}$ into|
|$\theta$|All weights inside the teacher policy|
|$\phi$|All weights inside the critic|
|$R_t$|Actual rewards collected from timestep $t$ onward|
|$A_t$|Advantage — was $R_t$ better or worse than $V$ predicted?|

---

**The flow in one line:**

$$\underbrace{\pi_\theta \text{ + } V_\phi \text{ trained together via RL}}_{\text{teacher phase}} \longrightarrow \underbrace{\pi_\text{student} \text{ mimics } \pi_\theta}_{\text{student phase}} \longrightarrow \text{deploy } \pi_\text{student}$$