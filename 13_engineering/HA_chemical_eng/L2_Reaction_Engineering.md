# Level 2: Reaction Engineering
*Kinetics, thermodynamics, and the sizing of reactors that turn feedstock into product*

<!-- Evidence Tier: Textbook -->

## What Chemical Reactors Do

L1 covered mass and energy balances — the bookkeeping of chemical processes. L2 turns to the heart of what makes chemical engineering distinct: the **reactor**, where chemical transformations occur. A refinery, a fertilizer plant, a pharmaceutical facility, a polymer factory, a brewery — all are organized around one or more reactors. Everything upstream prepares feed; everything downstream separates and purifies product. The reactor is where the molecule changes.

Reactor design sits at the intersection of three sciences: **thermodynamics** (what is possible — equilibrium, heat of reaction, temperature dependence), **kinetics** (how fast — rate laws, catalysis, temperature and concentration effects), and **transport** (mass, momentum, heat transfer that controls how reactants reach the reactive site and how heat is added or removed). Good reactor engineering requires holding all three together.

## Thermodynamic Framing

Before rate, a reaction engineer checks whether the reaction is thermodynamically allowed. The Gibbs free energy change ΔG determines equilibrium:

$$\Delta G = \Delta G^\circ + RT \ln K$$

Where K is the equilibrium constant. At equilibrium, ΔG = 0 and K is determined by ΔG°/RT. Reactions with strongly negative ΔG° go essentially to completion; near-zero ΔG° gives equilibrium-limited conversion.

**Heat of reaction** (ΔH) distinguishes:
- **Exothermic**: releases heat. Classic industrial reactions — ammonia synthesis, methanol synthesis, oxidation reactions.
- **Endothermic**: absorbs heat. Steam reforming, cracking, calcination.

Exothermic reactions need heat removal to avoid runaway; endothermic need heat addition to proceed. The heat-transfer strategy often dominates reactor design.

**Temperature and equilibrium**: exothermic equilibria are favored at low temperature (Le Chatelier); endothermic at high. But rates are faster at high temperature. The tension produces an **optimum temperature profile** — often impossible to achieve without staged reactors or temperature-controlled catalysts.

**Pressure**: reactions with decreasing moles (ammonia: N₂ + 3H₂ → 2NH₃) are favored by high pressure; reactions with increasing moles disfavored. Haber-Bosch runs at 150-300 bar for this reason.

## Kinetics

**Rate law** expresses rate as a function of concentrations, typically:

$$r = k [A]^a [B]^b \cdots$$

Where k is the rate constant, and exponents are reaction orders (not necessarily equal to stoichiometric coefficients — orders are empirical or derived from mechanism).

**Arrhenius temperature dependence**:

$$k = A e^{-E_a / RT}$$

Activation energy E_a determines sensitivity to temperature. A 10°C rise typically doubles rate for reactions with E_a around 50-80 kJ/mol. Many reactions are strongly temperature-dependent — runaway is a real hazard.

**Elementary reactions** have orders equal to stoichiometric coefficients and mechanism follows the equation. Most industrial reactions are multi-step; observed rate laws reflect the rate-determining step in a mechanism. Mechanistic models predict behavior outside measured ranges better than empirical power laws.

## Catalysis

Most industrial reactions use **catalysts** — substances that accelerate reactions without being consumed. Without catalysis, most industrial reactions would be too slow at manageable temperatures, and most that do proceed would produce unwanted byproducts instead of target products.

Catalyst types:
- **Heterogeneous**: solid catalyst, gas or liquid reactant. Ease of separation is major advantage. Examples: zeolites in catalytic cracking, supported metals in hydrogenation, iron in ammonia synthesis.
- **Homogeneous**: dissolved in reaction mixture. Higher activity and selectivity often; separation is harder. Examples: soluble metal complexes in polymerization, acid catalysis in organic chemistry.
- **Biocatalysts** (enzymes): extreme selectivity, mild conditions, but sensitive to temperature and pH. Used in pharma, food, increasingly in chemicals.

**Selectivity** — the fraction of converted reactant going to desired product vs. byproducts — often matters more than pure activity. A 99% selective catalyst with moderate activity beats a 100% active but 85% selective one, because the 15% byproducts must be separated and disposed of (expensive) or sold at discount.

**Catalyst deactivation**: fouling, poisoning, sintering, coking. Regeneration cycles (periodic shutdown and catalyst refresh) are standard for many processes. Fluid catalytic cracking circulates catalyst continuously between reactor and regenerator — a testament to how aggressive deactivation can be.

## Ideal Reactors

Three idealized reactor types bracket most real systems:

**Batch reactor**: feed loaded at start, reactions proceed, products unloaded at end. No inflow or outflow during reaction. Concentration varies with time. Used for small-scale, flexible, high-value products (specialty chemicals, pharmaceuticals, fermentation).

Design equation: moles and energy balances over time. For a single-reaction constant-volume batch reactor:

$$t = N_{A0} \int_0^X \frac{dX}{-r_A V}$$

Where X is conversion. The integral is the design equation — residence time needed for target conversion.

**Continuous stirred-tank reactor (CSTR)**: continuous flow in and out, contents perfectly mixed. Reactor composition equals exit composition. Used for liquid-phase reactions at scale — easy temperature control, simple construction.

Design equation: moles and energy balances at steady state. For a single reaction:

$$V = \frac{F_{A0} X}{-r_A(X)}$$

Rate is evaluated at exit concentration — usually low in a high-conversion CSTR. This makes CSTRs large for reactions with positive order in reactant; but forgiving for runaway and easy to control.

**Plug flow reactor (PFR)**: continuous flow with no axial mixing. Composition varies with position along the reactor tube. Used for gas-phase and most high-throughput processes.

Design equation:

$$V = F_{A0} \int_0^X \frac{dX}{-r_A(X)}$$

Rate decreases along the reactor as reactant is consumed, but starts high at the inlet. PFRs are generally smaller than CSTRs for the same conversion, at the cost of harder temperature control.

## Real Reactors

Real systems are combinations, variants, and compromises:

**Tubular packed-bed reactor**: solid catalyst packed in tubes; gas flows through. Approximates PFR with heterogeneous catalysis. Workhorse of petrochemicals, ammonia, methanol, SO₂ oxidation. Tubes are small diameter (25-100 mm) for radial heat transfer; reactor contains thousands of tubes.

**Fluidized-bed reactor**: catalyst particles suspended by upward gas flow. Excellent heat transfer; catalyst can circulate for regeneration. Used in fluid catalytic cracking, propylene ammoxidation, catalyst regeneration.

**Slurry reactor**: fine catalyst suspended in liquid; gas bubbled through. Used in Fischer-Tropsch synthesis, hydrogenations, liquid-phase oxidations.

**Bubble column**: gas bubbled through liquid with no stirring. Simple, limited for reactions requiring intense mass transfer.

**Membrane reactor**: reaction and separation combined; e.g., hydrogen-permeable membrane shifts equilibrium by removing product.

**Microreactor**: millimeter-scale channels; high surface-to-volume ratio, excellent heat transfer, safe for hazardous reactions. Used in pharma process development; scale-up via numbering up.

## Selectivity, Yield, Conversion

Key performance metrics:

- **Conversion (X)**: fraction of feed reactant consumed.
- **Selectivity (S)**: moles of target product per mole of reactant reacted.
- **Yield (Y)**: moles of target product per mole of feed = X × S.

Maximizing yield is the usual objective. But it's tension-filled:
- Higher conversion often means lower selectivity (desired product reacts further, or reactant runs into byproduct pathways).
- Higher temperature usually raises rate but may favor byproducts.
- Higher catalyst loading may improve selectivity but increases cost.

Reactor operating point balances these. The economic optimum is often at moderate conversion with recycle of unreacted feed — the cost of separation and recycle traded against the cost of low selectivity at higher conversion.

## Heat and Mass Transfer

Reactors are not just places where reactions happen — they're also heat exchangers and mixers. For many reactors, transport limitations, not intrinsic kinetics, set the rate.

**Heat transfer**:
- Wall-cooled tubular reactors: heat removed through the tube wall.
- Interstage cooling: exothermic reaction split across multiple catalyst beds with coolers between.
- Adiabatic with recycle: reactor runs adiabatically; product gas cooled and recycled.

Temperature profile design is a major part of reactor engineering. Hotspots can cause runaway or catalyst damage; cold zones reduce rate and conversion.

**Mass transfer**:
- Gas-liquid: bubble size, interfacial area, liquid-phase coefficients.
- Gas-solid and liquid-solid: external diffusion to catalyst particle; internal diffusion in porous catalyst.

**Thiele modulus** characterizes the importance of internal diffusion in catalyst particles — when Thiele modulus is large, reaction is diffusion-limited and effectiveness factor is less than 1. Small particles give higher effectiveness but higher pressure drop.

## Safety

Reactors are high-hazard equipment. Hazards:

- **Runaway**: exothermic reaction generates heat faster than it's removed, raising temperature, accelerating reaction further. Temperature spikes to hundreds of degrees in seconds; vessels can rupture.
- **Explosion**: flammable gases at elevated pressure; oxidizers; decomposition reactions.
- **Toxic release**: many reactants and products are toxic (HF, chlorine, phosgene, HCN).
- **Mechanical failure**: high pressure, temperature cycling, corrosion.

Safety engineering:
- **Relief valves and rupture disks**: open if pressure exceeds design.
- **Emergency quench**: inject cold solvent or reaction inhibitor to stop runaway.
- **Dump tank**: large vessel to dilute contents.
- **Inherent safety**: smaller reactor inventory, lower pressure, substituted less-hazardous chemistry. The safest reactor is a small one.
- **HAZOP (hazard and operability studies)**: systematic review of possible deviations and consequences.
- **Layer of protection analysis (LOPA)**: estimate frequency of incidents and required mitigation layers.

Bhopal (1984) killed thousands through methyl isocyanate release — reactor safety failure compounded by failed containment and emergency response. Industry safety culture has improved enormously since; major incidents still occur.

## Scale-Up

A reactor that works in the lab rarely scales directly. Challenges:

- **Heat transfer**: surface-to-volume ratio drops with scale. A 1 L beaker can hold a mildly exothermic reaction isothermal; a 10 m³ reactor of the same reaction may run away.
- **Mixing**: mixing time scales with reactor volume; at scale, concentration gradients develop.
- **Residence time distribution**: non-ideal flow appears at scale — channeling, dead zones, backmixing.
- **Catalyst performance**: what works on 1 g of catalyst in a 10 cm column may behave differently in a 10 m bed with its pressure drop, heat transfer, and flow patterns.

**Scale-up strategies**:
- Geometric similarity with dimensionless number matching (Reynolds, Peclet).
- Pilot plant at intermediate scale.
- Detailed CFD modeling of full-scale reactor.
- Number-up of microreactors instead of scale-up.

Process development runs $10-100 million and years to take a lab reaction to commercial scale. Skipping or shortening it is a common source of plant startup problems.

## Why This Level Matters

Reactor engineering is the distinctive core of chemical engineering as a discipline. Every major commodity chemical — ammonia, ethylene, methanol, sulfuric acid, nitric acid, the building blocks of polymers and pharmaceuticals — is produced in reactors designed on principles at this level. Modern life depends on the hundred-million-tonne scale of reactions driven by Haber-Bosch, Contact process, steam reforming, FCC, polyolefin reactors.

The craft extends beyond commodity chemistry. Pharmaceutical process development, advanced materials synthesis, biofuels, electrochemistry, fuel cells, batteries, hydrogen production — all are reactor-engineering problems at their heart. Understanding how to size, operate, and scale reactors remains one of the highest-leverage skills in applied chemistry and materials.

## The Transition to Level 3

L3 turns to **separations and process design** — the downstream operations that purify reactor product, recover unreacted feed, and combine unit operations into a full process. Reactors alone don't make products; full processes do, and process economics usually pivot on the separation train.

Next: [L3 — Separations & Process Design](./L3_Separations_and_Process_Design.md) *(deferred)*
