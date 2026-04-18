# Level 1: Circuits
*Ohm's law, Kirchhoff's laws, and how the world's wiring actually works*

<!-- Evidence Tier: Textbook -->

## Three Laws Run Everything

Every circuit — in your phone, your car, your home, your factory, the power grid — ultimately reduces to three principles applied repeatedly:

- **Ohm's law**: voltage across a resistor = current through it × resistance. $V = IR$.
- **Kirchhoff's Current Law (KCL)**: the sum of currents entering any node equals the sum leaving. (Conservation of charge.)
- **Kirchhoff's Voltage Law (KVL)**: the sum of voltages around any closed loop equals zero. (Energy conservation.)

Everything else is either a more detailed model of specific components (capacitors, inductors, diodes, transistors) or a calculation technique (mesh analysis, nodal analysis, Thevenin equivalents).

## The Units

Understanding what the symbols mean:

- **Voltage** (V, volts): difference in electric potential between two points. Analog of water pressure. A battery's "12 volts" means it pushes charge with that much potential energy per unit charge.
- **Current** (I, amperes): flow rate of electric charge. Analog of water flow rate in liters per second. 1 ampere = 1 coulomb of charge per second.
- **Resistance** (R, ohms): opposition to current flow for a given voltage. Analog of pipe restriction. 1 ohm = 1 volt per ampere.
- **Power** (P, watts): rate of energy delivery. $P = VI = I^2R = V^2/R$.
- **Energy** (E, joules or kWh): power × time. Your electric bill is in kWh.

A single phone charger provides ~10 watts. A household LED bulb draws ~10 watts. A hair dryer, ~1500 watts. An electric vehicle on a fast charger, 50,000–350,000 watts. A utility power plant, millions of watts.

## Ohm's Law, Properly Understood

$V = IR$ sounds simple. In practice it governs the behavior of every resistive element. A few implications:

- If you increase voltage across a fixed resistor, current increases proportionally.
- If you add resistance at constant voltage, current decreases.
- A wire is a low-resistance path (copper: ~17 nΩ·m resistivity). If you connect a power source across a low-resistance path with no load, you get a **short circuit** — high current, heat, possibly fire.
- A load (a lamp, a motor, a phone) has a design resistance that limits current at the intended supply voltage.

For non-ohmic elements (diodes, transistors, many active devices), Ohm's law doesn't apply directly — those have voltage-current relationships that aren't linear. But for resistors, heaters, and the resistive component of most real devices, Ohm's law is the rule.

## Series vs. Parallel

Two resistors in series have combined resistance $R_1 + R_2$. The same current flows through both; voltage divides between them in proportion to their resistance (**voltage divider**).

Two resistors in parallel have combined resistance $R_1 R_2 / (R_1 + R_2)$. The same voltage appears across both; current divides in inverse proportion to resistance (**current divider**).

Household wiring is parallel. Every outlet in a room is connected across the same supply voltage (120V in North America, 230V in most of the rest of the world). Plugging in more devices increases total current — up to the limit of the circuit breaker. If you exceed it, the breaker trips. If you had no breaker, the wiring would overheat.

Christmas lights from decades ago were series (cheap) — one bulb fails and the whole string goes dark. Modern strings are mostly parallel (with backup paths) to avoid this problem.

## Kirchhoff's Laws in Practice

KCL and KVL let you analyze circuits with multiple sources and branches.

**Nodal analysis**: assign variable voltages to each node, apply KCL to write current balance at each node, solve.

**Mesh analysis**: assign variable currents to each loop, apply KVL around each loop, solve.

Either approach works; which is easier depends on the circuit topology. Modern circuit simulators (SPICE and derivatives) do this numerically on circuits with thousands of nodes and components.

## Capacitors and Inductors

Three fundamental passive components:

- **Resistors** dissipate energy as heat. $V = IR$. Instantaneous relationship.
- **Capacitors** store energy in an electric field between parallel plates. $I = C dV/dt$. Current flows only when voltage is changing.
- **Inductors** store energy in a magnetic field around a coil. $V = L dI/dt$. Voltage develops only when current is changing.

In DC (direct current, constant voltage), capacitors act like open circuits (no current once charged) and inductors like short circuits (wire resistance only). In AC (alternating current), both capacitors and inductors have frequency-dependent **impedance** — generalized resistance that includes phase relationships.

- Capacitor impedance: $Z_C = 1/(j\omega C)$ — inversely proportional to frequency.
- Inductor impedance: $Z_L = j\omega L$ — proportional to frequency.

This frequency dependence is the basis of filters. A high-pass filter uses a capacitor to block low frequencies; a low-pass filter uses an inductor or capacitor to block high frequencies. Audio equalizers, radio tuners, and signal processing chains all use combinations of these.

## AC vs. DC

Direct current flows one direction. Batteries produce DC. Most electronic devices' internal workings are DC.

Alternating current reverses direction periodically. The grid delivers AC — 60 Hz in the Americas and parts of Asia, 50 Hz in most of Europe, Africa, and the rest of Asia. AC is delivered at high voltage (hundreds of kV on long-distance transmission lines) and stepped down to 120 or 230 V at distribution.

AC dominates for historical and physical reasons:

- **Transformers**: change voltage easily in AC, which lets grid operators transmit at high voltage (low current, low loss) and distribute at low voltage (safer).
- **Electric motors**: induction motors (simple, cheap, robust) run natively on AC.
- **Generators**: most electricity is generated by rotating machinery — which produces AC natively.

DC is increasingly common in long-distance transmission (HVDC, high-voltage DC) because HVDC has lower losses over very long distances and can link grids with different frequencies or phases.

Your wall outlet is AC. Your laptop's internal power is DC. Between them is a power supply — a circuit that rectifies AC to DC, smooths ripple, and regulates to a stable voltage.

## Diodes and Transistors

Active components break the linearity of passive circuits.

**Diodes** conduct in one direction and block the other. Useful for rectification (AC-to-DC conversion), signal detection, voltage clamping, and light emission (LEDs — light-emitting diodes, which are ubiquitous in modern electronics and lighting).

**Transistors** are voltage- or current-controlled switches. A small signal on one terminal controls a larger current between two others. Bipolar junction transistors (BJTs) amplify based on current; field-effect transistors (FETs, especially MOSFETs) amplify based on voltage.

A MOSFET can be either a logic gate (on/off) or an amplifier (continuous gain). Billions of MOSFETs on a chip, all switching between on and off, make digital logic possible. A modern CPU has on the order of 10–100 billion transistors on a die the size of a fingernail.

## Power Dissipation and Thermal Management

Resistors dissipate power: $P = I^2 R$. This energy becomes heat. A 100-ohm resistor carrying 1 A dissipates 100 W — enough to burn you and to destroy most small components.

Heat dissipation sets practical limits in electronics. A CPU may dissipate 100 W in a fingernail-sized area. Active cooling (heatsinks, fans, liquid cooling) keeps temperatures below the die's limit (typically 100°C). If the cooling fails, the CPU throttles its clock to reduce heat; if the cooling fails badly enough, the CPU is destroyed.

Thermal management is often the binding constraint in high-performance electronics. Datacenter layouts, GPU design, smartphone chassis — all are shaped by the need to move heat away from hot components.

## Grounding and Safety

**Ground** is a reference voltage, typically 0 V, to which all other voltages are measured. In a building's electrical system, ground is physically connected to the Earth via a grounding rod.

Safety grounding prevents electrical faults from becoming shock hazards. A metal appliance has its case connected to ground. If the live wire inside shorts to the case, current flows through the ground wire rather than through a person touching the appliance. A **ground-fault circuit interrupter (GFCI)** detects differences between hot and neutral current (suggesting current is flowing through an unintended path — possibly a person) and cuts power in milliseconds. GFCIs are required in kitchens, bathrooms, and outdoor circuits in modern codes.

Electrical safety standards have been refined over a century after early electrocution hazards. Modern code-compliant wiring is remarkably safe. Ignoring the code (overloaded circuits, amateur installations, water near outlets) is where things go wrong.

## Batteries

Batteries store energy chemically and release it electrically. Key parameters:

- **Voltage**: determined by the chemistry. Lithium-ion: ~3.7 V per cell. Lead-acid: ~2.0 V per cell. Alkaline: ~1.5 V per cell.
- **Capacity** (mAh or Ah): how much charge the battery can deliver. 2000 mAh means the battery can deliver 2 amps for 1 hour or 1 amp for 2 hours (approximately).
- **Energy density**: capacity × voltage / mass. Lithium-ion: ~250 Wh/kg. Gasoline (for comparison): ~12,000 Wh/kg, but internal combustion is only 25–35% efficient.
- **Cycle life**: how many charge-discharge cycles before significant degradation. Modern lithium-ion: 500–2000 cycles depending on depth of discharge and temperature.

Battery technology has improved roughly 6% per year in energy density and about 85% cumulatively in cost from 2010 to 2024. This is the enabling technology of portable electronics, electric vehicles, and increasingly grid-scale storage.

## Why This Level Matters

Electrical engineering begins with circuits. Every electronic device, every power system, every industrial process that uses electricity (almost all of them) starts from the laws and components covered here.

If you can read a simple circuit schematic — identify sources, loads, paths, series vs parallel sections, component types — you can understand what most consumer electronics are doing at a functional level. You can also diagnose simple faults (a blown fuse, a burned-out resistor, a loose connection) without needing specialized equipment.

## The Transition to Level 2

L2 moves from DC and simple circuits to **AC fundamentals** — phasors, impedance, power factor, three-phase systems, and the mathematical machinery that lets you analyze and design the AC systems that dominate practical electrical engineering.

Next: [L2 — AC Fundamentals](./L2_AC_Fundamentals.md) *(Phase 2D)*
