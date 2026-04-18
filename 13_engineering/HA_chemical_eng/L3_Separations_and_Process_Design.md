# Level 3: Separations and Process Design
*Pulling products from mixtures and assembling unit operations into complete processes*

<!-- Evidence Tier: Textbook -->

## After the Reactor

L2 covered reaction engineering — getting the desired chemistry to happen. L3 covers what happens next. In virtually every chemical process, the reactor outlet is a mixture: the desired product plus unreacted feed, byproducts, catalyst debris, solvents, and impurities. Separating these into pure product, recycling the unreacted feed, and disposing of waste is usually where most of the equipment, capital, and energy of the process live.

A rule of thumb: reactors are ~20% of a typical process's cost; separations are ~50-70%. Distillation alone consumes ~10-15% of industrial energy worldwide. Process economics pivot on the separation train more than on the reactor for most commodity chemicals.

This level covers the major separation methods, the principles behind them, and the discipline of process synthesis — assembling unit operations into a coherent process that is safe, economical, and operable.

## Why Separations Matter

**Thermodynamics of separation**: separating a mixture is the reverse of mixing, which is naturally spontaneous. Separation requires work — energy that creates order from disorder. The theoretical minimum work for separating ideal mixtures is given by the Gibbs free energy of mixing. Real separations require much more than this minimum because of inefficiencies.

**Driving forces**: every separation exploits a property difference between components.
- **Volatility** (vapor pressure): distillation.
- **Solubility**: absorption, extraction, crystallization.
- **Size**: filtration, membrane separations.
- **Charge**: ion exchange, electrodialysis.
- **Affinity for surface**: adsorption, chromatography.
- **Density**: centrifugation, gravity separation.

The bigger the property difference, the easier (and cheaper) the separation.

## Distillation

The workhorse of industrial separations, using differences in volatility. Basic operation:

Heat a liquid mixture to generate vapor. Vapor is richer in the more volatile component. Condense vapor to get a more concentrated product of that component. Remaining liquid is richer in the less volatile component.

**Single flash** provides a partial separation. Multistage distillation (a column with many trays or a packing equivalent) dramatically improves purity.

**Distillation column basics**:
- **Feed** enters mid-column.
- **Vapor** rises; **liquid** flows down — countercurrent contact.
- **Reboiler** at bottom: supplies heat, generates vapor.
- **Condenser** at top: removes heat, condenses vapor.
- **Reflux**: part of condensed top product returned to column.
- **Distillate**: overhead product.
- **Bottoms**: bottom product.
- **Trays or packing**: provide vapor-liquid contact area.

**Number of trays**: more trays → better separation, at cost of capital. **Reflux ratio**: higher reflux → better separation, at cost of energy. Design optimizes between.

**McCabe-Thiele method**: graphical construction using equilibrium curve and operating lines to determine number of theoretical stages for binary mixtures.

**Relative volatility** α: ratio of K-values (K = y/x, vapor/liquid mole fractions). Higher α means easier separation. Typical:
- Propane-butane separation: α ~2-3. Easy, few stages.
- Benzene-toluene: α ~2.5. Straightforward.
- Ethanol-water: α ~2 at low ethanol, falls to 1 at azeotrope (95.6% ethanol). Azeotrope limits.
- Isomers of xylene: α ~1.03-1.05. Very hard; hundreds of stages needed.

**Azeotropes**: mixtures where vapor and liquid have same composition; straight distillation cannot separate. Break via:
- **Pressure swing distillation**: azeotrope composition shifts with pressure; use two columns.
- **Extractive distillation**: add third component that changes relative volatilities.
- **Azeotropic distillation**: form ternary azeotrope that can be separated.
- **Membranes / adsorption**: non-distillation methods.

**Practical column considerations**:
- Diameter sized for vapor velocity (flooding limits at high, weeping at low).
- Tray efficiency 50-90% typical — real stages require more than theoretical.
- Packing (structured or random) alternative to trays; lower pressure drop, better for small columns, vacuum, heat-sensitive materials.
- Heat integration: use overhead vapor to preheat feed, use bottoms to reboil elsewhere.

**Types**:
- **Continuous distillation**: dominant for commodity chemicals.
- **Batch distillation**: small volumes, multiple products from same column. Common in specialty chemicals, pharmaceuticals.
- **Extractive, azeotropic, reactive**: specialized.

**Energy efficiency**:
- Heat-pumped distillation: use the column top and bottom as heat source and sink of a refrigeration cycle. Halves energy for close-boiling separations.
- Dividing wall columns: combine two columns in one. Capital and energy savings.
- Thermally coupled columns (Petlyuk): similar concept.

Distillation is ~50%+ of total separations energy globally. Replacing or improving it where possible is a target of decarbonization efforts.

## Absorption and Stripping

**Absorption**: gas component dissolves into liquid solvent.
- **Physical absorption**: Henry's law governs; e.g., CO₂ in water under pressure.
- **Chemical absorption**: reactive solvent; e.g., amine solutions absorbing CO₂ or H₂S.

**Stripping**: reverse — dissolved gas removed from liquid into gas stream.

Typical applications:
- **Natural gas sweetening**: remove H₂S, CO₂ with amines.
- **Scrubbing**: SO₂, NOₓ, HCl from flue gas.
- **CO₂ capture from flue gas**: major current focus; amines dominant but have limitations.
- **Ammonia recovery**.
- **VOC removal from vent streams**.

Design similar to distillation: countercurrent column, mass-transfer equilibrium staged. Kremser equation for absorbers.

## Liquid-Liquid Extraction

Partition between two immiscible liquid phases.
- **Aqueous phase + organic solvent**: separate components that distribute differently.
- **Example**: extracting penicillin from fermentation broth into organic solvent for purification.
- **Example**: phosphate removal from nuclear waste using tributyl phosphate (PUREX process).
- **Example**: in oil refining, extracting aromatics with sulfolane.

Operated in mixer-settlers, pulsed columns, centrifugal contactors.

Commonly used when distillation difficult (heat-sensitive, close boiling, azeotropes).

## Crystallization

Formation of solid crystalline phase from solution or melt. Separates by:
- **Solubility differences**: one component crystallizes, others stay dissolved.
- **Cooling**: reduce solubility.
- **Evaporation**: concentrate solution.
- **Antisolvent**: add solvent in which product insoluble.
- **Reactive crystallization**: form new compound that precipitates.

Key parameters:
- **Supersaturation**: driving force for crystal growth/nucleation.
- **Crystal size distribution**: affects downstream filtration, drying.
- **Polymorphism**: different crystal forms of same compound; dramatic effect in pharmaceuticals (e.g., ritonavir famously discovered to have more stable polymorph that didn't dissolve properly, requiring reformulation).
- **Chirality**: racemic vs. enantiopure; for drugs single enantiomer often critical.

Applications: salt production (evaporation), sugar, pharmaceutical APIs, chemicals.

## Filtration and Centrifugation

Mechanical separation by size or density:

**Filtration**: particles retained on porous medium while fluid passes.
- **Depth filters, surface filters, membrane filters**.
- **Cake filtration**: filter cake provides additional filtration.
- **Crossflow filtration**: reduces cake buildup.
- Applications: catalyst separation, crystal recovery, beverage clarification.

**Centrifugation**: density differences magnified by centrifugal force.
- **Sedimentation centrifuges**: separate solids from liquid.
- **Decanter centrifuges**: continuous solid/liquid separation.
- **Disk-stack centrifuges**: fine separation.
- **Hydrocyclones**: particle removal by centrifugal force without rotation.

## Adsorption

Binding of molecules to a solid surface. Properties:
- **Physisorption**: weak (van der Waals); easily reversible.
- **Chemisorption**: chemical bond; stronger; may require elevated temperature for regeneration.

**Adsorbents**:
- **Activated carbon**: high surface area; water and air treatment, gas cleanup.
- **Zeolites**: molecular sieves; natural gas drying, N₂/O₂ separation via pressure swing.
- **Silica gel**: drying, moisture control.
- **Ion exchange resins**: charged sites bind ions; water softening, pharmaceutical purification.

**Operations**:
- **Fixed-bed**: packed column; regenerate periodically.
- **Pressure swing adsorption (PSA)**: alternate high/low pressure; produces ~99% oxygen from air, H₂ from reforming.
- **Temperature swing**: regenerate by heating.
- **Chromatography**: continuous or pulsed; high resolution; analytical and preparative scales.
- **Simulated moving bed (SMB)**: approximates countercurrent adsorbent movement; para-xylene separation, sugar purification.

## Membrane Separations

Semi-permeable barriers that pass some species and retain others.

**Types by mechanism**:
- **Microfiltration (MF)**: removes particles 0.1-10 μm. Water treatment, beverages.
- **Ultrafiltration (UF)**: 10-100 nm. Protein concentration, wastewater.
- **Nanofiltration (NF)**: 1-10 nm. Water softening, brackish desalination.
- **Reverse osmosis (RO)**: dissolved ions and small molecules. Seawater desalination, water purification.
- **Gas separation**: natural gas (CO₂/CH₄), hydrogen recovery, air (N₂).
- **Pervaporation**: liquid feed, vapor permeate; dehydration, ethanol dewatering.

**Modules**: hollow fiber, spiral wound, plate-and-frame, tubular.

**Fouling**: major challenge; particles, bacteria, scale block membrane; requires pretreatment and regular cleaning.

Membranes are growing share of separations — often lower energy than distillation, but trade against membrane cost and fouling.

## Drying

Removing remaining moisture from solids to specified product moisture:
- **Tray dryers, belt dryers**: batch or continuous; low temperature.
- **Rotary dryers**: large throughput; common for minerals.
- **Spray dryers**: liquid feed into hot gas; produces powders (milk powder, detergents, pharmaceuticals).
- **Fluidized bed dryers**: gas fluidizes particles; rapid drying.
- **Freeze dryers (lyophilization)**: vacuum + freezing; heat-sensitive products.

## Process Synthesis

Assembling unit operations into a complete process is **process synthesis**.

**Heuristics** developed by experienced engineers guide choices:
- Do the easy separations first.
- Remove corrosive components early.
- Don't separate further than required for downstream use.
- Recycle unreacted feed to improve yield.
- Use heat from hot streams to heat cold streams (heat integration).
- Minimize utility consumption (steam, cooling water, power).
- Consider safety: inventories of hazardous materials, runaway potential, pressure relief.

**Process flowsheet**: diagram showing major equipment and streams. Evolves from simple concept to detailed design.

**Heat integration (pinch analysis)**: systematically identifies minimum heating and cooling requirements. Can reduce energy by 20-50% in conventional designs.

**Computer-aided process synthesis** and optimization tools (Aspen Plus, Pro/II, gPROMS, HYSYS) enable exploring many alternatives quickly.

## Safety and Process Hazards

Chemical processes concentrate hazards: flammables, toxics, high pressure, high temperature. Major accidents (Bhopal 1984, Texas City 2005, Buncefield 2005, West Fertilizer 2013, Beirut 2020) have killed thousands collectively.

**Hazard analysis**:
- **HAZOP (Hazard and Operability Study)**: systematic brainstorming of deviations and consequences.
- **Fault tree analysis**.
- **Layer of protection analysis (LOPA)**: quantitative evaluation of safety instrumented systems.

**Inherent safety** principles (Trevor Kletz):
- **Intensify**: smaller inventories.
- **Substitute**: less hazardous materials.
- **Attenuate**: milder conditions.
- **Simplify**: fewer chances to fail.

**Safety systems**:
- **Pressure relief**: valves, rupture disks release overpressure to flare or safe location.
- **Emergency shutdown systems**.
- **Flares**: burn unavoidable hydrocarbon releases safely.
- **Containment**: berms, dikes for liquid spills.
- **Detectors**: gas, flame, fire.
- **Protective buildings** for operators.

**Culture**: process safety requires ongoing management attention, not just technology. Many disasters involved known risks where procedures weren't followed, warnings weren't heeded, or management cut corners.

## Economic Analysis

Process economics involves:
- **Capital costs (CAPEX)**: equipment, piping, instruments, installation, land, engineering. Order-of-magnitude estimates (~30-50% accuracy) in early design; detailed at final design.
- **Operating costs (OPEX)**: raw materials, utilities, labor, maintenance, waste disposal, taxes.
- **Revenue**: product sales less byproduct credits less waste disposal.
- **Discounted cash flow**: NPV, IRR, payback period.
- **Sensitivity analysis**: identifies which assumptions most affect economics.

Raw material costs often dominate for commodity chemicals. Energy costs substantial. Capital recovery typically modest share for mature commodities, large for novel or small-scale.

## Process Control

Processes must operate at desired conditions despite disturbances.
- **Measurements**: flow, pressure, temperature, composition, level.
- **Controllers**: PID most common; model-predictive control for complex units.
- **Actuators**: control valves, pumps, heaters.
- **Distributed control systems (DCS)**: unify plant control.
- **Cybersecurity**: growing concern as systems digitized and networked.

**Advanced process control (APC)**: model predictive control for multivariable, constrained optimization. Adds capacity, improves yield, reduces variability.

## Sustainable Process Design

Process design now often addresses:
- **Energy**: minimize consumption, electrify where possible, integrate renewables.
- **Water**: minimize consumption, recycle, treat discharges.
- **Waste**: minimize generation, recover byproducts where possible.
- **Emissions**: CO₂, air pollutants, fugitive emissions (leaks).
- **Materials**: reduce hazardous, consider lifecycle.
- **Circularity**: design for recyclability of products.

**Green chemistry** principles (Anastas & Warner): 12 principles including atom economy, less hazardous synthesis, safer solvents, energy efficiency, renewable feedstocks, degradation pathways.

## Why This Level Matters

Separations and process design are where laboratory chemistry becomes an operational, profitable plant. Every industrial chemical product exists because engineers figured out how to isolate it from reaction mixtures, purify it to specification, recycle unreacted inputs, manage safety hazards, and do all this profitably.

Transition priorities — decarbonization, circular economy, bioeconomy, green chemistry — play out largely at this level. Replacing a high-CO₂ process with a low-CO₂ alternative requires redesigning separations too; often separations become harder because new reactions have different byproducts. Carbon capture is itself a separation problem at massive scale. Battery recycling is a complex separation challenge. Water reuse is separation. Hydrogen purification for fuel cells is separation.

Process engineering is one of the fundamental disciplines for the physical economy's transformation. It's also one where decades of accumulated know-how matter — which is why experienced process engineers are scarce and in demand.

## The Transition to Level 4

L4 turns to **specialty chemicals and biotechnology** — the fine and specialty chemical industries (pharmaceuticals, specialty materials, food ingredients) and the growing intersection with biological processing, including fermentation, enzymatic synthesis, and the emerging synthetic biology and bioeconomy.

Next: [L4 — Specialty Chemicals & Biotechnology](./L4_Specialty_Chemicals_and_Biotechnology.md) *(deferred)*
