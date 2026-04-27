# Drone Height Control Game

A single-file browser-based simulation for teaching **manual control**, **PID control**, **feedforward compensation**, and practical tuning of a simplified drone height-control system.

The app runs entirely in the browser as one HTML file and is suitable for hosting with **GitHub Pages**. The current public version is available at:

https://bobernhardsson.github.io/dronesimulation/

---

## Overview

The simulation focuses on **vertical motion only**. The user can either:

- fly the drone manually by adjusting thrust, or
- tune a **PID controller with feedforward** to track changing height references.

The interface contains:

- a drone visualization,
- live plots of:
  - height and reference,
  - control signal,
- real-time controller sliders,
- score calculation based on staying within an allowed tracking corridor.

---

## Dynamic Model

The drone is modeled as:

```math
\ddot{x}(t) = k(t)\,u(t) - m(t)\,g
```

where:

- `x(t)` = height  
- `u(t)` = thrust command  
- `g = 9.81 m/s²`
- `k(t)` = thrust effectiveness
- `m(t)` = drone mass

---

## Plant Variations During Simulation

To make the control problem realistic, the plant changes during flight:

### Battery Fade

The gain `k(t)` slowly decreases over time.

This means the same control signal gives less lift later in the simulation.

### Cargo Drop

At **45 seconds**, the mass drops by **30%**.

This creates a sudden plant change and tests controller robustness.

---

## Control Modes

## 1. Manual Control

The user directly controls thrust.

This demonstrates:

- difficulty of manual tracking,
- need for hover thrust,
- effect of changing plant parameters.

Keyboard control is supported.

---

## 2. PID + Feedforward Control

The controller is:

```math
u = u_{ff} + u_{PID}
```

with

```math
u_{PID}
=
K\left[
(br-y)
+
\frac{1}{T_i}\int(r-y)\,dt
-
T_d \hat{\dot y}
\right]
```

where:

- `r` = reference height
- `y` = measured height
- `b` = proportional setpoint weight
- `K` = controller gain
- `Ti` = integral time
- `Td` = derivative time
- `ŷdot` = low-pass filtered vertical velocity
- `u_ff` = feedforward hover bias

---

## Why Use `b`

The proportional term uses:

```math
br - y
```

instead of:

```math
r - y
```

This reduces overshoot after step changes.

Typical values:

- `b = 1.0` → normal proportional action  
- `b = 0.5` → softer setpoint response  
- `b = 0.0` → no proportional response to reference jumps

This is standard **setpoint weighting**.

---

## Feedforward Term

The feedforward term supplies most of the thrust needed to hover:

```math
u_{ff} \approx \frac{mg}{k}
```

Then PID only corrects:

- transients
- battery fade
- cargo drop
- modeling mismatch

---

## Derivative Filtering

The derivative part is based on measured motion, **not reference error**.

This avoids derivative kick when the reference changes suddenly.

A low-pass filter is used:

```math
\hat{\dot y} = LPF(\dot y)
```

This gives smoother control signals.

---

## Anti-Windup

The integral term uses **conditional integration** anti-windup.

If actuator saturation occurs:

- the integrator is frozen when it would worsen saturation
- it is allowed to move when it helps recover

This prevents runaway integral buildup.

---

## Tuning Parameters

The player can tune:

- `K`
- `Ti`
- `Td`
- `b`
- `FF`

Typical starting values:

- `K = 1`
- `Ti = 1`
- `Td = 1`
- `b = 1`

---

## Reference Profiles

Built-in references include:

- **Steps**
- **Sine**
- **Challenge**

The default game mode uses multiple step changes.

---

## Scoring System

The score no longer penalizes control effort.

Instead, the drone must stay inside a **tracking corridor** around the reference.

### Corridor Width

```math
r \pm 0.4
```

### Smart Delays

To avoid unfair penalties:

- when reference increases, the **lower boundary waits 1.5 seconds**
- when reference decreases, the **upper boundary waits 1.5 seconds**

This rewards practical tracking rather than impossible instant jumps.

### Score Meaning

- Stay inside corridor → high score
- Leave corridor → penalty accumulates
- Fast but wild control gives no direct penalty

---

## Why Overshoot Can Occur

For this drone system (approximately a double integrator), some overshoot after step changes is natural when integral action is used.

This makes the simulator useful for discussing:

- transient design
- damping
- setpoint weighting
- tradeoff between speed and overshoot

---

## Classroom Use

Useful for teaching:

- manual vs automatic control
- PID tuning
- feedforward vs feedback
- anti-windup
- robustness
- setpoint weighting (`b`)
- actuator saturation
- double-integrator dynamics

Suggested exercise:

1. Try manual mode  
2. Tune hover feedforward  
3. Add proportional action  
4. Add derivative damping  
5. Add integral correction  
6. Tune `b` to reduce overshoot  
7. Observe battery fade and cargo drop

---

## Running Locally

Open `index.html` in any modern browser.

---

## GitHub Pages

Host by placing `index.html` in the repository root and enabling GitHub Pages.

---

## Possible Future Extensions

- wind disturbances
- measurement noise
- sensor delay
- leaderboard
- saved best scores
- auto-tuning challenge
- 2D drone motion
- Kalman filter mode
