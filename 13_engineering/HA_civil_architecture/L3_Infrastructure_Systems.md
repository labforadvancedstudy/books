# Level 3: Infrastructure Systems
*Roads, water, sewers, power lines, networks — the connective tissue of cities*

<!-- Evidence Tier: Textbook -->

## What Connects the Buildings

L2 covered buildings as enclosures. L3 covers what connects them. A building without roads, water supply, sewer, electricity, and communications is just a shelter. Civilization's built environment consists more of networks than of buildings. Most of the investment, most of the maintenance burden, and most of the failure modes of urban life are infrastructure. When infrastructure works, we don't notice it; when it fails (Flint water crisis, Texas grid failure 2021, California wildfires from utility equipment, Italian bridge collapse 2018), the consequences are immediate and severe.

This level surveys the main infrastructure networks — transportation, water supply, wastewater, stormwater, power, telecommunications — the engineering principles that govern them, and the system-level concerns (resilience, aging, funding, equity) that apply across.

## Transportation Networks

**Roads and streets** carry most urban movement — private cars, transit, freight, emergency, walking, cycling. Hierarchical classification:
- **Freeways / motorways / expressways**: controlled access, high speed, long distance.
- **Arterials**: through-routes across cities, moderate access.
- **Collectors**: feed arterials from neighborhoods.
- **Local streets**: access individual properties.

Design parameters: lane width, speed, grade, curve radius, sight distance, intersection type, right-of-way width. Standards (AASHTO in US, similar elsewhere) codify these.

**Pavement**:
- **Asphalt (flexible pavement)**: bituminous surface, gravel base. Flexibility accommodates ground movement. Life ~15-25 years with resurfacing.
- **Concrete (rigid pavement)**: Portland cement concrete. Stiffer; life ~30-40 years.
- **Composite**: concrete base with asphalt overlay.
- **Pervious pavements**: allow water infiltration; emerging for stormwater management.

Roads degrade from traffic (heavy vehicles disproportionate — 10,000 cars ≈ one semi in pavement damage), weather (freeze-thaw, UV), and poor drainage. Maintenance is chronic and under-funded in many jurisdictions.

**Bridges**: span gaps where grade can't.
- **Beam bridges**: short-medium spans.
- **Arch**: stone historically; steel/concrete modern. Moderate spans.
- **Truss**: steel framework, efficient material use. Moderate-long spans.
- **Suspension**: main cables over towers, deck suspended. Long spans (~1-2 km), iconic (Golden Gate, Akashi Kaikyō 1991 m main span).
- **Cable-stayed**: cables direct from tower to deck. Emerged 1960s, now dominant for medium-long spans (~300-1000+ m).

Bridges have lifetimes ~50-100+ years with maintenance. Inspection regimes (biennial in US post-1967 Silver Bridge collapse) identify deterioration. ~7% of US bridges currently classified structurally deficient; ~40% past 50-year design life.

**Tunnels**: underground or underwater transport. Boring machines (TBM) for soft ground, rock. Expensive but sometimes only alternative. Subway tunnels, road tunnels, rail tunnels, utility tunnels.

**Rail**: covered in HA_transport_systems L2. Within urban context, light rail, heavy rail metro, commuter rail constitute separate infrastructure with stations, yards, power systems (third rail or catenary), signaling.

**Airports**: runways, taxiways, terminals, aprons, ground access. Major land consumers and regional economic anchors.

**Ports**: quays, docks, cranes, container yards, hinterland connections. Container revolution reshaped global ports since 1960s.

## Water Supply

Water supply moves potable water from source to consumer.

**Sources**:
- **Surface water**: rivers, lakes, reservoirs. Large cities often rely — NYC Catskill/Delaware system, London Thames Water.
- **Groundwater**: aquifers tapped by wells. Most of agricultural supply globally; many rural and some urban supplies. Depletion widespread (L3 of HA_hydrology).
- **Desalination**: seawater treatment; growing in arid regions (Israel ~85% of domestic, Saudi Arabia major, Australia, California expanding).
- **Rainwater capture**: traditional, modern adaptations in water-stressed areas.
- **Water reuse / recycling**: treated wastewater returned to supply. Singapore NEWater, Orange County California, expanding elsewhere.

**Treatment** (surface water):
1. **Screening**: remove debris.
2. **Coagulation/flocculation**: add alum or ferric; particles clump.
3. **Sedimentation**: particles settle.
4. **Filtration**: through sand, anthracite, activated carbon.
5. **Disinfection**: chlorine, chloramine, ozone, UV.
6. **pH adjustment, corrosion control**.

Groundwater usually cleaner, requires less treatment but may need softening, iron/manganese removal.

**Distribution**: pressurized pipes (~60-100 psi / 4-7 bar) under streets. Materials: cast iron (older), ductile iron, steel, concrete, PVC, HDPE. Trunk mains from treatment to pressure zones, distribution mains, service lines to buildings.

**Storage**: elevated tanks and ground reservoirs provide pressure, peaking capacity, fire flow, and reserve. Typical 1-2 days supply.

**Losses**: 15-40%+ of treated water lost through leaks in many systems; ~50%+ in older cities (London historically, many developing cities). Investment in leak detection and main replacement critical but expensive.

**Lead**: older service lines (pre-1940s) often lead. Health risk recognized since Roman times; addressing is expensive. Flint MI crisis (2014-15) and similar in others highlights ongoing risk.

## Wastewater

Wastewater collection and treatment is the partner system.

**Collection**:
- **Separate sanitary sewers**: sewage only. Modern standard.
- **Combined sewers**: sewage + stormwater. Historic; overflow to rivers during storms (combined sewer overflows, CSOs). ~700 US cities still have some CSOs; elimination projects cost $100B+.
- **Pumping stations**: lift sewage where gravity insufficient.

**Treatment** (activated sludge, the dominant process):
1. **Primary**: sedimentation of solids.
2. **Secondary (biological)**: bacteria in aerated tanks consume organic matter.
3. **Tertiary**: nutrient removal (N, P), filtration, disinfection.
4. **Sludge handling**: digestion (anaerobic, produces biogas), dewatering, disposal (landfill, incineration, composting, land application).

Discharge to surface water at regulated concentrations (Clean Water Act NPDES permits in US, similar elsewhere).

**Nutrient pollution** (N, P) from wastewater (and agricultural runoff) causes eutrophication, dead zones (Gulf of Mexico, Chesapeake Bay). Upgraded treatment reduces nutrient loading but is expensive.

**Emerging contaminants**: pharmaceuticals, personal care products, microplastics, PFAS. Conventional treatment removes many; some pass through. Advanced treatments (activated carbon, ozonation, membrane) emerging.

**On-site wastewater** (septic tanks, leach fields): serves ~20% of US households, often rural. Failures contaminate groundwater and surface water.

## Stormwater

Stormwater management prevents flooding and protects water quality.

**Conveyance**:
- **Surface drainage**: gutters, ditches, swales.
- **Storm sewers**: pipes to receiving water.
- **Detention ponds / basins**: slow runoff, reduce peak flows.
- **Retention ponds / wetlands**: hold and filter.

**Green infrastructure** (increasingly integrated with gray):
- **Rain gardens / bioswales**: vegetation filters and infiltrates.
- **Permeable pavement**.
- **Green roofs**.
- **Urban trees**.

**Flood control**:
- **Levees / dikes**: raise riverbank.
- **Flood walls**.
- **Dams and reservoirs**.
- **Floodways**: designated areas to flood deliberately.
- **Nonstructural**: flood maps, building codes, buyouts.

**Climate change impacts**: more intense rainfall, rising sea levels. Designs based on historical statistics increasingly inadequate. Retrofit and redesign underway globally.

## Power Distribution

Covered in HA_energy L2 at systems level; infrastructure-level specifics:

**Transmission**: high-voltage (100-800 kV) long-distance overhead lines on towers. Sometimes underground in cities (expensive). HVDC links for long/subsea.

**Substations**: transform voltage, switch circuits, protect against faults. Distributed through network.

**Distribution** (medium voltage, 4-35 kV): from substation to neighborhood transformers. Overhead poles in most US; buried increasingly in new developments and as infill; predominantly buried in most of Europe and Japan.

**Service drops**: from pole/pad transformer to building.

**Meters**: measure consumption. Smart meters (AMI) increasingly standard; two-way communication enables demand response, outage detection.

**Resilience concerns**:
- **Extreme weather**: ice storms, hurricanes, wildfires; increasing with climate change.
- **Wildfires**: California utility equipment has ignited major fires (PG&E bankruptcy 2019 after Paradise fire).
- **Cyber**: grid digitization creates attack surface; actual incidents rare but concern real.
- **Physical attacks**: substation sabotage incidents rising.
- **Aging infrastructure**: much distribution equipment >40 years old; replacement investment needed.

## Telecommunications

Connectivity infrastructure has shifted from copper to fiber to wireless.

**Fiber optic**:
- Backbone between cities and countries.
- Municipal / utility fiber for institutions, businesses.
- Fiber-to-the-home (FTTH) increasing; dominant in Japan, Korea, parts of Europe; slower in US.
- High capacity (Tbps per fiber pair), low loss.

**Copper telephony**:
- Historic PSTN still serves many; declining.
- DSL variants extend broadband over copper.

**Coaxial cable**:
- Cable TV origins; DOCSIS provides broadband.
- Still major internet delivery in many regions.

**Wireless**:
- **Cellular** (3G/4G/5G): mobile voice/data. 5G deployment ongoing; small-cell densification for capacity.
- **Wi-Fi**: local; ubiquitous in buildings.
- **Satellite**: traditional geostationary for broadcast; LEO constellations (Starlink, OneWeb) for broadband. Transforming rural/remote connectivity.

**Data centers**: massive facilities housing servers. Power-hungry (~2% of US electricity), water-using (cooling). Locating near cheap/clean power and cool climates. AI training driving explosive growth 2022+.

## Solid Waste

Municipal solid waste management:
- **Collection**: trucks, curbside, drop-off, pneumatic systems (some dense urban districts).
- **Transfer stations**: consolidate for long-haul.
- **Landfilling**: dominant disposal in US. Methane capture; engineering for leachate, gas.
- **Incineration / waste-to-energy**: produces electricity; common in Europe, Japan; environmental concerns (air pollution, ash).
- **Recycling**: paper, metals, glass, some plastics. Markets volatile; China's 2018 ban on contaminated recyclables reshaped.
- **Composting**: organics diversion. Expanding.
- **Hazardous waste**: separate stream, strict handling.

Zero-waste aspirations vs. material reality: only a fraction of waste is practically recyclable with current technology and markets.

## Natural Gas

Distribution of methane to buildings and industry:
- **Transmission pipelines**: high pressure (~1000 psi), cross-country.
- **Distribution mains**: city gates reduce pressure to ~60 psi for neighborhood mains, then ~0.5 psi service lines.
- **Leaks**: methane a potent greenhouse gas; leak rates from pipeline systems matter for climate math.
- **Safety**: explosive gas; odorant (mercaptan) added; leak detection critical.

**Transition pressure**: policies increasingly encouraging building electrification and moving off gas; political debates intense (bans vs. choice, cost impacts on existing customers).

## Integrated Corridors

Infrastructure tends to cluster in public right-of-way:
- Underground: water, sewer, gas, electric, fiber.
- Surface: roads, sidewalks, stormwater.
- Above: overhead power, communication, transit catenary.

**Utility coordination** critical: "one-call" services (811 in US) before digging; conflicts cost time and risk.

**Dig once** policies: during road work, coordinate all utility upgrades. Efficiency gains significant.

## System-Level Concerns

**Resilience**: infrastructure designed for single-threat failures increasingly challenged by cascading failures (hurricane + flood + extended outage + communication loss). Design for multi-hazard; redundancy and quick recovery.

**Aging infrastructure**: much US infrastructure built 1930s-1970s; past design life. American Society of Civil Engineers 2021 Report Card: overall C- grade; $2.6T investment gap over decade. Similar stress in UK, much of Europe.

**Funding**: infrastructure investment politically difficult — upfront costs, delayed benefits, diffuse beneficiaries. US Infrastructure Investment and Jobs Act (2021) authorized $1.2T; EU NextGenerationEU similar scale; China invests more consistently as share of GDP.

**Climate adaptation**: roads flooding, coastal infrastructure at risk, heat stress on pavement/rails/equipment, wildfires, extreme precipitation. Retrofit plus new designs with future climate in mind.

**Equity**: infrastructure quality and access correlate with income, race. Redlining, urban renewal displacement, ongoing maintenance disparities documented. Infrastructure bill explicitly includes equity considerations; implementation varies.

**Operations and maintenance**: often the poor cousin of new construction in funding but dominates lifecycle costs. "Fix it first" policies vs. new-build priorities.

## Smart Infrastructure

Digital overlay on physical:
- **Sensors**: water flow, pressure, quality; pavement conditions; bridge health; grid state.
- **Control systems**: traffic signal coordination, water pressure optimization, grid automation.
- **Asset management software**: tracking condition, prioritizing maintenance.
- **Citizen reporting**: 311 apps, digital engagement.
- **Digital twins**: model infrastructure for simulation, planning.

Cybersecurity: digital overlay creates attack surface. Utilities especially targets; incidents from denial-of-service to ransomware to state-actor intrusions.

## Why This Level Matters

Infrastructure is the substrate of urban life and economic activity. It's simultaneously taken for granted and chronically underinvested. Failures are sometimes spectacular (bridge collapses, water crises, blackouts) but more often chronic (pothole-ridden roads, leaky pipes, aging power systems) with cumulative costs that aren't salient until the bill comes due.

Engineers who work on infrastructure are shaping systems that will serve generations. Decisions about pavement life, pipe materials, grid design, transit alignments lock in patterns that persist for decades to centuries. Doing this well requires technical skill, political wisdom, equity consciousness, and long time horizons.

## The Transition to Level 4

L4 turns to **urban planning and sustainable cities** — the design of urban form, zoning, density, land use, and the comprehensive approach to making cities work as places of life and prosperity for billions.

Next: [L4 — Urban Planning & Sustainable Cities](./L4_Urban_Planning_and_Sustainable_Cities.md) *(deferred)*
