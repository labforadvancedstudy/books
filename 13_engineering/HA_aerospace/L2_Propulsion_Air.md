# Level 2: Propulsion (Air)
*Jet engines, piston engines, and how aircraft convert fuel into thrust*

<!-- Evidence Tier: Textbook -->

## The Thrust Problem

L1 covered aerodynamics — lift, drag, stability. An aircraft in steady level flight needs lift equal to weight and thrust equal to drag. Lift is "free" in the sense that it costs only the drag penalty induced by generating it; thrust is not free — it must come from somewhere, and that somewhere is an engine converting chemical energy in fuel to kinetic energy of exhaust, accelerating a mass of gas rearward to push the aircraft forward.

This is Newton's third law applied to aerospace. All air-breathing and rocket propulsion share one core equation:

$$F = \dot{m} \Delta V$$

Thrust equals the mass flow rate of propellant times the velocity change imparted to it. You can get thrust two ways: push a small mass to very high velocity (rocket, turbojet) or push a large mass to modest velocity (turboprop, turbofan, helicopter rotor). The tradeoff between these strategies — bounded by specific fuel consumption, engine weight, flight speed, and operating envelope — has shaped every air propulsion system built.

## Piston Engines

The first powered aircraft (Wright Flyer, 1903) used a piston engine driving a propeller. Piston engines dominated aviation until the jet age, and remain standard for light general aviation aircraft below ~300 hp.

A piston aviation engine is similar in principle to an automotive engine but optimized differently:
- **Air cooled**: simpler, lighter than liquid cooling (though some aircraft use liquid cooling).
- **Horizontally opposed cylinders**: compact, low-profile, balanced. Typical light-aircraft layout (Lycoming, Continental).
- **Magneto ignition**: each cylinder has two spark plugs driven by independent magnetos — redundancy for safety.
- **Fixed-pitch or constant-speed propeller**: propeller converts shaft power to thrust.

Power drops with altitude as air density decreases. **Turbocharging** or **supercharging** restores sea-level power at altitude and is standard on higher-performance piston engines. Above ~25,000 ft, piston engines become impractical and turboprop or turbofan becomes necessary.

Fuel is typically **avgas** (100LL) — high-octane leaded gasoline. Leaded fuel persists for historical and certification reasons; unleaded alternatives are in development. Small diesel aircraft engines running Jet A exist but have limited market penetration.

## Propellers

The propeller translates rotational shaft power into thrust by accelerating air rearward. It's essentially a rotating wing — each blade generates lift (thrust) and drag (absorbing shaft torque) from its airfoil section.

Propeller design tradeoffs:
- **Diameter**: larger accelerates more air less, more efficient at low speeds; limited by tip speed (must stay subsonic to avoid shock losses).
- **Pitch**: angle of the blade to plane of rotation. Coarse pitch good for cruise; fine pitch good for takeoff and climb.
- **Number of blades**: more blades spread load but increase interference drag; 2-4 is typical.
- **Constant-speed prop**: governor adjusts blade pitch to maintain constant RPM as conditions change, keeping the engine at optimum speed.

Propeller efficiency peaks around 85% at cruise for a well-designed installation. Tip speed is the limit — above ~Mach 0.85 at the tip, shock waves form and efficiency falls sharply. This caps propeller aircraft at ~400-500 knots in most designs; above that, the jet is necessary.

## Jet Engine Cycle

The **gas turbine engine** — the jet engine and its relatives — dominates modern aviation above light-aircraft sizes. All variants operate on the **Brayton cycle**:

1. **Intake**: air enters.
2. **Compression**: multi-stage compressor raises pressure 20-50×.
3. **Combustion**: fuel injected and burned at nearly constant pressure; temperature rises to 1500-2000 K.
4. **Expansion**: hot high-pressure gas expands through turbine and nozzle, extracting work and accelerating exhaust.
5. **Exhaust**: high-velocity gas leaves rearward.

The turbine extracts just enough power to drive the compressor (and any fan or propeller). The remaining enthalpy appears as kinetic energy of the exhaust, producing thrust.

**Thermodynamic efficiency** increases with pressure ratio and combustion temperature. Modern engines push both — composite and superalloy turbine blades with internal cooling and thermal barrier coatings enable turbine inlet temperatures above the melting point of the metal. Single-crystal blades eliminate grain boundaries that would be the failure sites.

## Engine Types

**Turbojet**: simplest jet engine. Compressor, combustor, turbine, exhaust nozzle. All thrust from the hot jet. Efficient at supersonic speeds but inefficient and noisy at subsonic speeds. Used in early jet airliners (707) and military fighters. Now largely replaced except in some military applications.

**Turbofan**: adds a large-diameter fan at the front, driven by a low-pressure turbine, that bypasses most of the air around the core engine. Bypass air is accelerated modestly — producing thrust more efficiently at subsonic speeds than a pure jet. Modern airliner engines have **bypass ratios** of 5-12, with ultra-high-bypass (geared turbofans) reaching 12-15. Essentially all commercial airliners and most military transports use turbofans.

**Turboprop**: adds a larger propeller driven by extra turbine stages. The propeller does most of the work; exhaust produces only a small fraction of thrust. Very efficient at 200-400 knots and at lower altitudes. Used in regional airliners, utility aircraft, military transports, maritime patrol. Above ~500 knots, propeller efficiency collapses and turbofan wins.

**Turboshaft**: similar to turboprop but all turbine power goes to a shaft, not a propeller. Drives helicopter rotors, marine propellers, power generation. Decoupled from engine operating speed via reduction gearbox.

**Ramjet**: no compressor, no turbine. Intake shape compresses supersonic incoming air by slowing it; fuel burns; hot gas exits. Only works above ~Mach 2; can reach Mach 5+. Used in some missiles and the SR-71 (with the J58 operating as a ramjet at high Mach).

**Scramjet**: ramjet where combustion occurs at supersonic internal flow. Enables Mach 5-15+ flight. Active research; limited operational deployment (X-51A, DF-ZF). Challenging combustion stability, materials, testing.

**Pulsejet**: intermittent combustion, simple valves. V-1 "buzz bomb" used them. Low efficiency, high noise.

## Thrust and Efficiency

Thrust is measured as force — kN or lbf. Specific fuel consumption (SFC) is fuel flow per unit thrust: kg/(N·s) or lbm/(lbf·hr). Lower is better.

**Propulsive efficiency** compares power delivered to the aircraft to power in the exhaust — maximized when exhaust velocity approaches flight velocity. A turbofan cruising at Mach 0.8 has high propulsive efficiency because bypass velocity is close to flight velocity. A turbojet has exhaust at Mach 2+ while flying at Mach 0.8 — most of the kinetic energy is wasted.

**Thermal efficiency** is fraction of fuel energy converted to exhaust kinetic energy; governed by the Brayton cycle and materials limits.

**Overall efficiency** = thermal × propulsive. Modern turbofans reach 35-40% overall — much better than 1960s turbojets at ~15%. The Pratt & Whitney geared turbofan, GE9X, and Rolls-Royce Trent series represent the current state of the art.

## Compressors

Compressors raise inlet air pressure without burning fuel. Two types:

- **Axial compressor**: multi-stage, each stage consisting of rotating rotor blades and stationary stator blades. Each stage contributes modest pressure ratio (1.1-1.4); stacks produce total ratios up to 50:1 in modern engines. Narrow, long, and efficient for high flow rates.
- **Centrifugal compressor**: accelerates air outward; diffuser decelerates it, converting kinetic energy to pressure. Higher pressure per stage but less efficient at high flows. Used in small engines (APUs, helicopters, early jets).

Many small engines combine axial (front stages) with centrifugal (final stage) for compactness.

**Surge** is a compressor instability — flow reverses momentarily. Destructive. Prevented by bleed valves, variable stator vanes, and careful operating limits.

## Combustion

The **combustor** burns fuel in a carefully controlled way:
- **Primary zone**: stoichiometric combustion at 2000+ K.
- **Intermediate zone**: additional air mixes in to complete combustion.
- **Dilution zone**: bypass air cools the flow to ~1800 K, the limit turbine materials can survive.

**Emissions** concerns:
- **NOx**: formed at high temperatures. Reduced by lean combustion, staged combustion, rich-quench-lean designs.
- **CO, unburned hydrocarbons**: incomplete combustion. Minimized with efficient combustors.
- **Particulates**: soot from fuel-rich zones. Concerns near airports.
- **CO₂**: proportional to fuel burn. Reduced only by reducing fuel use (efficiency, sustainable aviation fuels).

Sustainable aviation fuels (SAF) — biofuels and synthetic kerosene — are chemically similar to Jet A and drop-in compatible. Current SAF production is small; scale and cost remain the challenges.

## Turbines

The **turbine** extracts work from hot gas to drive the compressor and fan. It operates at the highest temperatures in the engine — above 1800 K at the inlet. Key technologies:

- **Single-crystal superalloy blades**: cast with no grain boundaries; resist creep at high temperature.
- **Internal cooling**: air bled from the compressor flows through cooling passages inside each blade, exits through film-cooling holes that create an insulating layer on the surface.
- **Thermal barrier coatings**: ceramic layers on blade surface reduce heat transfer into the metal.
- **Ceramic matrix composites**: higher temperature capability than metals; entering service in some applications.

A single turbine blade costs tens of thousands of dollars. A modern engine has hundreds of them. Blade failure is catastrophic — a liberated blade can cut through adjacent structure. Manufacturing and inspection are exquisitely controlled.

## Integration with Airframe

Engine installation is not just bolting the engine on:
- **Nacelle** shape manages air flow to and around the engine.
- **Inlet** slows and compresses supersonic flow (military) or simply delivers uniform subsonic flow (airliners).
- **Mount** carries thrust loads and isolates vibration. Pylons on wing-mounted engines are critical structural elements.
- **Thrust reversers** redirect exhaust forward on landing to reduce runway length.
- **Fire protection**: detection, suppression, drain systems, containment.

**Engine-out** is a certified failure mode. Twin-engine airliners must safely continue takeoff after losing an engine at the most critical moment (V1). This drives minimum thrust requirements, asymmetric thrust handling, and ETOPS (Extended-range Twin-engine Operational Performance Standards) — the certification that allows twins to fly long over-water routes.

## Operation and Maintenance

A modern turbofan has ~25,000 flight hours between major overhauls, with on-condition maintenance between. Health monitoring — vibration, oil debris, performance deterioration — is continuous in flight. Engine manufacturers (GE, Pratt & Whitney, Rolls-Royce, Safran, CFM) increasingly sell "power by the hour" — airline pays per flight hour; manufacturer handles maintenance.

Engine **certification** — demonstrating compliance with FAR Part 33 in the US or equivalent elsewhere — includes bird strike, blade-out containment, icing, endurance, emissions testing. New engine development runs $2-5 billion and 7-10 years.

## Why This Level Matters

Jet and turbofan engines are among the most demanding engineered systems ever built — operating at the edge of materials limits, with reliability rates that make them safer than almost any other transportation mode per mile. They enabled affordable long-distance air travel in the second half of the 20th century and continue as the technical backbone of commercial aviation and much of military aviation.

Engine design is where thermodynamics, fluid dynamics, combustion, materials science, and mechanical engineering converge at their most intense. The efficiency improvements that have cut fuel burn per seat-mile by ~80% since the 1960s are mostly in the engine, not the airframe. Further progress — open rotor engines, hybrid-electric propulsion, hydrogen combustion, sustainable fuels — will shape the next generation of aviation's environmental footprint.

## The Transition to Level 3

L3 turns to **spaceflight** — rockets, orbits, satellites, and the particular challenges of leaving the atmosphere. Propulsion there operates on the same Newton's-third-law principle but with no atmospheric oxidizer, demanding carried propellants and a different economic calculus.

Next: [L3 — Spaceflight](./L3_Spaceflight.md) *(deferred)*
