# Level 1: Statics
*Forces in equilibrium — the foundation of everything that doesn't move*

<!-- Evidence Tier: Textbook -->

## The First Question

If something is sitting still, the forces on it sum to zero. This is Newton's first law. A chair holding you up is doing real work (gravity pulls you; the chair pushes back; net force = zero). A bridge standing under traffic is in equilibrium (weight down, reactions up, horizontal forces balanced). The roof over your head is not moving because every force on it is canceled by another.

**Statics** is the branch of mechanics that studies bodies in equilibrium. It sounds dull. It is the basis on which every static structure in the world is designed, from the chair to the skyscraper.

The two equations that do nearly everything:

- $\sum \vec{F} = 0$: the vector sum of all forces on a body is zero.
- $\sum \vec{M} = 0$: the sum of all moments (torques) about any point is zero.

In three dimensions, this gives six scalar equations — three force components, three moment components. With six independent equations, you can solve for up to six unknowns per rigid body.

## Forces and Their Types

In a typical statics problem, forces come in several categories:

- **Gravity**: acts on the mass of every body, directed downward. For a uniform body, acts at the center of mass.
- **Reactions**: forces from supports. Pin supports provide two force components; roller supports provide one; fixed supports provide two forces plus a moment.
- **Applied loads**: external forces on the system (someone pushing, weight piled up, wind).
- **Internal forces**: tension, compression, shear in members of a structure. These are the things the engineer needs to size members for.

**Free body diagrams** (FBDs) are the primary tool. Draw the object of interest, isolate it, draw every force acting on it with direction and magnitude. A correct FBD makes the problem half-solved. A wrong FBD — missed force, wrong direction, wrong point of application — makes everything that follows worthless.

Engineering students spend semesters learning to draw FBDs reliably. It is a skill more than a knowledge; you learn it by practice.

## Trusses — The Simplest Structures

A **truss** is a structure made of straight members connected at joints. The members carry only axial loads (tension or compression, no bending). Bridges, roofs, cranes, and transmission towers are often trusses.

If the joints are idealized as frictionless pins and loads act only at the joints, each member is either in tension (being stretched) or compression (being squeezed). The analysis reduces to force balance at each joint — two equations per joint in 2D, three in 3D.

Two standard solution methods:

- **Method of joints**: analyze each joint in turn, solving for the forces in connected members using the joint's force balance.
- **Method of sections**: slice through the truss, treat one part as a rigid body, and apply the three equilibrium equations to solve for forces in the cut members.

A determinate truss has enough members to be stable but not so many that the problem is ill-posed. The count: for a planar truss with $j$ joints and $m$ members with $r$ reaction forces, the truss is determinate if $m + r = 2j$. Fewer members and the truss is unstable; more and the problem is statically indeterminate and requires additional equations (material behavior, compatibility).

## Beams

A **beam** is a long, slender member loaded transversely (perpendicular to its length). The beam holding up the floor above you is in bending: weight from above creates internal bending moments that try to curve the beam, balanced by bending stresses across its cross-section.

Beam analysis produces **shear force diagrams** and **bending moment diagrams** along the beam's length. These show where internal forces are largest and therefore where the beam needs the most strength. A simply supported beam with a central point load has maximum shear at the supports and maximum bending moment at the center. A cantilever (one end fixed, other free) has maximum bending moment at the fixed end.

The **flexure formula** relates stress in the beam to bending moment: $\sigma = M c / I$, where $M$ is bending moment, $c$ is distance from the neutral axis, and $I$ is the cross-section's second moment of area. Bigger $I$ means stiffer beam. A steel I-beam's flanges carry most of the bending stress; the web holds the flanges apart. This is why I-beams dominate structural steel — they minimize material for a given stiffness.

**Deflection** of a beam under load is also a statics problem (with the addition of material properties). A simply-supported beam of length $L$ loaded at the center deflects $\delta = PL^3/(48EI)$, where $P$ is load, $E$ is the modulus of elasticity, and $I$ is the second moment of area. Deflection matters for serviceability — a floor that sags visibly is unacceptable even if it's not about to collapse.

## Friction

Friction is the force that opposes relative sliding between surfaces in contact. The simple model: friction force ≤ μ × normal force, where μ is the friction coefficient. When sliding is impending, friction equals μN (at the **static friction** limit). Once sliding starts, the friction force is μ_kinetic × N, typically a bit smaller.

Coefficient of friction depends on the surfaces. Rubber on concrete: 0.6–1.0. Steel on steel (dry): 0.5–0.8. Steel on ice: 0.01. Lubricated surfaces: 0.01–0.1.

Statics problems with friction often involve finding whether a body will slide or tip. The **angle of repose** for a granular material (the steepest slope it can sustain without flowing) is determined by internal friction. Sand's angle of repose is about 34°; gravel, about 40°. Beyond that, the slope fails.

Friction is what keeps your car from slipping, your shoes from sliding, the nut on the bolt from loosening. It is also what generates brake heat, tire wear, and energy losses in every mechanical system. Tribology — the study of friction, wear, and lubrication — is a specialty in its own right.

## Centers of Mass and Moments of Inertia

**Center of mass**: the point where you can treat a body's weight as acting. For uniform shapes, it's the geometric centroid. For compound shapes, it's a weighted average of the component centroids.

**Second moment of area** (also called moment of inertia, area moment of inertia): a property of a cross-section that measures how its area is distributed relative to an axis. It determines the cross-section's resistance to bending. A hollow square tube has higher $I$ per unit area than a solid round rod of the same weight — which is why bicycle frames, car chassis, and aluminum ladders are tubes, not solid bars.

For a rectangle of width $b$ and height $h$ about its centroidal horizontal axis: $I = bh^3/12$. The cubic dependence on height is why a 2×12 joist loaded on edge is many times stiffer than the same board loaded flat.

## Stability and Buckling

A column in compression can fail in two ways: by crushing (stress exceeds yield) or by **buckling** (bowing sideways before stress limits are reached). Buckling is a stability phenomenon — the column is in equilibrium up to a critical load, then becomes unstable.

Euler's formula for the critical buckling load: $P_{cr} = \pi^2 EI / L_e^2$, where $L_e$ is the effective length (depends on end conditions). A long slender column fails at a low critical load. A short stubby column fails by crushing before buckling matters.

Buckling design is critical in compression members: columns, struts, ship hulls, pressure vessels. Many spectacular structural failures (the Tacoma Narrows Bridge being partially a related phenomenon, various silo and pressure vessel collapses) involve instability.

## Indeterminate Structures

Most real structures have more supports or members than the three equilibrium equations can resolve — they are **statically indeterminate**. Additional equations come from material properties and geometric compatibility (how much a member stretches under a given force relates to how much adjacent members stretch).

For example, a beam fixed at both ends has six reaction components (three at each support) but only three equilibrium equations. The remaining three unknowns are found by requiring that the fixed ends don't rotate and don't translate — compatibility conditions that tie the reactions to the beam's stiffness.

Indeterminate structures have advantages: load redistribution when one part yields, backup load paths if something fails. They also require more sophisticated analysis. Modern structural design is done with computer finite-element methods that handle indeterminacy routinely.

## Where Statics Fails: Dynamics

Statics assumes nothing is accelerating. In reality, many structures experience time-varying loads — wind gusts, earthquakes, moving vehicles, machinery vibrations. When acceleration matters, you need **dynamics**, which adds inertial forces to the force balance.

A resonant structure (a suspension bridge, a tall slender building, a machinery foundation) can be driven to large-amplitude vibrations by modest periodic forcing. Designers have to consider natural frequencies and damping, not just static capacity. This extends into a different set of skills (structural dynamics, vibration analysis) that build on top of statics.

## Factors of Safety

No engineer designs a structure to carry exactly its calculated maximum load. Material properties vary; loads are uncertain; workmanship is imperfect; analysis is approximate. Design codes require **factors of safety** — typically 1.5–3.0 for building structures, higher for pressure vessels or aircraft, lower for dead-load-dominated structures with well-known materials and loading.

In allowable stress design, you keep stresses below yield divided by the safety factor. In load and resistance factor design (LRFD), you multiply loads up and material strengths down by various factors based on the load type and the confidence in the material behavior.

Factor of safety is not redundant design — it's acknowledgment of uncertainty. Too low and you have frequent failures. Too high and you waste material and money. The codes encode accumulated industry experience about what factors give acceptable reliability for given applications.

## Why This Level Matters

Everything that stands up — every building, bridge, chair, bookshelf, crane, vehicle frame — was designed using statics. The methods are old (Archimedes worked on lever equilibrium; Galileo did beam bending; Newton gave the modern vector foundation), well-validated, and essential.

A mechanical or civil engineering undergraduate spends a full semester on statics because it's the vocabulary of the field. Everything that follows — mechanics of materials, structures, machine design — builds on it.

## The Transition to Level 2

L2 moves from equilibrium (static forces) to **dynamics** — what happens when things accelerate, rotate, and respond to time-varying forces. Newton's second and third laws come fully into play.

Next: [L2 — Dynamics](./L2_Dynamics.md) *(Phase 2D)*
