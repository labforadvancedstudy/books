# Level 1: Aerodynamics
*Lift, drag, airfoils, and the Reynolds number that organizes everything*

<!-- Evidence Tier: Textbook -->

## Four Forces

Every aircraft in flight has four forces acting on it:

- **Lift** (upward, perpendicular to flow): produced by the wings, opposes weight.
- **Weight** (downward): gravity acting on aircraft mass.
- **Thrust** (forward): from engines, opposes drag.
- **Drag** (backward): resistance from the air.

In steady level flight, lift = weight and thrust = drag. Climb increases required lift or decreases required drag (or both). Descent is the opposite. Every maneuver adjusts the balance.

Designers' central problem: maximize lift while minimizing drag, at the speeds and altitudes where the aircraft needs to operate, within weight and structural constraints.

## The Airfoil

An **airfoil** is the cross-sectional shape of a wing. Classic airfoils have a rounded leading edge, a sharp trailing edge, a curved upper surface, and a flatter lower surface.

When an airfoil moves through air (or air flows past a stationary airfoil — same thing relatively), the curvature deflects the flow. The deflection produces two linked effects:

- Pressure differences: lower on top, higher on bottom. The pressure difference integrates to a net upward force per unit area.
- Momentum change: air is deflected downward behind the wing. Newton's third law: the air pushes the wing up.

These are not alternative explanations. They are the same phenomenon seen from two angles. The pressure and velocity fields are related by the fluid equations (Euler for inviscid flow, Navier-Stokes with viscosity). Solve either and you get the same lift.

The **angle of attack** is the angle between the airfoil's chord line and the oncoming flow. Lift rises roughly linearly with angle of attack up to a limit — typically around 15° for a classical airfoil — then drops precipitously as the flow separates from the upper surface. That is **stall**.

## Lift Coefficient and the Lift Equation

The lift produced by a wing is given by:

$L = \frac{1}{2} \rho V^2 S C_L$

where $\rho$ is air density, $V$ is airspeed, $S$ is wing area, and $C_L$ is the lift coefficient (dimensionless, a function of angle of attack and airfoil shape).

A typical passenger aircraft cruises at $C_L \approx 0.5$, takes off and lands at $C_L \approx 1.5$ with flaps extended. Maximum $C_L$ before stall is typically 1.5–2.0 for classic airfoils, higher with advanced high-lift devices.

For a given weight and wing area, cruise speed and air density set each other. At high altitude (low density), you must fly faster to generate the same lift. This is why airliners cruise at 10+ km altitude at ~80% of the speed of sound — the density and speed combination minimizes fuel burn.

## Drag — The Enemy

Drag comes from several physically distinct sources:

- **Skin friction drag**: from viscous shear at the surface. Dominates at low speeds and for streamlined bodies.
- **Pressure (form) drag**: from pressure differences between front and back of the body. Dominates for blunt bodies.
- **Induced drag**: a price paid for generating lift. The wing produces downwash, which tilts the effective lift vector backward, producing a drag component.
- **Wave drag**: appears at near-sonic speeds and above, from shock waves.
- **Interference drag**: from flow interactions at junctions (wing-fuselage, engine-wing).

Induced drag falls with increasing speed; form drag rises. The minimum total drag occurs at a specific speed — the aircraft's best-glide speed. Cruise speed is usually a bit above this, optimized for speed/fuel tradeoff rather than pure range.

Induced drag also falls with increasing **aspect ratio** (wingspan²/wing area). That's why gliders have long narrow wings (aspect ratios 20–40) — they prioritize maximum range per unit altitude lost. Fighters have low aspect ratios (3–4) because they need high roll rates and structural strength. Airliners sit in the middle (8–12).

## The Reynolds Number

The Reynolds number ($Re$) is a dimensionless ratio of inertial to viscous forces:

$Re = \frac{\rho V L}{\mu}$

where $L$ is a characteristic length (chord length of a wing, for example) and $\mu$ is the dynamic viscosity.

Reynolds number organizes aerodynamic regimes:

- $Re \lesssim 10^3$: viscous forces dominate. Flow is laminar and stable. Insects, raindrops, particles.
- $Re \sim 10^3$ to $10^5$: transitional. Laminar flow becomes unstable and transitions to turbulence under disturbances. Model aircraft, small birds.
- $Re \sim 10^5$ to $10^7$: fully turbulent boundary layers are the rule. Large birds, gliders, small aircraft.
- $Re \sim 10^7$ and up: turbulent throughout, high-Re aerodynamics. Commercial jets, missiles.

The same airfoil shape behaves very differently at different Reynolds numbers. A shape optimized for $Re = 10^5$ (small model) will not perform the same at $Re = 10^7$ (airliner). This is why wind-tunnel testing of scale models can mislead — the flow regime changes.

## The Boundary Layer

Air molecules immediately next to a surface are at rest (the no-slip condition). Away from the surface, they're moving at the freestream velocity. Between them is a thin layer where velocity rises from zero to freestream — the **boundary layer**.

A boundary layer starts laminar at the leading edge and eventually transitions to turbulent. Laminar boundary layers have lower skin friction but are more prone to separation under adverse pressure gradients. Turbulent boundary layers have higher friction but stick better to curved surfaces.

Managing boundary layers is a big part of high-performance aerodynamics. Laminar flow airfoils try to delay transition to reduce friction. Vortex generators intentionally trip boundary layers into turbulence to prevent separation. Suction slots remove boundary layer material to energize flow.

Aircraft ice accretion ruins aerodynamics by disturbing the boundary layer — even thin ice layers can dramatically increase drag and reduce stall angle. Ice protection systems (heated leading edges, pneumatic boots, running-wet anti-icing) are critical safety equipment.

## Subsonic, Transonic, Supersonic

**Subsonic** flow (Mach < ~0.7): air behaves essentially as incompressible for aerodynamic purposes. Standard lift and drag analysis apply.

**Transonic** flow (~0.7 < Mach < 1.2): local Mach numbers on parts of the aircraft exceed 1.0, producing shock waves that add wave drag and change lift characteristics dramatically. The "sound barrier" of the 1940s was the transonic drag rise that limited early jets until swept wings and area-ruled fuselages addressed it.

**Supersonic** flow (Mach > 1): shock waves organize the flow. Different airfoils (thin, sharp-leading-edged diamonds) work better than subsonic airfoils. The Concorde and military fighters are designed for this regime.

**Hypersonic** (Mach > 5): heating becomes dominant. Aerothermal effects at leading edges produce temperatures that melt ordinary materials. Special heat-resistant materials and active cooling required. Re-entry capsules, hypersonic missiles, and research vehicles operate here.

Modern commercial airliners cruise just below the transonic regime (Mach 0.78–0.86), balancing speed, fuel efficiency, and the drag penalty of approaching Mach 1.

## High-Lift Devices

Takeoff and landing require high $C_L$ at low speeds. Wings optimized for cruise don't produce enough lift at approach speeds. High-lift devices fix this:

- **Trailing-edge flaps**: extended and rotated down to increase camber and effective area. Various types — plain, slotted, Fowler, double-slotted, triple-slotted — with increasing complexity and lift capability.
- **Leading-edge slats**: small airfoils ahead of the main wing that delay flow separation and allow higher angles of attack.
- **Krueger flaps**: alternative leading-edge devices.

The combination of full flaps and slats can roughly double the landing $C_L$ compared to clean configuration. This is why you watch the wings of an airliner and see complex mechanical assemblies extending and rotating as the plane approaches the runway — and why you hear them retracting on climb-out.

## Stability and Control

An aircraft must be aerodynamically stable — tending to return to trimmed flight after small disturbances — in all three axes:

- **Longitudinal stability** (pitch): tail-down force from the horizontal stabilizer usually provides this. Elevator controls pitch.
- **Directional stability** (yaw): vertical stabilizer (fin) provides this. Rudder controls yaw.
- **Lateral stability** (roll): wing dihedral (slight upward angle) provides this. Ailerons control roll.

Fighters are often designed intentionally marginally stable or unstable — flight control computers stabilize them. This gives maneuverability at the cost of needing always-on flight control. Airliners are stable so they can be flown by humans without computers if needed.

## Why This Level Matters

Aerodynamics is the physics that gives aircraft their possibility of flight. Every shape and proportion of an aircraft — the wing sweep, the tail size, the fuselage fineness ratio, the engine placement — reflects aerodynamic trade-offs within structural, propulsion, and operational constraints.

Understanding the basic aerodynamics equips you to read aircraft designs and explain why some configurations are common (swept wings on transonic airliners) and some are rare (tailless wings on commercial aircraft — harder to stabilize, but efficient for certain missions).

## The Transition to Level 2

L2 moves from aerodynamics to **propulsion** — the engines that overcome drag and turn fuel into thrust. Turbofans, turbojets, turboprops, piston engines, and the fundamental physics of jet propulsion.

Next: [L2 — Propulsion (Air)](./L2_Propulsion_Air.md) *(Phase 2D)*
