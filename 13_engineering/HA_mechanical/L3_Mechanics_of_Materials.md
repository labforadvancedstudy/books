# Level 3: Mechanics of Materials
*Stress, strain, and the internal response of solid bodies to loads*

<!-- Evidence Tier: Textbook -->

## Inside the Body

L2 covered dynamics — how bodies move under force. L3 goes inside the body. A beam doesn't just deflect as a rigid whole; it develops internal stresses and strains that vary from point to point. A bolt carries load by shear on its cross-section. A pressure vessel contains pressure via hoop stress in its walls. A shaft transmits torque via torsional shear. Every mechanical component — from a paperclip to a turbine disk to a bridge girder — is designed by analyzing the internal force distribution and ensuring the material can sustain it without yielding, fracturing, or deforming unacceptably.

Mechanics of materials — sometimes called strength of materials or solid mechanics at the introductory level — is the branch of engineering that analyzes these internal effects. It's foundational for machine design, structural engineering, pressure vessel design, bolt and weld sizing, and much else. It is where material properties (from L2 of HA_materials) meet geometry and loading (from L2 of HA_mechanical) to produce component-level design.

## Stress and Strain

**Stress** is internal force per area. Units: Pa (N/m²) in SI; ksi or psi in US customary.

Two basic types:
- **Normal stress** σ: perpendicular to a surface. Tension (positive) or compression (negative).
- **Shear stress** τ: parallel to a surface.

**Strain** is deformation per length.
- **Normal strain** ε: change in length / original length. Dimensionless.
- **Shear strain** γ: angular distortion. Dimensionless (radians).

For a uniform bar under axial load F and area A: σ = F/A, ε = ΔL/L.

**Stress-strain curve** (tension test):
- **Linear elastic region**: σ = E·ε, where E is Young's modulus (GPa, material constant). Steel ~200 GPa, aluminum ~70 GPa, copper ~117 GPa, titanium ~115 GPa, concrete ~30 GPa, wood along grain ~10-15 GPa.
- **Proportional limit** / **elastic limit**: upper bound of linear/elastic region.
- **Yield point**: stress at which permanent deformation begins. Steel ~200-1000+ MPa depending on grade.
- **Plastic region**: permanent deformation; work hardening.
- **Ultimate tensile strength**: maximum stress.
- **Fracture**: specimen breaks.

**Poisson's ratio** ν: ratio of transverse to axial strain under uniaxial stress. ~0.3 for metals, up to ~0.5 for incompressible rubbers.

**Shear modulus** G: τ = G·γ. Related: G = E / [2(1+ν)].

**Bulk modulus** K: resistance to volumetric compression.

These four constants (E, ν, G, K) are interrelated for isotropic materials; two are independent.

## Axial Loading

Simplest case: straight member with axial load F.
- **Stress**: σ = F/A.
- **Strain**: ε = σ/E.
- **Deformation**: δ = FL/(AE).
- **Thermal expansion**: ε_thermal = α·ΔT (α = coefficient of thermal expansion; steel ~12×10⁻⁶/°C).

**Statically indeterminate problems** (more unknowns than equilibrium equations): require compatibility (deformation) conditions. Common in bolted joints, composite members.

**Saint-Venant's principle**: stress distributions due to concentrated loads become uniform at distances comparable to the cross-section dimension — except at the point of application.

## Torsion

Circular shaft twisted by torque T:
- **Shear stress**: τ = Tr/J, where r is radial position, J is polar moment of inertia.
- **J for solid circular** shaft: πd⁴/32.
- **Angle of twist**: φ = TL/(GJ).

Maximum shear stress at outer radius. Torsional stiffness scales with fourth power of diameter — doubling diameter gives 16× torsional strength.

**Non-circular shafts**: warp under torsion. Formulas more complex; often tabulated.

**Torsional failure**: ductile materials fail in shear (flat fracture); brittle in tension (helical fracture at 45°). Chalk-vs-steel demonstrations.

## Bending of Beams

**Beam**: structural member primarily in bending. Analysis staples:

**Shear force and bending moment diagrams**: plot V(x) and M(x) along beam length for given loading and supports. Foundation for beam design.

**Flexure formula**: σ = My/I, where M is bending moment, y is distance from neutral axis, I is area moment of inertia.

**I for common shapes**:
- Rectangle (b wide, h tall): I = bh³/12.
- Circle (d diameter): I = πd⁴/64.
- I-beam: tabulated; optimized by placing material far from neutral axis.

**Maximum bending stress**: σ_max = Mc/I = M/S, where S = I/c is section modulus. Larger S = more bending capacity per weight.

**Beam deflection**: differential equation EI·d⁴y/dx⁴ = w(x). Integrated for deflection; tabulated solutions for common cases (cantilever with point load, simply supported with uniform load, etc.).

**Statically indeterminate beams**: use superposition, conjugate beam, or finite element.

**Composite beams**: different materials in same cross-section (reinforced concrete, sandwich panels). Transformed section method.

## Shear in Beams

Bending is accompanied by transverse shear:
- **Shear stress distribution**: τ = VQ/(Ib), where Q is first moment of area above (or below) the point of interest.
- **Rectangular beam**: max shear at neutral axis, = 1.5 × V/A.
- **I-beam**: flange carries most bending, web carries most shear.

**Shear flow** in thin-walled sections: q = VQ/I (force per length). Used in aircraft structures, box beams.

## Combined Loading and Principal Stresses

Real components carry combinations — axial + bending + torsion + pressure. At any point, stress state is fully described by stress tensor (3 normal components, 3 shear components in 3D; fewer in 2D).

**Mohr's circle** visualizes stress transformation under coordinate rotation. Gives:
- **Principal stresses** (maximum and minimum normal stresses; zero shear on principal planes).
- **Maximum shear stress**.
- **Orientation** of principal planes.

**Failure criteria**: for multiaxial stress, when does yielding/fracture occur?
- **Maximum normal stress (Rankine)**: brittle materials. σ_max ≥ σ_ult → fracture.
- **Maximum shear stress (Tresca)**: ductile materials. τ_max ≥ σ_y/2 → yield. Conservative.
- **Distortion energy (von Mises)**: ductile materials, common in design. σ_vm ≥ σ_y → yield. Slightly less conservative than Tresca, often more accurate.
- **Mohr-Coulomb, Drucker-Prager**: materials with different tension/compression strengths (concrete, soils, ductile iron).

## Buckling

Long slender columns under compression can fail by **buckling** — sudden lateral deflection — well before yielding.

**Euler buckling load**: P_cr = π²EI/(KL)², where KL is effective length (K depends on end conditions — 0.5 for fixed-fixed, 1 for pinned-pinned, 2 for cantilever).

**Slenderness ratio**: L/r (r = radius of gyration = √(I/A)). Above a critical slenderness, Euler governs. Below, inelastic buckling or yielding.

**Johnson formula**: empirical intermediate range.

**Practical buckling**: imperfections (initial curvature, load eccentricity) reduce capacity. Design codes include generous safety factors.

**Local buckling** of thin-walled sections (flanges, webs, tubes) a separate concern; tables and FEA used.

## Energy Methods

Alternative to force/deformation: use strain energy.
- **Strain energy** U = ½·σ·ε per volume for linear elastic.
- **Castigliano's theorem**: δ = ∂U/∂F. Powerful for deflections of complex structures.
- **Principle of virtual work, unit-load method**: systematic deflection calculation.
- **Rayleigh-Ritz, Galerkin**: approximate methods; foundation of FEA.

These methods scale better than direct integration for complicated geometries.

## Pressure Vessels

**Thin-walled pressure vessels** (wall thickness < ~10% of radius):
- **Cylindrical**: hoop stress σ_h = pr/t; axial stress σ_a = pr/(2t). Hoop is 2× axial.
- **Spherical**: σ = pr/(2t). Most efficient shape.

**Thick-walled cylinders**: Lamé equations. Stress varies from inside to outside.

Applications: gas tanks, pipelines, boilers, nuclear vessels, scuba tanks.

## Failure Modes in Service

Components can fail by:

**Yielding**: plastic deformation exceeds tolerance. Design: keep σ < σ_y / safety factor.

**Fracture**: material separates. Covered in L3 of HA_materials.

**Fatigue**: failure under repeated loading below static strength. S-N curves relate stress amplitude to cycles to failure. Endurance limit (for steels) ~50% of ultimate, below which fatigue life is effectively infinite. Miner's rule sums damage fractions.

**Creep**: time-dependent deformation at high temperature (>~0.4·T_melt absolute). Critical in turbines, boilers, steam lines.

**Fretting**: wear at contacting surfaces with small relative motion.

**Stress concentration**: notches, holes, fillets, keyways concentrate stress by factors of 2-5×. Fatigue and brittle fracture particularly sensitive. Rounded fillets, polished finishes mitigate.

## Experimental Methods

Theoretical analysis is complemented by test:
- **Strain gauges**: bonded to part; resistance changes with strain. Rosettes for multi-direction.
- **Photoelasticity**: transparent polymer models show fringe patterns under polarized light indicating stress.
- **Digital image correlation (DIC)**: optical full-field strain measurement.
- **X-ray diffraction**: measures residual stress in crystalline materials.
- **Proof testing**: applying known load to verify capacity without failure.
- **Fatigue testing**: component-level life tests, often taking weeks to months.

## Finite Element Analysis (FEA)

For complex geometries, FEA is standard:
- **Discretization**: solid divided into elements (tetrahedra, hexahedra, beams, shells).
- **Element formulation**: derived from energy methods; stiffness matrices.
- **Assembly**: global stiffness matrix.
- **Solution**: [K]{u} = {F}; solve for displacements.
- **Post-processing**: stresses, strains, displacements visualized.

Tools: ANSYS, Abaqus, Nastran, COMSOL, many cloud-based. Accuracy depends on mesh refinement, element choice, material model fidelity, boundary condition realism.

FEA can be powerful but misleading: "garbage in, garbage out" — bad inputs produce plausible-looking wrong results. Verification and validation (V&V) discipline important.

## Design Applications

Mechanics of materials directly drives:
- **Shaft design**: torsion + bending + fatigue. Endurance limit with notch, surface, size factors.
- **Bolt sizing**: preload, shear, bending, fatigue.
- **Weld design**: shear/tension through throat, fatigue.
- **Springs**: helical torsion, deflection = 8FD³n/(Gd⁴).
- **Gears**: tooth bending (Lewis formula → AGMA refinements) and surface fatigue (pitting).
- **Bearings**: contact stress (Hertzian), rolling fatigue L10 life.
- **Pressure piping**: thickness from hoop stress plus allowances.
- **Structural members in machines**: columns, beams, brackets.

Design codes (ASME, AISC, API) codify approaches with safety factors calibrated from experience.

## Dimensional Considerations and Scaling

Material strength ~ area (length²). Weight ~ volume (length³). Scaling up: weight grows faster than strength. Structures must get relatively more massive, or use lighter materials, as they grow.

This principle limits natural and built structures. Elephants can't jump; large ships need progressively beefier hulls; tall buildings need super-high-strength concrete and steel at lower floors.

## Plastic Analysis

For ductile materials loaded beyond yield, plastic analysis can give higher capacities than elastic:
- **Plastic section modulus Z**: for rectangle, Z = 1.5·S. For I-beam, Z ≈ 1.1-1.15·S.
- **Plastic hinges**: fully yielded cross-sections in beams at collapse.
- **Limit analysis**: upper/lower bound theorems.

Applied in structural steel design, pressure vessel design for specific codes, seismic design where ductility is desired.

## Why This Level Matters

Every machine component and every structural member is sized via mechanics of materials. A pump shaft too small fails; too large wastes weight and cost. A bolt too small tears; too large is unnecessary mass. A beam too thin sags; too thick is overkill. These decisions, made thousands of times in any substantial engineered product, determine whether it works, what it costs, how long it lasts.

Computational tools have automated much of the calculation. But the fundamental understanding — what stress means, how it distributes, what failure modes exist, what geometric choices affect — remains essential. An engineer who understands only software outputs without the underlying principles cannot recognize when the software is wrong, what simplifications are reasonable, or how to design rather than just analyze.

## The Transition to Level 4

L4 turns to **machine design and manufacturing integration** — how components are designed as systems, how manufacturing processes shape what's producible, and the iterative cycle of design, test, refinement, and production.

Next: [L4 — Machine Design & Manufacturing](./L4_Machine_Design_and_Manufacturing.md) *(deferred)*
