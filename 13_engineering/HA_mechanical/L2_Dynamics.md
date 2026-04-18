# Level 2: Dynamics
*Motion, acceleration, vibration, and the forces that cause them*

<!-- Evidence Tier: Textbook -->

## When Things Move

Statics assumed nothing accelerates. Dynamics handles what happens when that assumption fails — which is most real mechanical engineering. A car braking, an airplane banking, a piston moving in an engine, a rotating shaft, a vibrating machine, a rocket accelerating — all are dynamics problems.

Two ancient equations run most of dynamics:

- **Newton's second law**: $\sum \vec{F} = m \vec{a}$. The net force on a body equals its mass times acceleration.
- **Euler's equation for rotation**: $\sum \vec{M} = I \vec{\alpha}$ (for fixed axis). The net moment equals rotational inertia times angular acceleration.

From these plus energy conservation and momentum conservation, you derive everything — vehicle dynamics, vibration theory, rotating machinery, impact mechanics, orbital mechanics.

## Kinematics: The Description of Motion

Before we ask what causes motion, we describe it:

- **Position**: $\vec{r}(t)$, location as a function of time.
- **Velocity**: $\vec{v} = d\vec{r}/dt$, rate of change of position.
- **Acceleration**: $\vec{a} = d\vec{v}/dt = d^2\vec{r}/dt^2$.

For rotational motion:
- **Angular position**: $\theta(t)$.
- **Angular velocity**: $\omega = d\theta/dt$.
- **Angular acceleration**: $\alpha = d\omega/dt$.

For constant acceleration, standard equations apply: $v = v_0 + at$, $x = x_0 + v_0 t + \frac{1}{2}at^2$, and $v^2 = v_0^2 + 2a(x - x_0)$. Similar equations for rotation.

When motion is more complex — variable acceleration, curved paths, multiple reference frames — the description gets harder, but the procedure stays the same: write position as a function of time (or angle, or whatever parameter is convenient), differentiate to get velocity and acceleration.

**Curvilinear motion** — motion along a curved path — decomposes into tangential acceleration (changing speed) and centripetal acceleration (changing direction). A car rounding a curve at constant speed has zero tangential acceleration but nonzero centripetal acceleration pointing toward the center of the curve: $a_n = v^2/r$. That's why you feel pressed against the door.

## Dynamics of a Particle

For a point mass, Newton's second law directly governs motion:

$\sum \vec{F} = m \vec{a}$

Set up the free body diagram (same as statics — forces on the object). Write the equation in each direction. Solve for unknowns.

**Friction** is often the key force in dynamics problems. Kinetic friction opposes motion: $f_k = \mu_k N$. Static friction prevents motion up to $\mu_s N$; when the applied force exceeds this, the object starts to slide.

**Air resistance** (drag) becomes important at higher speeds. At low speeds (Reynolds numbers), drag is linear in velocity ($F_d = bv$). At higher speeds (most engineering situations), drag is quadratic ($F_d = \frac{1}{2}\rho C_d A v^2$). A skydiver, after falling a few seconds, reaches terminal velocity when weight and drag balance.

Projectile motion, inclined planes, Atwood machines, and countless textbook problems all reduce to careful application of $\vec{F} = m\vec{a}$ with appropriate force enumeration.

## Work, Energy, Power

Forces do **work** when they cause motion: $W = \vec{F} \cdot d\vec{r}$. The work-energy theorem says that net work on a body equals change in kinetic energy:

$W_{net} = \Delta KE = \frac{1}{2}m v^2 - \frac{1}{2}m v_0^2$

For many problems, energy methods are easier than direct force methods. A block sliding down a frictionless incline — instead of resolving forces and integrating acceleration, equate loss of potential energy to gain of kinetic energy.

**Conservative forces** (gravity, spring, electric) have associated potential energies. **Nonconservative forces** (friction, air resistance) dissipate energy. For a closed system, total mechanical energy is conserved among conservative forces but decreases when nonconservative forces act.

**Power** is the rate of doing work: $P = dW/dt = \vec{F} \cdot \vec{v}$. Engine ratings are in power. A 200-hp car engine delivers 200 hp peak — about 150 kW. Lifting 1000 kg up 10 m in 10 seconds requires 9.8 kW — well within the engine's capacity, which is why cars accelerate fast.

## Momentum and Impulse

**Linear momentum**: $\vec{p} = m\vec{v}$. Newton's second law can be written as $\sum \vec{F} = d\vec{p}/dt$. Conservation of momentum: for a closed system with no external forces, total momentum is conserved. This is one of the deepest results in physics.

**Impulse**: $\vec{J} = \int \vec{F} dt = \Delta \vec{p}$. A large force applied briefly or a smaller force applied longer can produce the same impulse and the same momentum change.

**Collisions** are applications:
- **Elastic collision**: both momentum and kinetic energy conserved. Billiard balls approximate.
- **Inelastic collision**: momentum conserved, kinetic energy partially lost. Everyday collisions.
- **Perfectly inelastic**: bodies stick together. Maximum energy lost.

Car crash safety engineering is impulse management. For a given momentum change (speed change on impact), longer deceleration time (crumple zones, airbags) means lower force, which means lower injury risk. A car designed to crumple at lower force but over longer distance is safer than a rigid car stopping suddenly, even though the passenger experiences the same total momentum change.

## Rotational Dynamics

For a rigid body rotating about a fixed axis:

$\sum \tau = I \alpha$

where $I$ is the moment of inertia (rotational mass) about that axis. $I = \int r^2 dm$ — mass farther from the axis contributes more.

Important moment-of-inertia formulas:
- Solid cylinder about its axis: $I = \frac{1}{2}MR^2$.
- Hollow cylinder (thin shell): $I = MR^2$.
- Solid sphere: $I = \frac{2}{5}MR^2$.
- Thin rod about center: $I = \frac{1}{12}ML^2$.
- Thin rod about end: $I = \frac{1}{3}ML^2$.

**Parallel axis theorem**: if $I_{cm}$ is moment of inertia about center of mass, moment of inertia about a parallel axis distance $d$ away is $I = I_{cm} + Md^2$.

**Rotational kinetic energy**: $\frac{1}{2}I\omega^2$.

**Angular momentum**: $\vec{L} = I\vec{\omega}$ for a body rotating about a symmetry axis; more generally, $\vec{L} = \vec{r} \times \vec{p}$. Conservation of angular momentum for systems with no external torques. An ice skater spinning arms-out has more moment of inertia and lower angular velocity; tucking arms in decreases $I$ and increases $\omega$ to conserve $L$.

Rotating machines — turbines, compressors, generators, engines, rotor craft, flywheels — are all analyzed with these equations.

## Vibration

Vibration is oscillation about an equilibrium. Understanding it is essential for machinery design, structural design, earthquake engineering, and aerospace.

**Simple harmonic motion**: a mass on a spring, restoring force proportional to displacement:

$m\ddot{x} + kx = 0$

Solution: $x(t) = A \cos(\omega_n t + \phi)$, where $\omega_n = \sqrt{k/m}$ is the **natural frequency**. Any system with a quadratic potential well behaves this way for small oscillations.

**Damped oscillation**: adding viscous damping ($c\dot{x}$):

$m\ddot{x} + c\dot{x} + kx = 0$

Three regimes depending on damping ratio $\zeta = c / (2\sqrt{mk})$:
- **Underdamped** ($\zeta < 1$): oscillates with decreasing amplitude.
- **Critically damped** ($\zeta = 1$): returns to equilibrium fastest without oscillating.
- **Overdamped** ($\zeta > 1$): returns slowly without oscillating.

Automotive shock absorbers target critical damping or slightly above — so the car returns to level quickly after hitting a bump without oscillating.

**Forced oscillation**: apply a harmonic force at frequency $\omega$:

$m\ddot{x} + c\dot{x} + kx = F_0 \cos(\omega t)$

The steady-state response has amplitude that depends strongly on $\omega/\omega_n$:
- Far from natural frequency: modest response.
- At natural frequency (**resonance**): large response, limited only by damping.

Resonance can be useful (musical instruments, radio tuning, vibration sensors) or catastrophic (Tacoma Narrows Bridge collapse in 1940; various structural failures; machinery destroyed by running at resonance speed). Good machine design avoids operating at or near resonance of any significant structural mode.

**Multi-degree-of-freedom systems** have multiple natural frequencies (modes), each a characteristic pattern of vibration. Real structures (buildings, aircraft wings, engine crankshafts) have hundreds or thousands of modes. Modal analysis — either experimental (testing) or computational (finite element) — identifies and characterizes each, enabling engineers to design around problematic modes.

## Rotating Machinery

Many mechanical systems involve rotating parts — engines, turbines, compressors, pumps, motors, gearboxes. Specific dynamics issues:

**Rotor balancing**: even tiny imbalances in a rotating part create centrifugal forces proportional to $m r \omega^2$. At high speeds, small imbalances produce large forces, causing vibration, noise, and bearing wear. Precision balancing (measuring and correcting imbalance) is routine for aerospace turbines, machine tool spindles, automotive rotating parts.

**Critical speeds**: rotating shafts have natural bending frequencies. Running a shaft at or near these critical speeds causes large bending deflections — shaft whip — and possible failure. Designers either run below the first critical speed (subcritical operation) or accelerate quickly through critical speeds into supercritical operation.

**Gyroscopic effects**: rotating masses resist changes in their rotation axis. A rapidly spinning rotor in an aircraft or helicopter produces gyroscopic moments that must be accounted for in control design. Motorcycles leaning into turns use gyroscopic effects from wheel rotation to stabilize.

**Bearings**: support rotating shafts. Ball bearings, roller bearings, journal bearings, and fluid-film bearings each have speed, load, and precision characteristics. Bearing life is usually the limit on rotating machinery reliability.

## Vehicle Dynamics

Cars, trucks, and other vehicles are dynamic systems with specific characteristics:

**Handling**: how a vehicle responds to steering inputs. A well-designed vehicle has good **directional stability** (returns to straight after disturbance), reasonable **responsiveness** (turns when asked), and predictable behavior at the limit. Poor handling (excessive oversteer, understeer that doesn't recover, instability at high speeds) is a safety hazard.

**Suspension**: springs and dampers isolate the body from road irregularities while keeping tires in contact with the ground. Design is a balance — stiff enough for handling, soft enough for comfort, damped enough to not oscillate, allowing enough wheel travel for large bumps.

**Tires**: the only contact with the road. Tire-road friction limits acceleration, braking, and cornering. Modern tire design is a sophisticated materials and mechanics problem balancing wet grip, dry grip, rolling resistance, durability, and noise.

**Active systems**: stability control, anti-lock braking, traction control, adaptive suspension, autonomous driving — all dynamics problems where sensors, computers, and actuators augment the vehicle's inherent dynamics. ESC (electronic stability control) is estimated to have prevented tens of thousands of fatalities since becoming common.

## Multibody Dynamics

Complex mechanical systems — robots, vehicles with suspensions, linkages, rotorcraft — have many interconnected parts. **Multibody dynamics** computes the motion of all parts simultaneously.

Modern software (ADAMS, Simpack, MSC Adams, Ansys Motion, and others) handles thousands of rigid and flexible bodies with joints and forces. Input: geometry, mass properties, joint types, applied forces and motions. Output: full motion history, internal forces, stresses.

Multibody simulation is essential for:
- **Vehicle design**: ride and handling development, crash simulation.
- **Robotics**: dynamics of multi-link robots, control design.
- **Aerospace**: landing gear, deployment mechanisms.
- **Biomechanics**: human and animal motion studies.
- **Machine design**: linkage synthesis, gearbox dynamics.

Doing it without software was impractical for anything beyond simple two- or three-body systems; today it's routine for systems with hundreds of bodies.

## Why This Level Matters

Dynamics is where mechanical engineering graduates the static abstractions of statics and confronts the time-dependent reality of everything that moves. Every vehicle, machine, robotic system, and consumer device that moves is engineered using dynamics.

Beyond engineering, dynamics is the foundation for classical physics, orbital mechanics (satellites, spacecraft), biomechanics (human and animal motion), and many modeling problems elsewhere. A person who can formulate and solve dynamics problems has a transferable toolkit.

## The Transition to Level 3

L3 turns to **mechanics of materials** — how forces produce stress and strain inside materials, how materials respond under static and dynamic loads, and how this understanding translates into component design.

Next: [L3 — Mechanics of Materials](./L3_Mechanics_of_Materials.md) *(deferred)*
