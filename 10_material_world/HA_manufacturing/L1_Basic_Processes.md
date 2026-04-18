# Level 1: Basic Processes
*How you turn a raw lump into a useful part*

<!-- Evidence Tier: Textbook -->

## Five Families

Nearly every object made by humans is the result of one or more of five basic families of manufacturing process:

- **Casting**: pour liquid material into a mold; let it solidify.
- **Forming**: reshape solid material without removing any (forging, rolling, extrusion, drawing, stamping).
- **Machining**: remove material from a solid blank (turning, milling, drilling, grinding).
- **Joining**: combine multiple parts into one (welding, brazing, soldering, fastening, adhesives).
- **Additive** (a.k.a. 3D printing): build up a part by adding material layer by layer.

Real parts often combine these. A cast engine block is machined to final tolerances, then assembled (joined) with other parts. A sheet of steel is rolled (formed), then stamped (formed again) into a car door, then spot-welded (joined) to the body-in-white, then painted.

## Casting — The Oldest

Casting is about 6,000 years old. Melt metal, pour it into a mold shaped like what you want, let it cool. Remove the mold.

Modern casting varieties:

- **Sand casting**: mold made of compacted sand with a binder. Cheap, large parts, rough surface. Used for engine blocks, heavy industrial parts.
- **Die casting**: molten metal forced under pressure into a permanent steel mold. Fast cycle, good surface finish. Used for aluminum and zinc parts — laptop bodies, small engine components, toys.
- **Investment casting** (lost-wax): wax pattern encased in ceramic slurry, wax melted out, metal poured in. Expensive but capable of complex shapes with fine detail. Jet engine turbine blades are made this way.
- **Continuous casting**: steel is poured continuously through a water-cooled mold, producing endless billets or slabs. Almost all modern steel starts here.

Casting's power is geometric freedom. You can make shapes that would be impossible to machine. Casting's limits are dimensional accuracy, surface finish, and defects (porosity, inclusions, shrinkage cracks). A cast part usually needs some machining afterward to be dimensionally usable.

## Forming — Reshaping Without Removing

Forming is the largest-volume category by tonnage. The world's steel and aluminum are mostly sold as rolled sheet, bar, or plate, produced by passing hot or cold metal through rollers.

- **Forging**: hammering or pressing metal into shape. A crankshaft or connecting rod for an engine is typically forged to align the grain flow with the load direction, producing a part stronger than a machined or cast equivalent.
- **Rolling**: metal is passed through rollers to reduce thickness. Makes sheet, strip, plate, rails, beams.
- **Extrusion**: metal is forced through a die to produce a continuous cross-section. Aluminum extrusions are ubiquitous — window frames, heat sinks, structural channels.
- **Drawing**: wire drawn through progressively smaller dies. The wires in your walls and in your fiber-optic cables are drawn.
- **Stamping/pressing**: sheet metal pressed between a die and a punch into complex shapes. Car body panels, appliance shells, aluminum cans.
- **Bending**: simpler deformations — sheet into channels, tubes, brackets.

Forming works because metals are ductile and can deform plastically without breaking. The work required to deform them is captured as heat; the metal gets stronger through **work hardening** as its dislocation density increases; and if deformed enough, it needs annealing (heat treatment) to restore ductility for further processing.

## Machining — Subtractive Precision

A CNC (Computer Numerical Control) milling machine can remove metal in a programmed pattern to tolerances of a few micrometers. Modern machining is the basis of most precision manufacturing.

- **Turning**: workpiece rotates; a cutting tool removes material in a circular pattern. Makes shafts, rods, cylindrical parts. Done on a lathe.
- **Milling**: cutting tool rotates; workpiece (or tool) moves. Can make flat surfaces, slots, pockets, and — with multi-axis machines — almost any shape. Done on a mill or machining center.
- **Drilling**: rotating tool pushes into the workpiece to produce holes.
- **Grinding**: abrasive wheel removes small amounts to achieve very fine tolerances and surface finishes. Used for bearings, gears, and precision fits.
- **EDM** (Electrical Discharge Machining): electrical sparks erode a conductive workpiece. Capable of making shapes that mechanical tools can't reach.

Machining is precise, flexible, and capable of complex geometries, but it's wasteful — you pay for the bulk material and then throw most of it away as chips. That's acceptable for prototypes, low-volume parts, and precision work; less so for mass production of simple shapes where forming or casting is cheaper.

## Joining — Making One Thing From Many

Almost nothing is a single part. A car has roughly 30,000 parts. A jet engine, 25,000. A passenger airliner, about 4 million. Getting all these parts to stay together reliably is a discipline.

- **Welding**: locally melt both parts and a filler material so they fuse into a continuous solid. Arc welding (SMAW, GMAW, GTAW), laser welding, resistance (spot) welding. Cars are held together primarily by spot welding.
- **Brazing and soldering**: a lower-melting filler bonds two parts without melting them. Plumbing joints, electronics.
- **Fasteners**: bolts, screws, rivets. Mechanically removable (usually). Aircraft fuselages are riveted in large numbers.
- **Adhesives**: glues and structural adhesives. Modern automotive and aerospace designs use structural adhesives in load-bearing joints, often replacing or supplementing welds and rivets.
- **Interference fits**: press two parts together at a size where they grip each other permanently. Used for bearings, pulleys, some shaft-hub joints.

Each joining method has specific strength, failure modes, inspection requirements, and cost. Welds can crack along heat-affected zones. Fasteners can loosen. Adhesives can creep. Good manufacturing specifies the joining method to match the service conditions.

## Additive — The New One

3D printing (officially, **additive manufacturing**) builds up parts from raw material layer by layer rather than removing material from a blank. There are several technologies:

- **FDM** (Fused Deposition Modeling): plastic filament melted and extruded through a nozzle. Cheap, hobbyist-friendly, limited materials.
- **SLA** (Stereolithography): UV laser cures liquid photopolymer layer by layer. Smooth finish, small parts.
- **SLS** (Selective Laser Sintering): laser fuses powdered plastic or metal. Capable of complex shapes in functional materials.
- **DMLS / SLM** (Direct Metal Laser Sintering / Selective Laser Melting): laser melts metal powder. Used in aerospace and medical for complex titanium and superalloy parts.
- **Binder Jetting**: inkjet-printed binder onto powder, then sintered or infiltrated.

Additive's strengths: geometric complexity (internal channels, lattice structures, lightweighting), low-volume custom parts (medical implants), rapid prototyping.

Additive's limits: slow compared to mass production (hours to days per part), limited material options, surface finish usually needs machining, and parts often need post-processing. It is not replacing traditional manufacturing for most high-volume applications. It is adding a new capability niche.

## Tolerance and Surface Finish

Real parts are not perfect. Every manufactured feature has a **tolerance** — a specified allowable deviation from the nominal dimension. A shaft might be specified as 25 mm ± 0.05 mm. A cylinder head bolt torque might be 100 ± 5 N·m. A precision optical surface might be specified to 100 nm flatness.

Tighter tolerances cost more. Much more. A rough sand-cast part might tolerate ±1 mm dimensional variation. A machined part might hit ±0.01 mm. A ground bearing race might hit ±0.001 mm. Laboratory-scale optical surfaces reach ±10 nm. Each step of tighter tolerance typically costs 10× more.

Good engineering practice: specify tolerances as loose as the function allows. Tight tolerances everywhere is a mark of amateur design.

**Surface finish** — the roughness of a surface — is similarly cost-sensitive. A sand-cast part has rough millimeter-scale bumps. A machined part has micrometer-scale tool marks. A ground or polished part has nanometer-scale smoothness. Polished optical surfaces can be smoother than that.

## Quality Control

A modern factory running at millions of parts per year cannot inspect every dimension on every part. Instead it uses **statistical process control**: sample parts, measure, compute statistics, adjust the process if the distribution drifts out of specification.

Measurement devices include calipers, micrometers, coordinate measuring machines (CMM), optical profilometers, laser scanners, and increasingly automated vision systems. For critical parts (medical, aerospace), every feature on every part is measured and recorded for traceability.

**Six Sigma** is the goal of keeping defect rates below 3.4 per million parts. Auto parts, semiconductors, and medical devices routinely operate at that level or better. Achieving it requires discipline across the entire supply chain, not just the final factory.

## Why This Level Matters

Everything around you was made by one of these processes, usually several in sequence. Understanding them helps you:

- Estimate the cost of making something.
- Decide what to make vs. what to buy.
- Recognize the limits of each process (why can't I just print this part? because FDM plastic has 1/10 the strength of machined aluminum).
- Design parts that can actually be made efficiently (Design for Manufacturing, DFM).

A designer who knows manufacturing draws parts that can be made cheaply. A designer who doesn't draws parts that either can't be made at all or cost ten times what they should.

## The Transition to Level 2

With basic processes understood, L2 moves to the factory as a system — workflow, automation, quality assurance, logistics. Manufacturing at scale is a different discipline from making a single part.

Next: [L2 — Factory Systems](./L2_Factory_Systems.md) *(Phase 2D)*
