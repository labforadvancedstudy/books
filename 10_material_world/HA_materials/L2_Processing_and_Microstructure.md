# Level 2: Processing and Microstructure
*How processing creates microstructure, and microstructure determines properties*

<!-- Evidence Tier: Textbook -->

## The Central Triangle

Materials science is organized around a triangle:

- **Composition**: what elements and in what proportions.
- **Processing**: how the material is made, shaped, and treated.
- **Microstructure**: the internal arrangement of grains, phases, and defects.
- **Properties**: what the material does — strength, hardness, toughness, conductivity, and so on.

Composition sets limits. Processing creates the microstructure within those limits. Microstructure determines properties. Change the processing and the same composition can give very different properties — a wire of pure copper drawn cold is stronger and harder than the same copper annealed; a steel with 0.5% carbon is soft after slow cooling and hard after quenching.

If L1 explained what materials are, L2 explains how we tune them.

## Grains and Grain Boundaries

Metals solidifying from the melt form **grains** — individual crystals, each with the atomic lattice in a specific orientation. Where two grains meet, the lattice is disordered — a **grain boundary**.

Grain size matters:

- **Small grains (fine-grained metal)**: more grain boundaries, more obstacles to dislocation motion, higher strength and hardness. The **Hall-Petch relationship** quantifies this: yield strength increases as grain size decreases.
- **Large grains (coarse-grained)**: fewer grain boundaries, softer but often more ductile and creep-resistant (grain boundaries are weak points at high temperature).

Controlling grain size is one of the main tools of metallurgy. Casting gives one grain structure; hot-working another; cold-working and recrystallization yet another. The same alloy rolled into a sheet will have different grain structure than the same alloy cast into a block.

For specific applications, grains are engineered:
- **Turbine blades**: single-crystal castings (one grain, no boundaries) eliminate creep-prone grain boundaries and let the blade withstand higher temperatures.
- **Electrical steels for transformers**: grain-oriented steel with specific crystallographic texture to minimize magnetic losses.
- **Forged aerospace parts**: controlled fine grain structure for strength and fatigue resistance.

## Phase Diagrams

When you have two or more elements, they form **phases** — regions with a specific composition and crystal structure. Which phases appear depends on composition and temperature, captured by **phase diagrams**.

The iron-carbon diagram is the most famous:

- Pure iron has three crystal structures at different temperatures: ferrite (BCC), austenite (FCC), and δ-ferrite (BCC again).
- Adding carbon extends or shrinks these regions and introduces a hard, brittle phase — cementite (Fe₃C).
- Cooling austenite to room temperature gives different microstructures depending on cooling rate: slow gives pearlite (fine layers of ferrite and cementite), faster gives bainite, very fast (quenching) gives martensite — a very hard, distorted version of the crystal structure trapped by the rapid cooling.

The same 0.8% carbon steel can be soft and ductile (annealed → pearlite) or hard and brittle (quenched → martensite) depending on processing. Add tempering (reheating the quenched steel) to trade some hardness for toughness, and you can tune the mechanical balance precisely. Swords, springs, gears, tools — all steels made by variations on this theme.

Non-ferrous systems are similarly complex. Al-Cu, Ti-Al-V, Ni-Cr-Fe, and dozens of other binary and ternary systems each have a phase diagram that dictates what microstructure is achievable.

## Solidification

Most metals start as liquid and freeze. The solidification sequence controls the starting microstructure:

- **Nucleation**: solid crystals first appear at sites — wall surfaces, impurities, or by homogeneous freezing at supercooled points.
- **Growth**: existing crystals grow as more metal freezes. If cooling is directional (as in a casting mold), grains grow preferentially along the heat flow.
- **Solute rejection**: as metal solidifies, alloying elements are pushed into the remaining liquid; the last material to freeze is enriched in alloying elements. This is **segregation** — often undesirable, contributing to inhomogeneity.
- **Dendrite formation**: in most alloys, the solidification front is unstable and forms tree-like dendrites. The interdendritic regions become segregated.

Castings that cool quickly have fine structure; castings that cool slowly have coarse structure. **Inoculation** (adding nuclei to the melt) produces fine grains even at slow cooling rates. **Directional solidification** produces columnar grains aligned with the heat flow — used for turbine blades with a preferred grain orientation.

Continuous casting (used for most steel now) cools steel quickly at the surface and slowly at the core, producing complex through-thickness structure that hot-rolling later partially homogenizes.

## Deformation and Work Hardening

When you deform a metal plastically — by rolling, forging, drawing, extruding — two things happen to the microstructure:

- **Grains elongate**: in the direction of flow. Rolled sheet has pancake-like grains; drawn wire has long fibrous grains.
- **Dislocation density increases**: dislocations multiply and tangle. This increases the resistance to further deformation — **work hardening**.

A cold-drawn copper wire is 2–3× stronger than an annealed wire but much less ductile — keep working it and it cracks. **Annealing** — heating the deformed metal — lets new, defect-free grains nucleate and grow (**recrystallization**), restoring ductility at the cost of strength.

**Cold working**: deforming below the recrystallization temperature. Adds strength and residual stress.

**Hot working**: deforming above the recrystallization temperature. Recrystallization happens simultaneously with deformation, so the metal stays soft. Most bulk shape change (forging, hot rolling) is done hot; final finishing is often cold.

The sequence of work and anneal cycles — plus the temperature, strain rate, and rolling pass schedule — controls the final grain structure, texture, and mechanical properties of the product.

## Heat Treatment

Many alloys can be strengthened not just by working but by heat treatment alone:

**Quench and temper**: quench from austenite to form martensite (very hard, very brittle), then temper at intermediate temperature to let some carbon precipitate out of the distorted lattice, restoring toughness. Most structural steels are used in quenched-and-tempered condition.

**Precipitation hardening (age hardening)**: used for aluminum alloys, nickel superalloys, titanium alloys, some stainless steels. Solution treat (dissolve alloying elements in single phase), quench (trap them), then age (hold at intermediate temperature) to let nanoscale precipitates form. These precipitates obstruct dislocation motion and strengthen the alloy. Aircraft aluminum (2024, 7075) and turbine disks (Inconel 718) depend on precipitation hardening.

**Carburizing, nitriding, induction hardening**: surface treatments that harden just the outer layer (for wear resistance and fatigue) while keeping the core tough. Used for gears, shafts, bearings.

Heat treatment is both art and science. The exact times, temperatures, and cooling rates have been developed over decades for each alloy-application pair. A good heat treater can produce components that perform twice as well as the same alloy treated to generic recipes.

## Casting

**Casting** — pouring molten metal into a mold — is the oldest shaping process and still dominant for many parts.

Types:
- **Sand casting**: simple mold made of bonded sand. Cheap, low tooling cost, suitable for small runs and large parts. Lower surface finish, wider tolerance. Engine blocks, large industrial parts.
- **Investment casting**: wax pattern coated in ceramic, melted out, metal poured in. High precision, good finish, complex shapes. Turbine blades, surgical instruments, jewelry.
- **Die casting**: metal pushed into metal dies under pressure. High throughput, good finish, tight tolerances. Automotive components (transmission housings), consumer electronics.
- **Continuous casting**: used for semi-finished steel, aluminum, and copper shapes (billets, slabs). Produces near-net-shape mass quantities efficiently.

Every casting has defects: porosity (trapped gas), shrinkage voids, segregation, oxide inclusions. Good casting design minimizes these; inspection catches the rest.

## Additive Manufacturing

**Additive manufacturing (AM)** — 3D printing — builds parts layer by layer. For metals, the main technologies are:

- **Powder bed fusion**: laser or electron beam melts metal powder layer by layer. High precision, complex shapes. Used for aerospace parts, medical implants, tooling. Slow and expensive but enables designs impossible with traditional manufacturing.
- **Directed energy deposition**: powder or wire fed into a melt pool. Faster for larger parts. Used for repair and large aerospace components.
- **Binder jetting**: binder printed into metal powder, then sintered. Lower cost, good for medium complexity.

AM lets you design with internal channels for cooling, lattice structures for lightweight parts, topology-optimized geometries, and on-demand parts. It struggles with surface finish, tolerances in large parts, anisotropic properties (layer-to-layer bonding creates weaker direction), and build cost for simple geometries.

Current state (2025): AM is adopted for specific high-value applications (aerospace brackets, medical implants, tooling) but has not displaced traditional manufacturing for most parts. Expect continued growth in value-add applications rather than mass substitution.

## Joining

Finished assemblies usually require joining. The main methods:

- **Welding**: localized melting fuses parts. Many variants — arc welding (MIG, TIG, stick), friction stir welding, laser welding, resistance welding. Each has a characteristic heat-affected zone (HAZ) where the microstructure is disturbed by the welding thermal cycle.
- **Brazing and soldering**: lower-temperature joining with a filler metal that melts but the base metals don't. Brazing (600–1000°C), soldering (<450°C). Used for plumbing, electronics, HVAC.
- **Mechanical fastening**: bolts, rivets, screws. Reliable, removable in some cases, but heavier than welded joints.
- **Adhesive bonding**: epoxies, structural adhesives. Increasingly used in aerospace and automotive composite structures.

Welding quality depends on the process, the filler material, and often a post-weld heat treatment. Certified welders follow qualified procedures; critical welds (nuclear, aerospace, pressure vessels) are nondestructively inspected (ultrasonic, radiographic).

## Testing and Characterization

Knowing what microstructure you've produced requires characterization:

- **Optical microscopy**: polished and etched samples reveal grain structure at ~0.1 μm resolution. Standard quality control tool.
- **Scanning electron microscopy (SEM)**: resolution down to ~1 nm. Used for fine microstructure, fractography (analyzing failure surfaces).
- **Transmission electron microscopy (TEM)**: atomic-scale resolution. Used for dislocation structures, precipitates, thin films.
- **X-ray diffraction (XRD)**: identifies phases, measures residual stress, quantifies texture.
- **Electron backscatter diffraction (EBSD)**: maps grain orientations across a sample surface.
- **Hardness testing, tensile testing, impact testing, fatigue testing**: mechanical properties directly.

A modern materials lab uses most of these, often correlating. A failed turbine blade, for example, might be studied by optical microscopy (look for cracks and their pathways), SEM fractography (identify initiation sites), EBSD (verify grain structure), and XRD (measure residual stress) — and the combination tells you whether it was a material, processing, or operational problem.

## Why This Level Matters

Every engineered metal, ceramic, or polymer part is a product of specific processing that created specific microstructure that produces specific properties. Change any part and the others change. An engineer who wants specific properties must specify processing, not just composition.

Commercial material specifications (ASTM, AISI, AMS, ISO standards) are composition plus a processing state — not just "steel" but "AISI 4140 quenched and tempered to 32 HRC". The specification is meaningless without the processing state.

## The Transition to Level 3

L3 turns to **properties under service conditions** — how materials actually behave under load, heat, corrosive environments, and time. Fatigue, creep, stress corrosion, fracture mechanics. The behaviors that determine whether a part lasts a decade or fails next week.

Next: [L3 — Properties & Service Behavior](./L3_Properties_and_Service.md) *(deferred)*
