# Level 3: Spaceflight
*Rockets, orbits, satellites, and the engineering of leaving Earth*

<!-- Evidence Tier: Textbook -->

## Above the Air

L2 covered aircraft propulsion — engines that breathe atmospheric air. L3 turns to spaceflight, where there is no air. The fundamental shift: without atmospheric oxygen, the vehicle must carry both fuel and oxidizer; without atmospheric lift, the vehicle must be supported by its own thrust against gravity until orbital velocity is achieved; without atmospheric drag at orbital altitudes, vehicles can coast indefinitely on ballistic trajectories.

Space is a regime of extreme energies. Reaching low Earth orbit requires ~30 MJ/kg — about 7 times the energy per kg of TNT. Achieving this in a single stage is physically very hard given current materials and propellants; virtually all orbital launchers use multiple stages. Staying in space requires thermal, radiation, and micrometeoroid protection; operating there requires power, communications, and attitude control.

Spaceflight also is dominated by an equation — the Tsiolkovsky rocket equation — whose implications no engineering cleverness can escape. This level surveys the fundamentals: the rocket equation, orbital mechanics, launch vehicles, spacecraft subsystems, reentry, and the emerging commercial space industry.

## The Rocket Equation

**Tsiolkovsky rocket equation** (1903):

Δv = v_e · ln(m₀/m_f)

Where:
- Δv: total velocity change achievable ("delta-v budget").
- v_e: exhaust velocity of propellant = I_sp · g₀ (where I_sp is specific impulse in seconds, g₀ = 9.81 m/s²).
- m₀: initial mass (wet, with propellant).
- m_f: final mass (dry, propellant burned).

**Implications**:
- Δv is exponential in mass ratio. To double Δv requires squaring the mass ratio.
- Higher I_sp gives more Δv per mass of propellant — making propellant choice critical.
- Structure fraction limits ultimate performance. Even massless payloads need some structure and fuel tanks.

**Specific impulse values**:
- **Solid propellants**: I_sp 220-270 s. High thrust density, simple, storable but lower performance.
- **Liquid: kerosene + LOX** (RP-1): I_sp ~290-330 s. Most flown; RD-180, Merlin, F-1 (Saturn V first stage).
- **Liquid: hydrogen + LOX**: I_sp ~440-460 s. Highest chemical performance. RS-25 (Space Shuttle, SLS), RL10, Vulcain. Low density makes tanks larger.
- **Methane + LOX**: I_sp ~350 s. New focus: Raptor (SpaceX Starship), BE-4 (Blue Origin). Clean burning, manufacturable from CO₂ + H₂, storable enough for Mars operations.
- **Electric (ion)**: I_sp 1500-5000+ s. Very low thrust; used for satellite station-keeping and interplanetary.
- **Nuclear thermal (theoretical, some tests)**: I_sp 800-1000 s. High thrust possible; political and technical barriers.

**Delta-v budgets** (approximate):
- Earth surface to Low Earth Orbit (LEO): ~9.4 km/s (gravity + drag + orbital velocity).
- LEO to geostationary transfer orbit (GTO): ~2.5 km/s.
- GTO to geostationary orbit (GEO): ~1.5 km/s.
- LEO to Moon orbit: ~4 km/s.
- LEO to Mars orbit: ~4.5 km/s + aerobraking or propulsive capture.

With kerosene-LOX at ~330 s I_sp, a single-stage-to-orbit vehicle would need ~94% of mass to be propellant — leaving 6% for structure, engines, payload. Infeasible with real materials. Multi-staging drops empty tanks, raising achievable mass fraction.

## Orbital Mechanics Basics

**Kepler's laws** (applied to spacecraft):
1. Orbits are conic sections (ellipse, parabola, hyperbola).
2. Line from focus to spacecraft sweeps equal areas in equal times.
3. T² ∝ a³ (period squared proportional to semi-major axis cubed).

**Circular orbit velocity**: v = √(GM/r). At LEO (~400 km): ~7.7 km/s, period ~90 min.

**Escape velocity**: v = √(2GM/r). From Earth surface: ~11.2 km/s.

**Hohmann transfer**: two-burn elliptical transfer between circular orbits; most fuel-efficient for coplanar orbits.

**Orbit types**:
- **LEO (Low Earth Orbit)**: 200-2000 km. Most satellites, ISS, most Starlink. Short periods, atmospheric drag at lower altitudes.
- **MEO (Medium)**: 2000-35786 km. GPS (~20200 km), Galileo, GLONASS.
- **GEO (Geostationary)**: 35786 km. Orbits match Earth rotation — appears stationary. Communications, weather. Limited slots.
- **HEO (Highly Elliptical)**: Molniya orbits for high-latitude coverage.
- **Polar/Sun-synchronous**: pass over each point at same local solar time. Earth observation, reconnaissance.
- **Lagrange points**: L1, L2, etc. around Sun-Earth or Earth-Moon. JWST at L2, Gaia at L2.

**Orbital elements** (6 parameters fully specify orbit): semi-major axis, eccentricity, inclination, RAAN, argument of periapsis, true anomaly.

**Perturbations**: orbits are not fixed. J2 (Earth oblateness), atmospheric drag (at LEO), solar radiation pressure, third-body (Moon, Sun), thermospheric variations all perturb. Station-keeping burns counteract.

## Launch Vehicles

A launch vehicle lifts payload from ground to orbit. Components:

**Stages**:
- **First stage (booster)**: largest, lifts from surface. Sometimes assisted by solid rocket boosters (SRBs).
- **Second stage**: ignites after first stage drops; continues to orbit.
- **Upper stage**: often with additional engines for orbit circularization, higher orbits, or deep space.
- **Payload fairing**: aerodynamic shroud protecting payload during atmosphere; jettisoned.

**Major launch vehicles** (2020s):
- **Falcon 9** (SpaceX): kerosene-LOX, partially reusable (first stage lands). Dominant commercial launcher.
- **Falcon Heavy**: three Falcon 9 cores. Heavy lift.
- **Starship / Super Heavy** (SpaceX): methane-LOX, fully reusable goal. Most ambitious; test flights ongoing; massive payload capacity.
- **Atlas V / Vulcan Centaur** (ULA): legacy and new. Vulcan with BE-4 engine, methane-LOX.
- **Delta IV Heavy** (retiring): hydrogen-LOX, high-value payloads.
- **Ariane 5 / Ariane 6** (ESA): European workhorse; hydrogen-LOX core, solid boosters.
- **Long March series** (China): multiple configurations; increasingly capable.
- **Soyuz** (Russia): oldest operational design; crew and cargo to ISS historically.
- **H-IIA / H-III** (Japan).
- **PSLV / GSLV / LVM3** (India).
- **New Glenn** (Blue Origin): BE-4 engines, reusable first stage.
- **Neutron** (Rocket Lab, in development).
- **Electron** (Rocket Lab): small launch, kerosene-LOX.

**Trends**:
- **Reusability**: SpaceX's first-stage recovery (landing back) and Starship's goal of full reusability aim at orders-of-magnitude cost reduction.
- **Methalox**: methane-LOX increasingly standard for new designs.
- **Small-launch market**: Electron, Firefly, others for small dedicated launches.
- **Rideshare**: secondary payloads on larger launches.
- **Launch cost**: Falcon 9 ~$3-5k/kg to LEO (marginal); target for Starship <$500/kg or less if fully reusable.

## Spacecraft Subsystems

A spacecraft consists of the **bus** (vehicle providing utilities) plus **payload** (the reason for the mission).

**Structure and mechanisms**:
- Primary structure bearing loads.
- Deployables (solar arrays, antennas, booms, instruments).
- Separation mechanisms.

**Propulsion**:
- **Cold gas**: simple; low I_sp; small thrusters.
- **Monopropellant** (hydrazine): decomposes on catalyst. Common station-keeping.
- **Bipropellant**: small versions of launch engines. Larger maneuvers.
- **Electric propulsion**: ion, Hall-effect, arcjet. Long burns, high I_sp.

**Power**:
- **Solar arrays + batteries**: near Earth and inner solar system. Arrays from ~kW to 100+ kW (ISS).
- **Radioisotope thermoelectric generators (RTG)**: plutonium-238 decay heat to electricity. Deep space missions (Voyagers, Curiosity, Perseverance).
- **Batteries**: mostly Li-ion for spacecraft. Cycle life concerns.

**Thermal control**:
- **Passive**: coatings, radiators, multilayer insulation (MLI — the gold-foil-looking layers).
- **Active**: heaters, heat pipes, fluid loops, louvers.
- Space is not just cold — facing the Sun can drive surfaces hot; facing deep space drives cold. Managing both is the challenge.

**Attitude determination and control (ADCS)**:
- **Sensors**: star trackers, sun sensors, earth sensors, gyroscopes, magnetometers.
- **Actuators**: reaction wheels (momentum storage, spin to torque spacecraft), thrusters (desaturate wheels, large changes), magnetic torquers (react against Earth's magnetic field).
- Modern spacecraft hold pointing to arcseconds or better.

**Communications**:
- **Antennas**: high-gain for primary link, low-gain for omnidirectional emergency.
- **Bands**: S, X, Ka for different uses; optical (laser) communications emerging.
- **Deep Space Network** (NASA, 70 m dishes) for missions beyond GEO.
- **Data handling**: onboard storage, compression, error correction.

**Command and data handling**: computers, software, autonomy. Radiation-hardened for space environment.

**Life support** (crewed): atmosphere, water recycling, waste management, food storage, radiation shielding. Most complex subsystem for long-duration human missions.

## The Space Environment

Challenges:

**Radiation**: cosmic rays, solar particle events, Van Allen belts. Damages electronics, DNA. Shielding and rad-hard components required. For humans, long-duration missions (Mars transit) pose health risks.

**Vacuum**: outgassing, materials behavior, no convective cooling.

**Thermal extremes**: -150°C to +150°C on typical surfaces.

**Micrometeoroids and debris**: small particles at high relative velocity. Shielding (Whipple bumper) on crewed vehicles. Debris cloud in LEO worsening (Kessler syndrome concerns; Chinese 2007 ASAT test, Indian 2019 ASAT, Russian 2021 ASAT all generated debris).

**Plasma**: charging can damage electronics.

## Reentry

Returning to Earth requires converting orbital kinetic energy (~30 MJ/kg at LEO) into heat and dissipating it without destroying the vehicle.

**Ablative heat shield**: material chars and carries heat away in vaporizing products. Apollo, Orion, Crew Dragon (PICA-X), Starship (ceramic tiles + ablatives).

**Reusable thermal protection**: Space Shuttle tiles, Starship stainless steel + ceramic tiles. Reuse challenges are real; every flight damages some tiles.

**Entry angle**: critical. Too shallow — skip off atmosphere. Too steep — excessive heating and G-loads. Control via reaction control and lifting body shape.

**Hypersonic regime**: plasma around vehicle blocks communications (radio blackout). Lift:drag ratios of 0.3-1.0 typical for blunt capsules; higher for winged reentry vehicles.

**Recovery**: parachute into ocean (Crew Dragon, Starliner, Orion), parachute onto land (Soyuz), runway landing (Shuttle), propulsive landing (Starship plan, Boosters already).

## Crewed Spaceflight

Human presence in space:
- **Early**: Gagarin 1961, Mercury/Gemini/Apollo 1960s. Moon landings 1969-72.
- **Space Shuttle 1981-2011**: reusable partial system; 135 missions; two losses (Challenger, Columbia).
- **Mir 1986-2001**: first long-duration space station.
- **International Space Station 2000-present**: continuous crew since Nov 2000. Multi-national cooperation.
- **Chinese Tiangong**: operational since 2022.
- **Commercial crew**: SpaceX Crew Dragon since 2020 to ISS; Boeing Starliner behind schedule but emerging.
- **Private orbital flights**: Inspiration4 (2021), multiple since. Commercial market nascent.
- **Suborbital tourism**: Virgin Galactic, Blue Origin. Niche market.
- **Lunar return plans**: NASA Artemis (delayed), Chinese lunar program both targeting late 2020s/2030s.
- **Mars**: aspirational; SpaceX Starship development aimed at enabling; crewed Mars missions 2030s at earliest, more likely later.

Human physiology challenges: microgravity effects on bones, muscles, fluid shifts, vision; radiation; isolation psychology. Decades of ISS data guides countermeasures but long missions to Mars remain challenging.

## Satellites and Applications

Commercial and scientific satellites define much of spaceflight's value:

**Communications**:
- **GEO satellites**: broadcast TV, telephony, data. Slots valuable and regulated.
- **LEO constellations**: Starlink (>6000 satellites by 2024), OneWeb, Amazon Kuiper. Lower latency, global coverage.
- **IoT / M2M** smallsat constellations.

**Earth observation**:
- **Weather**: polar (NOAA, EUMETSAT) and geostationary (GOES, Meteosat).
- **Commercial imaging**: Planet (small sats, daily coverage), Maxar (high-resolution), Airbus, ICEYE (radar).
- **Government reconnaissance**: classified but very substantial capability.

**Navigation**:
- **GPS** (US), **Galileo** (EU), **GLONASS** (Russia), **BeiDou** (China). Global coverage; critical infrastructure.

**Science**:
- **Hubble Space Telescope** (1990-present, still operating).
- **JWST** (2022+): infrared, L2 orbit.
- **Mars rovers** (Curiosity 2012, Perseverance 2021).
- **Outer planet missions**: Juno (Jupiter), Europa Clipper, Dragonfly.
- **Sun observation**: Parker Solar Probe, Solar Orbiter.

## Commercial Space

Space economy ~$500B in 2024, estimated $1T+ by 2040. Sectors:
- **Satellite services** (largest): telecom, TV, IoT, data.
- **Earth observation data**.
- **Launch services**.
- **Manufacturing of spacecraft and components**.
- **Ground equipment** (user terminals, stations).
- **Government (civil + military)**.
- **Emerging**: space tourism, in-space manufacturing, space debris removal, space-based solar power (long-term), asteroid mining (speculative).

**SpaceX** dominance: ~60-80% of global commercial launch by mass in 2023-24. Reusability advantage structural.

**New entrants**: Rocket Lab, Blue Origin, Relativity, Firefly, Chinese firms, European start-ups.

**Regulatory**: ITU for frequency/orbital slot allocation, national space agencies licensing launch, national export controls (ITAR in US).

## Why This Level Matters

Spaceflight is the youngest engineering discipline among those at L3. Its first 70 years produced astonishing achievements — landing on the Moon, continuous human presence in orbit, robotic exploration of every planet in the solar system, global networks providing navigation and communications — and also long periods of stagnation. Reusable rockets and falling launch costs in the 2010s-2020s opened a new era.

Space now permeates daily life — GPS, weather, communications — in ways most people don't realize. It's also a domain of state power, scientific discovery, and commercial ambition. Extension to crewed lunar return, Mars missions, space-based infrastructure (solar power, manufacturing, data centers) is plausibly ahead in coming decades.

Engineering for space remains demanding: unforgiving environment, no in-situ repair for most missions, long development cycles, limited test opportunities. But costs and cycle times are falling. The transition from "space is for governments" to "space is commercial infrastructure" is underway.

## The Transition to Level 4

L4 turns to **aviation safety and air traffic management** — the systems that make aviation one of the safest forms of transportation, and the coordination that enables tens of thousands of daily flights to operate safely in shared airspace.

Next: [L4 — Aviation Safety & Air Traffic](./L4_Aviation_Safety_and_Air_Traffic.md) *(deferred)*
