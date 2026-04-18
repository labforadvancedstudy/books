# Level 3: Properties and Service Behavior
*Mechanical behavior, failure modes, corrosion, fatigue, and how materials perform in real use*

<!-- Evidence Tier: Textbook -->

## From Making to Using

L2 covered how materials are processed and how microstructure emerges. L3 turns to what microstructure does. Strength, toughness, hardness, ductility — the familiar mechanical properties — are the first-pass answer, but real service is harder. Materials fail in service through fatigue after billions of stress cycles, through corrosion in aggressive environments, through creep at high temperature, through wear at sliding surfaces, through brittle fracture at low temperature. Each failure mode has its own physics, its own characteristic signatures, and its own design approach.

Understanding service behavior is what distinguishes materials selection that works from materials selection that costs lives. A steel that is strong enough at room temperature but becomes brittle below freezing took down ships (the Liberty ship fractures of WWII); a titanium alloy that is inert to most environments but reacts violently with chlorine took down refineries. The relationship between microstructure, composition, and service behavior is the working knowledge of industrial materials practice.

## Mechanical Behavior

**Stress-strain behavior** remains the foundation. Under uniaxial tensile testing, a ductile metal shows:
- **Elastic region**: stress proportional to strain (Hooke's law), fully recoverable. Slope = Young's modulus E.
- **Yield point**: onset of permanent deformation. Typically defined as 0.2% offset for metals.
- **Plastic region**: stress increases with strain due to work hardening.
- **Ultimate tensile strength (UTS)**: peak stress.
- **Necking and fracture**: localized deformation leads to failure.

Key properties:
- **Young's modulus (E)**: stiffness. Diamond ~1000 GPa, steel ~200 GPa, aluminum ~70 GPa, polymers 1-10 GPa, rubbers 0.001-0.1 GPa.
- **Yield strength (σy)**: onset of plastic deformation.
- **Tensile strength (σUTS)**: maximum stress.
- **Ductility**: elongation or reduction in area at fracture. Measure of how much deformation before failure.
- **Hardness**: resistance to indentation. Correlates roughly with yield strength for metals.
- **Toughness**: energy to fracture. Area under stress-strain curve, or specific measures like Charpy impact energy, fracture toughness KIC.

Materials differ enormously. Ceramics are stiff and strong but brittle (low toughness). Polymers are compliant, tough, ductile. Metals are intermediate on stiffness but often have excellent combination of strength and toughness.

## Fracture Mechanics

Understanding fracture requires going beyond simple stress-strain. Real materials contain defects — pores, inclusions, cracks. Failure initiates at defects.

**Linear elastic fracture mechanics (LEFM)** — Griffith, Irwin, 20th century — describes cracks in brittle materials. Stress intensity factor K characterizes the stress field near a crack tip:

$$K = Y \sigma \sqrt{\pi a}$$

Where σ is applied stress, a is crack length, Y a geometry factor. Fracture occurs when K exceeds material fracture toughness KIC.

KIC ranges:
- **Ceramics**: 1-10 MPa·m^½. Glass: 0.7. Silicon nitride: 5-10.
- **Metals**: 30-150 MPa·m^½. Steels: 50-100. Aluminum: 30-50. Titanium: 50-100.
- **Polymers**: 1-5 MPa·m^½.
- **Metal-matrix and fiber composites**: 20-100.

A fracture-tough material tolerates larger cracks before failure — critical for structures that contain inspectable defects.

**Ductile vs. brittle**: ductile materials deform plastically at crack tip, blunting it and absorbing energy. Brittle materials fracture with minimal deformation.

**Ductile-brittle transition**: many BCC metals (steels especially) become brittle at low temperature. Charpy V-notch testing at various temperatures identifies the transition temperature. Below it, steel that would tolerate defects at room temperature fractures catastrophically. Liberty ship fractures, Titanic hull plates, and many Arctic pipeline failures trace to this phenomenon.

## Fatigue

Most mechanical failures in service are **fatigue failures** — crack initiation and growth under cyclic loading below monotonic strength. Load rises and falls repeatedly; cracks nucleate and grow a tiny amount each cycle; eventually the crack reaches critical size and the component fractures.

**S-N (Wöhler) curves**: plot stress amplitude vs. cycles to failure. Most steels have an **endurance limit** — below which fatigue life is effectively infinite (~10⁸ cycles). Aluminum and many other materials lack a clear endurance limit; continue to fatigue at very low stresses.

**Paris law**: fatigue crack growth per cycle scales as:

$$\frac{da}{dN} = C (\Delta K)^m$$

Where ΔK is stress intensity range. m is typically 2-4 for metals, higher for brittle materials.

**Fatigue life factors**:
- **Stress concentrations**: notches, holes, sharp corners dramatically reduce fatigue life.
- **Surface finish**: rough surfaces initiate cracks earlier. Polishing, shot peening improve fatigue.
- **Residual stresses**: compressive residual stresses at surface (from shot peening, surface rolling, case hardening) delay crack initiation.
- **Environment**: corrosive environment accelerates fatigue (corrosion fatigue).
- **Mean stress**: tensile mean stress reduces fatigue life.

Most aircraft, automotive, rotating machinery, and structural design is fatigue-dominated. Design factors of safety account for fatigue uncertainty; periodic inspection (NDE) for crack detection is critical.

**Famous fatigue failures**: de Havilland Comet (pressurization cycles, square windows, 1950s), Aloha Airlines 737 (corrosion fatigue, 1988), rail car axles, steam turbine rotors. Each drove major changes in inspection and design.

## Creep

At elevated temperatures (typically >0.3-0.4 of absolute melting temperature), materials deform slowly under constant load — **creep**.

Three stages:
- **Primary creep**: decelerating strain rate as work hardening accumulates.
- **Secondary (steady-state) creep**: constant strain rate; most of service life.
- **Tertiary creep**: accelerating strain rate as damage accumulates, leading to rupture.

Steady-state creep rate follows:

$$\dot{\epsilon} = A \sigma^n \exp(-Q/RT)$$

Where A, n depend on mechanism (dislocation climb, grain boundary sliding, diffusion). Q is activation energy; low Q means creep accelerates rapidly with temperature.

Materials for high-temperature service (gas turbine blades, steam turbine rotors, nuclear reactor components):
- **Nickel-based superalloys**: gas turbine blades. Operate near 1100°C in single-crystal form.
- **Ferritic and austenitic stainless steels**: steam turbine rotors, boiler tubes.
- **Refractory metals**: W, Mo, Ta, Nb for extreme temperatures. Oxidation-sensitive.
- **Ceramics**: SiC, Si3N4 for burner liners, turbine blades (experimental).

Creep design uses time-to-rupture data and damage accumulation models (Larson-Miller parameter) to extrapolate short-term tests to decades of service.

## Corrosion

**Corrosion** is degradation of a material by chemical reaction with its environment. For metals, usually electrochemical:

- **Anodic reaction**: metal oxidizes (M → M^(n+) + ne-).
- **Cathodic reaction**: water/oxygen reduces.
- **Electrolyte**: carries ionic current between anode and cathode.

Types:
- **Uniform corrosion**: relatively uniform material loss. Predictable, manageable by corrosion allowance.
- **Galvanic corrosion**: two metals in contact with electrolyte; more active metal corrodes preferentially. Galvanic series ranks metals.
- **Pitting**: localized attack, small deep holes. Penetrates faster than uniform corrosion; hard to detect.
- **Crevice corrosion**: attack in confined spaces where oxygen is depleted.
- **Intergranular corrosion**: along grain boundaries. Sensitization of stainless steels a classic example.
- **Stress corrosion cracking (SCC)**: combined effect of tensile stress and specific corrosive environment. Chloride SCC in austenitic stainless steels, caustic SCC, ammonia in copper alloys.
- **Erosion-corrosion**: flow of corrosive fluid removes protective films, accelerating attack.
- **Hydrogen embrittlement**: hydrogen atoms absorbed into metal cause brittle failure under stress.

Mitigation:
- **Material selection**: stainless steels, nickel alloys, titanium for aggressive environments.
- **Protective coatings**: paint, galvanizing, metal plating, organic coatings.
- **Cathodic protection**: sacrificial anodes (zinc, magnesium) or impressed current. Standard for pipelines, ship hulls, storage tanks.
- **Inhibitors**: chemicals added to environment to reduce corrosion rate.
- **Design**: drain holes, avoid crevices, isolate dissimilar metals.

Global cost of corrosion is ~3-4% of GDP — trillions annually. Bridge deterioration, infrastructure maintenance, refinery downtime, pipeline failures.

## Wear

**Wear** is removal of material from surfaces by mechanical action. Types:
- **Adhesive wear**: sliding surfaces cold-weld at asperities; adhered fragments transfer or detach.
- **Abrasive wear**: hard particles or asperities plow through softer material.
- **Fatigue wear**: cyclic contact stresses cause subsurface crack formation and spalling. Gears, bearings.
- **Corrosive wear**: chemical action combined with mechanical removal.
- **Fretting**: small-amplitude oscillatory contact. Causes surface damage, crack initiation.

Wear-resistant materials:
- **Hardened steels, tool steels**: high hardness.
- **Cemented carbides (WC-Co)**: cutting tools, dies.
- **Ceramics**: Al₂O₃, SiC, Si₃N₄ for bearings, cutting tools.
- **Polymers** (PTFE, UHMWPE): low-friction, self-lubricating bearings.
- **Coatings**: hard chrome, TiN, DLC, thermal spray coatings.

**Tribology** — the science of friction, wear, and lubrication — integrates materials, surfaces, and lubricants. Key to engine, transmission, bearing, and cutting tool design.

## Environmental Interactions

Beyond corrosion, materials interact with environments in other ways:
- **Oxidation at high temperature**: forms oxide scales that may be protective (alumina on superalloys) or destructive (iron oxide scale).
- **Radiation damage**: neutrons in reactors displace atoms; accumulated damage embrittles pressure vessels.
- **UV degradation**: polymers degrade in sunlight. Stabilizers extend life.
- **Biological attack**: marine growth, bacterial corrosion (MIC), fungal degradation of organics.
- **Humidity effects**: swelling, plasticization of polymers; surface oxidation of metals.

Service environments rarely involve single degradation mechanisms. Realistic lifetime prediction requires understanding combined effects.

## Testing and Characterization

Standards-based testing (ASTM, ISO, DIN) supports materials selection and design:
- Tensile, compression, bending for basic mechanical.
- Hardness, impact for simple indicators.
- Fracture toughness (KIC per ASTM E399).
- Fatigue (S-N curves, crack growth rates).
- Creep (per ASTM E139).
- Corrosion (salt spray, immersion, electrochemical per many ASTM methods).

Non-destructive evaluation (NDE) for in-service inspection:
- **Visual**: first and often best.
- **Dye penetrant**: surface-breaking defects.
- **Magnetic particle**: surface and near-surface in ferromagnetic materials.
- **Ultrasound**: internal defects, thickness measurement.
- **Radiography**: X-ray or gamma ray imaging.
- **Eddy current**: surface cracks, thickness, conductivity changes.

Failed components are analyzed to identify failure mechanism, root cause, and corrective action. Failure analysis — fractography, metallography, chemical analysis — is a specialty connecting field experience to materials selection and design.

## Design Selection

Materials selection methodology (Ashby):
1. Define function and constraints.
2. Derive performance indices (e.g., stiffness/weight, strength/weight, toughness/weight).
3. Screen materials against constraints; rank by performance indices.
4. Consider supporting info (cost, availability, processability, reliability).
5. Iterate as design evolves.

Ashby charts plot material properties (E vs. density, strength vs. toughness, etc.) on log-log scales. Families of materials occupy distinct regions; performance indices appear as lines of constant slope; selection is visual.

Life-cycle cost, not just purchase cost, drives real decisions. Titanium may be 10× the cost of steel but last 30× as long in a chlorinated process, paying back many times over.

## Why This Level Matters

Materials failures in service kill people and cost billions. Every major industrial accident of the last century — structural collapses, boiler explosions, aircraft crashes, pipeline failures, reactor incidents — has a materials-failure component. Understanding how materials behave under the stress, temperature, time, and environmental conditions of actual service is what turns a theoretical design into a safe, durable component.

The intellectual core here is the coupling: microstructure from L2 processing drives the defects, interfaces, and metastability that control L3 service behavior. Predictive materials engineering — the ability to design for specific service requirements rather than picking from a catalog — is one of the most valuable skills in industrial practice.

## The Transition to Level 4

L4 turns to **advanced and emerging materials** — composites, nanomaterials, biomaterials, metamaterials, and the design-for-property approaches that are reshaping what's possible in aerospace, medicine, electronics, and energy.

Next: [L4 — Advanced Materials](./L4_Advanced_Materials.md) *(deferred)*
