# Level 1: Atomic Structure & Properties
*Why steel is strong, why glass is brittle, why copper conducts*

<!-- Evidence Tier: Textbook -->

## Materials Are Not Continuous

Pick up a steel bolt. It looks solid, uniform, unchanging. It is none of those things.

At the atomic scale, the bolt is iron atoms packed in a lattice, with a few percent carbon atoms wedged into gaps, trace amounts of manganese, silicon, and other elements scattered through, and everywhere **defects** — missing atoms, extra atoms, misaligned planes, grain boundaries where one crystal ends and another begins. The macroscopic properties of the bolt — its strength, hardness, toughness, ductility — emerge from that messy atomic-scale structure.

This is the central insight of materials science: structure determines properties, and structure exists at many scales simultaneously (atoms, defects, grains, phases, composites). A materials engineer who understands the structure can predict the properties, and sometimes design the structure to produce the properties wanted.

## The Four Classes

Almost every engineering material falls into one of four classes:

- **Metals**: iron, aluminum, copper, titanium, and alloys of these. Atoms held together by a sea of shared electrons (metallic bonding). Dense, conductive, tough, ductile (they deform rather than snap).
- **Ceramics**: oxides, nitrides, carbides. Alumina (Al₂O₃), silica (SiO₂), zirconia, silicon carbide. Strong ionic or covalent bonds. Hard, stiff, high melting point, but brittle — they crack rather than bend.
- **Polymers**: long chains of repeating molecular units. Polyethylene, nylon, PVC, rubber, biological polymers like cellulose and proteins. Held by covalent bonds along the chain, weaker intermolecular forces between chains. Low density, low stiffness, viscoelastic.
- **Composites**: two or more materials combined to get properties neither has alone. Fiber-reinforced polymers (carbon fiber, fiberglass), concrete (aggregate in a cement matrix), wood (cellulose fibers in lignin), bone (collagen with calcium phosphate mineral).

Every material you touch is in one of these classes or is a composite of them. The class determines the basic property set; the microstructure determines the refinements.

## Why Steel Is Strong

Pure iron is soft. Pure iron with 0.2–2% carbon, appropriately heat-treated, is steel — one of the strongest structural materials ever used.

The reason comes down to defects. When you pull on a metal, the atoms don't all slide past each other at once. Instead, **dislocations** — line defects in the crystal — move through the metal. A dislocation is like a wrinkle in a rug: you can move the wrinkle much more easily than you can slide the whole rug.

Pure metal has few obstacles to dislocation motion, so dislocations move easily and the metal deforms under modest force. Adding carbon (and other elements) introduces obstacles — foreign atoms, small precipitates, grain boundaries. Dislocations get pinned. The metal is harder and stronger.

**Heat treatment** is how metallurgists control this. Heat steel into the austenite phase (above ~720°C), then quench it rapidly in water or oil, and you trap carbon atoms in positions where they distort the crystal heavily — this is **martensite**, extremely hard but brittle. Temper it by reheating to 200–600°C and you allow some relaxation, trading hardness for toughness.

The same principle — control of defects and phases — makes modern aluminum alloys, nickel superalloys for jet engines, and titanium for aerospace structures. A jet engine turbine blade is a single crystal (no grain boundaries at all) with precisely engineered precipitates — a century of metallurgy in one part.

## Why Glass Is Brittle

Glass is mostly silica (SiO₂), sometimes with added soda (Na₂O) and lime (CaO) to lower the melting point. Unlike most crystalline ceramics, glass has a **disordered** atomic structure — it's frozen-in liquid. Cool it fast enough from the melt and the atoms never find their crystalline positions.

Glass is strong in compression and weak in tension. A glass bottle can hold significant internal pressure. The same bottle shatters if you drop it on concrete.

The weakness is about cracks. Brittle materials fail when a small surface flaw concentrates stress until the material pulls apart at that flaw. Cracks propagate through brittle materials because there's no mechanism to stop them — no dislocations to blunt the tip, no ductile yielding to redistribute stress.

Engineers deal with brittleness by:
- **Avoiding tensile loads** where possible (concrete is used in compression; steel rebar handles the tension).
- **Toughening** (pre-compressing the surface with tempered glass; adding fibers to make fiberglass; laminating with polymer layers to make auto safety glass).
- **Controlling flaw size** (carefully polished optical glass can be far stronger than rough window glass — smaller initial flaws mean higher achievable stresses).

## Why Copper Conducts

A copper atom has one loosely held outer electron. In solid copper, those electrons become a shared "electron gas" that moves almost freely through the crystal. Apply a voltage, and the electrons drift in response, carrying current.

Aluminum, silver, and gold conduct for the same reason, with slight differences in efficiency. Silver is the best conductor per unit mass but too expensive for most uses. Copper is the workhorse of electrical wiring. Aluminum is used for long-distance power lines because it's lighter and cheaper despite being slightly less conductive per cross-section.

In contrast, plastics and glasses have all their electrons tightly bound in covalent bonds. No free electrons to carry current. These are insulators.

Semiconductors — silicon, germanium, gallium arsenide — are in between. Nearly all their electrons are bound, but a few can be promoted to carry current, and the number can be controlled by doping with impurities. This is the physical basis of modern electronics and is covered in more depth in the electrical engineering book.

**Resistance** arises because electrons scatter off defects, impurities, and thermal vibrations. Cool a metal, and resistance drops. In certain materials at very low temperatures, resistance drops to zero — **superconductivity** — a quantum-mechanical effect that is the subject of ongoing research and one of the most dramatic examples of how atomic-scale structure can produce macroscopic surprises.

## The Stress-Strain Curve

If you pull a metal rod in a testing machine and plot force (normalized to cross-sectional area, giving **stress**) against extension (normalized to original length, giving **strain**), you get a curve that tells you almost everything about the material's mechanical behavior:

- **Elastic region**: initial straight line. Stress is proportional to strain (Hooke's law). Remove the load and the material returns to its original shape. Slope is the **elastic modulus** (also called Young's modulus, stiffness).
- **Yield point**: where the curve starts to deviate from the line. Beyond yield, the material deforms permanently.
- **Plastic region**: large strain at modest stress. The material is flowing plastically.
- **Ultimate tensile strength**: maximum stress the material can sustain.
- **Necking and fracture**: material thins locally and then breaks.

Steel has high elastic modulus (~200 GPa), yield around 250–700 MPa depending on alloy and treatment, and ultimate strength higher still. Rubber has very low elastic modulus (~0.01 GPa) and huge elastic strains. Concrete has modest elastic modulus (~30 GPa) and negligible plastic range — it cracks rather than yields.

A structural engineer uses these curves to design for **strength** (not exceeding yield) or **serviceability** (not deflecting too much) or **fatigue** (repeated loading below yield can still cause failure over enough cycles).

## Density and Specific Properties

Aerospace engineering obsesses over **specific strength** (strength divided by density) and **specific stiffness** (stiffness divided by density) because on an aircraft, every kilogram of structure is a kilogram that can't be paying cargo.

Carbon fiber composites have specific strength several times that of steel. This is why modern airliners (Boeing 787, Airbus A350) are built substantially from carbon fiber composite rather than aluminum. The raw carbon fibers have strength comparable to steel, but the composite is a quarter to a fifth the density.

For pure weight-bearing ground structures, where density matters less, steel and concrete still dominate because they're cheap. The right material is always the one that optimizes for the specific use case.

## Corrosion — The Material War You Are Always Losing

Most metals oxidize. Iron rusts, aluminum forms Al₂O₃ (which actually protects the underlying metal — a benefit), copper develops a green patina, steel structures lose a measurable fraction of their cross-section per decade if unprotected.

Corrosion costs the global economy an estimated 3–4% of GDP — trillions of dollars per year in lost infrastructure, replacement parts, and preventive maintenance. Fighting corrosion is:

- **Coating** (paint, galvanizing with zinc, plating with chromium).
- **Alloying** (stainless steel adds chromium, which forms a protective oxide).
- **Cathodic protection** (sacrificial zinc anodes on ship hulls).
- **Material selection** (titanium or plastic for highly corrosive environments).

Corrosion protection is a mundane but enormous branch of materials engineering. The reason your car lasts 15 years and not 5 is largely the quality of rustproofing developed over the last few decades.

## Why This Level Matters

At the level of atomic structure and basic properties, materials science is about knowing what materials can do and why. Every engineered object was chosen — deliberately or by inheritance — from the material palette available to the designer.

Once you understand the four classes, the role of defects, and the basic mechanical and electrical properties, you can read a cross-section and predict how a material will behave. You can also recognize when you've reached the limits of what the material allows and need to switch classes or invent a new composite.

## The Transition to Level 2

With basic materials understood, L2 moves to the **processing-structure-properties triangle**: how manufacturing processes shape the microstructure, and how microstructure determines properties. That's where materials engineering gets useful in practice.

Next: [L2 — Processing & Microstructure](./L2_Processing_and_Microstructure.md) *(Phase 2D)*
