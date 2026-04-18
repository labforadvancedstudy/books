# Level 3: Power Electronics and Motors
*Semiconductor switches, power conversion, and the motors that do civilization's mechanical work*

<!-- Evidence Tier: Textbook -->

## From Fixed AC to Flexible Electricity

L2 covered AC fundamentals — the fixed-frequency, fixed-voltage world of power systems. L3 turns to how that rigid AC is converted to and from other forms — DC at various voltages, AC at variable frequencies, pulses, and precisely controlled waveforms — using semiconductor switches. Power electronics, invented as viable technology only from the 1950s and becoming dominant in the 2000s, is now essential infrastructure for:

- Variable-speed motor drives (the biggest single electricity user worldwide).
- Photovoltaic and wind power conversion into grid-compatible AC.
- Battery charging and discharging (electric vehicles, stationary storage).
- HVDC transmission lines (long-distance power transmission).
- Virtually every consumer electronic device (phones, laptops, appliances).

Together with electric motors, power electronics convert electrical energy into useful work more efficiently and flexibly than any prior technology. About 45% of global electricity powers electric motors; another 15% passes through power electronics for lighting, computing, and control. The field touches almost everything that uses electricity.

## Power Semiconductor Devices

Power electronics is built on a small set of switching devices:

**Diode**: passive switch; conducts in one direction when forward-biased. Rectifier diodes handle high current; Schottky diodes for low voltage/fast switching.

**Thyristor (SCR)**: turns on by gate pulse, turns off when current reaches zero. High voltage (up to ~10 kV) and current (~5 kA+). Used in HVDC, large motor drives, industrial rectifiers. Cannot be turned off by gate.

**MOSFET**: voltage-controlled; fast switching; low on-resistance at low voltages. Dominant for low-voltage (<200 V) high-frequency applications — computer power supplies, phone chargers, automotive.

**IGBT (Insulated Gate Bipolar Transistor)**: voltage-controlled; handles 600 V to 6.5 kV, tens to thousands of amperes. Workhorse of motor drives, utility-scale inverters, induction heating.

**GTO (Gate Turn-Off thyristor)**: turn-off by gate current; high power; older technology being displaced by IGBT.

**IGCT (Integrated Gate-Commutated Thyristor)**: very high power; HVDC, large industrial drives.

**Wide-bandgap devices** (SiC, GaN): higher voltage, higher temperature, faster switching. Silicon Carbide MOSFETs now common in EV drives; GaN in chargers. Rapidly displacing silicon in new high-performance applications.

Device selection balances voltage rating, current rating, switching speed, on-state losses, switching losses, thermal management, and cost.

## Switching Principles

Power electronics manipulate energy flow by switching devices on and off fast. Key concepts:

**Duty cycle**: fraction of switching period the device is on. Controls average output.

**Pulse width modulation (PWM)**: high-frequency (kHz to hundreds of kHz) switching to synthesize desired waveform. Fundamental technique across almost all converters.

**Space vector PWM**: sophisticated 3-phase PWM minimizing ripple.

**Hard vs. soft switching**: turning on/off with voltage and current simultaneously high (hard) causes losses; resonant / zero-voltage / zero-current switching reduces losses.

**Filters**: inductors, capacitors smooth switched waveforms. Input and output.

**Isolation**: transformers decouple high-voltage from low-voltage sides; safety and flexibility.

## Converter Types

Power electronics are often categorized by what they convert:

**AC-DC rectifier**:
- **Uncontrolled**: diode bridge. Converts AC to pulsing DC.
- **Controlled**: thyristor bridge. Variable DC output by phase-angle control.
- **PWM rectifier**: IGBT-based; controlled power factor, lower harmonics.

**DC-DC converter**:
- **Buck**: steps voltage down (e.g., 12 V to 3.3 V).
- **Boost**: steps voltage up.
- **Buck-boost**: either direction.
- **Forward, flyback, push-pull, half-bridge, full-bridge**: isolated topologies with transformers.
- **Resonant converters**: LLC, LCC for high efficiency at high frequency.

**DC-AC inverter**:
- **Voltage source**: fixed DC bus, variable AC output. Dominant topology.
- **Current source**: less common; specific applications.
- **Single-phase, three-phase**: depending on application.
- **Modular multilevel (MMC)**: high voltage, low distortion; HVDC, FACTS.

**AC-AC cycloconverter**: direct AC-AC without DC intermediate. Large motor drives; being displaced by back-to-back converters.

## Motor Drives

Modern motor drives combine rectifier + DC bus + inverter to convert grid AC to variable-frequency variable-voltage AC for the motor. This enables:

- **Variable-speed operation**: motor runs at the speed the process needs, not the grid's fixed frequency.
- **Soft starting**: avoids large inrush currents at startup.
- **Regenerative braking**: decelerating motor pushes energy back to DC bus; can return to grid or absorb in resistor.
- **Dynamic torque control**: precise, fast control of motor output.
- **Efficiency**: matching speed to load saves enormous energy.

**Variable Frequency Drive (VFD)** is the standard term for a motor drive unit. Ubiquitous in industry, HVAC, pumps, fans, conveyors. 20-50%+ energy savings typical when VFD replaces fixed-speed operation with throttle control.

## Electric Motors

Motor types:

**DC motors** (brushed):
- Simple control, high starting torque.
- Brushes wear and spark.
- Used in small applications; declining in most industrial.

**DC motors** (brushless, BLDC):
- Permanent-magnet rotor, electronic commutation.
- No brushes; long life; high efficiency.
- Common in drones, EVs (some), pumps, fans.

**Induction motors** (asynchronous):
- Three-phase stator, squirrel-cage rotor.
- Rotor lags stator field (slip).
- Robust, cheap, no permanent magnets.
- Vast majority of industrial motors historically.
- Good efficiency (>90% for larger sizes); best with VFD.

**Synchronous motors**:
- Rotor locked to stator field frequency.
- Higher efficiency than induction.
- Requires DC rotor excitation (wound-rotor) or permanent magnets.
- Large industrial, where precise speed critical.

**Permanent magnet synchronous motors (PMSM)**:
- Neodymium magnets on rotor.
- Highest efficiency, highest power density.
- Dominant in EVs (Tesla, Hyundai, most modern designs), hybrid vehicles, some industrial drives.
- Dependence on rare earth magnets a supply concern.

**Switched reluctance motors**:
- Salient-pole rotor, no magnets or windings.
- Simple, robust, can be efficient.
- Emerging in EVs (Tesla's Model 3 rear motor derivative, others) to reduce rare earth dependence.
- Acoustic noise traditionally a challenge.

**Linear motors**:
- Unrolled rotary motor.
- Direct linear motion without gears.
- CNC machines, maglev trains, semiconductor wafer handling.

## Motor Sizing and Selection

Motor selection considerations:
- **Torque and speed requirements**: rated and peak.
- **Duty cycle**: continuous, intermittent, variable.
- **Ambient temperature and cooling**: air, water, TEFC, ODP.
- **Efficiency class**: IE1 through IE5 (EU); increasingly mandated for new motors.
- **Starting current**: direct-on-line vs. soft-start vs. VFD.
- **Environment**: dust, moisture, chemicals, explosion hazard.
- **Mounting and shaft**: standard frame sizes (NEMA in US, IEC elsewhere).

Large industrial motors are commodity items, specified by standard frame and rating; custom motors for special applications.

## Control of Motor Drives

Control sophistication varies:

**Scalar (V/f) control**: simplest; maintain constant voltage-to-frequency ratio for constant flux. Adequate for pumps, fans.

**Vector (field-oriented) control**: transform into rotating reference frame; independently control flux and torque. Precise dynamic response; standard for servo drives, EVs, advanced industrial.

**Direct torque control (DTC)**: ABB's approach; switches inverter states directly to control torque and flux. Fast response.

**Sensorless control**: estimate rotor position/speed from electrical measurements; avoid encoder. Mature for most applications.

**Model predictive control**: emerging; computationally intensive but powerful.

## Grid-Tie Inverters

Solar panels and batteries produce DC; to feed the grid, inverters convert to AC synchronized with grid voltage and frequency. Requirements:
- **Grid synchronization**: phase-locked loop (PLL) to match grid.
- **MPPT (Maximum Power Point Tracking)** for solar: extract maximum power despite changing irradiance and temperature.
- **Anti-islanding**: detect grid loss, shut down for utility worker safety.
- **Reactive power support**: newer standards require inverters to provide reactive power to grid.
- **Low voltage ride-through (LVRT)**: ride through transient grid faults without disconnection.
- **Harmonic distortion limits**: IEEE 1547, grid codes.

Residential: string inverter or microinverter per panel.
Utility-scale: central inverter (1-5 MW) or distributed string inverters.

## HVDC

**High-Voltage Direct Current (HVDC) transmission**: long-distance lines use DC for efficiency and controllability.

Advantages over AC:
- No reactive power along line; two conductors instead of three.
- No skin effect limiting conductor utilization.
- Can connect asynchronous AC grids (different frequencies or out of phase).
- Controllable power flow.

Applications: long overhead (~600 km+), submarine cable (~50 km+ AC losses prohibitive), grid interconnections.

Technologies:
- **Line-commutated converters (LCC)**: thyristor-based. Highest power, mature. Require strong AC grids at both ends.
- **Voltage source converters (VSC)**: IGBT-based. Can start into dead grid (black start); bidirectional; more compact. Dominant for new projects.

Large HVDC systems: Belo Monte to São Paulo in Brazil, China's ultra-high voltage (±800 kV+, 1100 kV) network spanning thousands of km.

## Battery Systems

Lithium-ion batteries need sophisticated battery management systems (BMS):
- **Cell balancing**: individual cells voltage-equalized for capacity use and safety.
- **State of charge (SoC) estimation**: not directly measurable; estimated from voltage/current/temperature.
- **State of health (SoH)**: capacity degradation over cycles and time.
- **Thermal management**: cooling/heating to maintain optimal temperature range; critical for safety and life.
- **Safety monitoring**: detect abnormal voltage, current, temperature; disconnect/isolate as needed.

Battery pack integration: hundreds-to-thousands of cells in series (for voltage) and parallel (for capacity) configurations; each monitored and managed.

## EV Charging

Charging infrastructure:
- **Level 1** (120 V AC, US residential): ~1.4 kW; slow.
- **Level 2** (240 V AC): ~7-22 kW; home fast charging, workplace, most public stations.
- **DC fast charging** (CCS, CHAdeMO, NACS/Tesla): 50-350+ kW; highway stations. Minutes to meaningful range.
- **Ultra-fast** (350-500+ kW): emerging for heavy trucks.

Onboard chargers (AC): convert grid AC to battery DC; power limited by onboard electronics cost and thermal.

Off-board DC fast chargers: large power electronics outside vehicle; vehicle just connects to DC.

**V2G (vehicle-to-grid)**: cars feed power back to grid during peak or emergencies. Standards emerging; Nissan Leaf leading.

## Efficiency and Losses

Power electronics are ~95-99% efficient:
- **Conduction losses**: forward voltage drop × current.
- **Switching losses**: turn-on and turn-off times × voltage × current.
- **Gate drive losses**: small.
- **Magnetic losses** in inductors, transformers.
- **Filter losses**.

Higher frequency reduces filter size but increases switching loss. SiC and GaN enable higher frequency at lower loss.

Motor efficiency: 85-95% typical, higher for larger motors. Losses: stator copper (I²R), rotor copper, iron (hysteresis + eddy current), mechanical (bearings, windage).

System efficiency (source-to-useful-work) in motor-driven systems is often dominated by the load coupling (e.g., throttling in pump systems). VFDs that match speed to load demand can double system efficiency.

## Why This Level Matters

Power electronics and motors are the substrate of electrified civilization. Every EV, every solar farm, every wind turbine, every HVDC link, every industrial variable-speed drive depends on the semiconductor switches and motor technologies described here. Efficiency gains in these technologies compound across billions of installations.

The pace of change is fast. Wide-bandgap semiconductors (SiC, GaN) are displacing silicon in high-performance applications; permanent magnet motors with rare-earth reductions or eliminations are active research; grid-forming inverters are needed to replace the inertia lost as synchronous generators retire; V2G and distributed resources require new control paradigms.

Engineers in these fields work at the intersection of electromagnetics, semiconductor physics, thermal management, control theory, and power systems. The work is increasingly essential for the energy transition — arguably the most consequential engineering transition of the century.

## The Transition to Level 4

L4 turns to **electronics and signal processing** — the other side of electrical engineering: integrated circuits, semiconductor physics, digital logic, signal processing, communications, and the systems that have built the information technology revolution.

Next: [L4 — Electronics & Signal Processing](./L4_Electronics_and_Signal_Processing.md) *(deferred)*
