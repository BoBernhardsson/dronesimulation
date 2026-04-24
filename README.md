# Drone Height Control Game

A single-file browser-based simulation for teaching **manual control**, **PID control**, and **feedforward compensation** on a simplified drone height-control problem.

The app is designed to run as a static webpage, for example through **GitHub Pages**.

## Overview

The simulation focuses on **vertical motion only**. The user either:

- flies the drone manually by adjusting thrust, or
- tunes a **PID controller with a feedforward term** to make the drone follow a height reference.

The interface shows:

- a **drone visualization** on the left,
- streaming plots on the right for:
  - **height and reference**,
  - **control signal**,
- compact controls for switching mode and tuning controller parameters,
- a score based on tracking performance.

## Model

The drone is modeled as

```math
\ddot{x}(t) = k(t)u(t) - m(t)g
```

where:

- `x(t)` is height,
- `u(t)` is the control input / thrust command,
- `g = 9.81 m/s²`,
- `k(t)` is a time-varying input gain,
- `m(t)` is a time-varying mass.

### Time-varying plant effects

To make the control task realistic and educational, the plant changes during the simulation:

- **Battery fade:** `k(t)` slowly decreases over time, meaning the same input gives less acceleration later in the run.
- **Cargo drop:** at **15 seconds**, the mass drops by **30%**, creating an abrupt plant change.

These effects are useful for discussing robustness, feedforward mismatch, disturbance rejection, and integral action.

## Control modes

### 1. Manual mode

In manual mode, the user directly controls the thrust command `u`.

This is useful for showing:

- how hard it is to track a reference manually,
- why steady thrust is needed just to hover,
- how plant variations make manual control difficult.

### 2. PID + feedforward mode

In automatic mode, the control law is split into two parts:

```math
u =  u_{PID} + u_{ff}
```

where:

```math
u_{PID} =  K \left(e + \frac{1}{T_i}\int e\,dt + T_d \frac{de_f}{dt}\right), \qquad e_f = LP(r - y)
```

Here:

- `r` is the height reference,
- `y` is the measured height,
- `e` is the tracking error,
- `u_ff` is a constant feedforward term selected by the user.

### Role of the feedforward term

The feedforward term is intended to provide most of the control effort needed for hover.

For a nominal hover condition,

```math
u_{ff} \approx \frac{m g}{k}
```

If feedforward is chosen well, the PID controller only has to correct for:

- transient tracking errors,
- model mismatch,
- battery fade,
- the cargo drop.

This makes the control action easier to interpret pedagogically:

- **feedforward** handles the known baseline demand,
- **feedback** handles what the model does not predict exactly.

## PID tuning parameters

The simulation exposes the following controller parameters:

- `K`: proportional gain
- `Ti`: integral time
- `Td`: derivative time
- `FF`: feedforward level (hovering-bias approximation)

These can be changed live during the run.

## Anti-windup

Yes — the current implementation includes a **simple anti-windup mechanism**.

It is **not** a full back-calculation scheme, but it does prevent the integral state from growing in the wrong situations.

The controller first computes a tentative integral update and forms the unsaturated command. Then:

- if the command is **not saturated**, the integral update is accepted,
- if the command **is saturated**, the integral is only allowed to update when the error would help drive the controller **back out of saturation**.

In other words, the integral state is blocked when it would worsen saturation, and allowed when it would help recover from it.

This is a standard and useful **conditional integration** anti-windup strategy.

## Scoring

The score rewards accurate and smooth tracking.

The current score penalizes:

- squared tracking error,
- control effort,
- rapid control changes.

This means a good score requires more than simply reaching the reference:

- low overshoot helps,
- persistent oscillation hurts,
- unnecessarily aggressive control also hurts.

## Reference profiles

The app includes multiple built-in reference trajectories:

- **Steps**
- **Sine**
- **Challenge**

These allow the same controller to be tested on different tracking tasks.

## Classroom use

This simulation is intended for teaching topics such as:

- manual versus automatic control,
- equilibrium and hover thrust,
- feedback versus feedforward,
- PID tuning,
- actuator saturation,
- integral action,
- robustness to plant changes.

A useful classroom sequence is:

1. Start in **manual mode**.
2. Ask students to track a changing reference.
3. Switch to **PID + FF** mode.
4. Set feedforward near the hover value.
5. Tune `K`,  `Td`, and `Ti`.
6. Discuss the impact of battery fade and the cargo drop.

## Running locally

Because the app is a single HTML file, you can run it by simply opening it in a browser.

For GitHub Pages, place the file in a repository as:

- `index.html`

and enable GitHub Pages for the branch.

## Possible future extensions

Ideas for further development:

- disturbance wind gusts,
- noise on the height measurement,
- explicit hover operating-point display,
- level-based challenges,
- saved best score,
- selectable anti-windup methods,
- automatic feedforward based on nominal model values.
