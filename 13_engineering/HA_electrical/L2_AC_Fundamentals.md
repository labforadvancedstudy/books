# Level 2: AC Fundamentals
*Alternating current — phasors, impedance, power, and three-phase systems*

<!-- Evidence Tier: Textbook -->

## Why AC Won

L1 covered DC circuits — constant voltages, resistors, capacitors charging on transients. Real power systems don't work that way. The electricity in the wall outlet reverses direction 50 or 60 times per second. Nearly all electric power is generated, transmitted, and distributed as alternating current (AC). The reason is historical, physical, and decisive: AC voltages can be stepped up and down easily by transformers, enabling long-distance transmission at high voltage and low current (minimizing resistive losses), then reduced to safe voltages for use. DC couldn't do this efficiently when the standard was being chosen in the 1880s. That settled the War of Currents in favor of AC, and AC has dominated ever since — though modern power electronics have partially reopened the question with HVDC transmission for specific applications.

L2 builds the tools for analyzing AC: sinusoidal signals, phasors, complex impedance, power factor, and three-phase systems. These are the mathematical machinery underlying the entire electric power system and much of modern electronics.

## Sinusoidal Signals

AC voltages and currents vary sinusoidally with time:

$$v(t) = V_m \cos(\omega t + \phi)$$

Where $V_m$ is the amplitude (peak), $\omega = 2\pi f$ is the angular frequency in rad/s, $f$ is the frequency in Hz (60 Hz in North America, 50 Hz in most of the rest of the world), and $\phi$ is the phase angle.

Sinusoids arise naturally because rotating machinery (generators) produces them — a coil spinning in a magnetic field generates a voltage proportional to the sine of the angle. They're also mathematically special: derivatives and integrals of sinusoids are sinusoids at the same frequency, just shifted in phase and scaled in amplitude. This closure under differentiation makes linear circuits driven by sinusoids solvable in a particularly elegant way.

**RMS (root mean square) values**: for power calculations, the effective value of a sinusoid is $V_{RMS} = V_m / \sqrt{2}$. The "120 V" outlet in North America is 120 V RMS, corresponding to ~170 V peak. RMS matters because it's the DC equivalent that would deliver the same average power to a resistor.

## Phasors

Analyzing circuits with sinusoids using direct time-domain differential equations is cumbersome. **Phasors** convert the problem to complex algebra.

A sinusoid $v(t) = V_m \cos(\omega t + \phi)$ is represented as the complex number $\mathbf{V} = V_m e^{j\phi}$ (often written $V_m \angle \phi$). This captures amplitude and phase; frequency is implicit and assumed common to all signals in the network.

Phasor relationships for ideal components:
- **Resistor**: $\mathbf{V} = R\mathbf{I}$. Voltage and current are in phase.
- **Inductor**: $\mathbf{V} = j\omega L \cdot \mathbf{I}$. Voltage leads current by 90°.
- **Capacitor**: $\mathbf{V} = \mathbf{I} / (j\omega C)$. Voltage lags current by 90°.

Define **impedance** $Z$ as the complex ratio of voltage phasor to current phasor:
- $Z_R = R$
- $Z_L = j\omega L$
- $Z_C = 1/(j\omega C) = -j/(\omega C)$

Impedance combines series and parallel the same way as DC resistance — just using complex arithmetic. Kirchhoff's laws hold in phasor form. Thévenin and Norton equivalents extend naturally. The entire linear circuit analysis toolkit transfers.

## Power in AC Circuits

AC power is more subtle than DC. Instantaneous power is $p(t) = v(t) i(t)$; it varies through the cycle and can be positive (flowing to the load) or negative (flowing back to the source).

For a sinusoidal voltage and current with phase difference $\theta$:

- **Real (active) power**: $P = V_{RMS} I_{RMS} \cos\theta$ — average power dissipated in the load. Units: watts (W).
- **Reactive power**: $Q = V_{RMS} I_{RMS} \sin\theta$ — power that oscillates back and forth between source and load without being consumed. Units: volt-amperes reactive (VAR).
- **Apparent power**: $S = V_{RMS} I_{RMS}$ — the magnitude of the complex power. Units: volt-amperes (VA).
- **Complex power**: $\mathbf{S} = P + jQ$.

**Power factor** is $\cos\theta = P/S$. A resistive load has PF = 1 (all apparent power is real). An ideal inductor or capacitor has PF = 0 (all apparent power is reactive). Most loads are between — motors are typically lagging (inductive, current lags voltage); capacitive loads are leading.

Low power factor is a problem for utilities. A load drawing 1000 W at PF 0.6 requires 1667 VA of apparent power — the wires and transformers must be sized for the apparent power, not just the real power. Utilities charge commercial customers for low power factor and operators install **power factor correction capacitors** to cancel inductive reactive power, bringing PF closer to 1.

## Resonance

In a circuit with both inductance and capacitance, impedances have opposite signs on the imaginary axis. At a specific frequency, they cancel:

$$\omega_0 = \frac{1}{\sqrt{LC}}$$

**Series RLC** at resonance: impedance is purely resistive and minimum; current is maximum. A sharp resonance has high **Q factor** (quality factor) and narrow bandwidth.

**Parallel RLC** at resonance: impedance is maximum; current is minimum.

Resonance is exploited in filters, oscillators, and radio tuning — matching the receiver's resonant frequency to a broadcast frequency selects it from the ambient signal soup. Resonance is also a threat in power systems — harmonics from nonlinear loads can excite unintended resonances and cause overvoltages.

## Three-Phase Systems

Power systems don't use single-phase AC for most generation and transmission. They use **three-phase** — three voltages of equal magnitude at frequencies identical but phase-shifted 120° apart.

Why three phases? Several reasons:
- **Constant instantaneous power**: three sinusoidal powers summed are constant, unlike single-phase which pulses at twice the line frequency. Steadier torque on motors, less vibration.
- **Efficient transmission**: three-phase transmission uses three conductors (plus ground) and delivers three times the power of single-phase on just ~1.7× the conductor material.
- **Motor advantages**: three-phase windings create a rotating magnetic field naturally, enabling the robust three-phase induction motor.

**Wye (star) connection**: three phase windings connect to a common neutral. Line-to-line voltage is $\sqrt{3}$ times line-to-neutral. North American distribution uses 120/208 V wye or 277/480 V wye.

**Delta connection**: three phase windings form a closed triangle. No neutral. Used in many industrial motors and transformers.

**Balanced analysis**: with balanced sources and loads, three-phase analysis reduces to single-phase — compute one phase, then exploit symmetry. Unbalanced conditions (common with single-phase loads distributed across phases) require full three-phase analysis using sequence components.

## Transformers

The **transformer** is why AC won. Two coils sharing a magnetic core couple magnetically; AC voltage applied to the primary induces a voltage in the secondary proportional to the turns ratio:

$$\frac{V_1}{V_2} = \frac{N_1}{N_2}$$

Currents scale inversely: $I_2/I_1 = N_1/N_2$. Power is approximately conserved (real transformers have losses in copper resistance and iron hysteresis, but 98-99% efficient is normal for large units).

Applications:
- **Step-up transformers** at generating stations raise voltage from ~20 kV to 230-765 kV for transmission.
- **Step-down transformers** at substations reduce to 4-35 kV for distribution.
- **Distribution transformers** on utility poles step down to 120/240 V (North American residential).
- **Isolation transformers** separate circuits galvanically while passing AC.
- **Instrument transformers** (CTs, PTs) scale current and voltage for metering and protection.

Transformers only work for AC — they require a changing magnetic flux to induce secondary voltage. DC in the primary just dissipates in the winding resistance with no secondary output. This is the fundamental physical reason AC won the transmission battle.

## Harmonics and Distortion

Real AC systems aren't purely sinusoidal. Nonlinear loads — computer power supplies, LED drivers, variable-frequency drives, arc furnaces — draw current in pulsed or distorted waveforms, injecting **harmonics** at multiples of the fundamental frequency (60 Hz, 120 Hz, 180 Hz, and so on).

Harmonics cause:
- Extra heating in transformers and motors.
- Voltage distortion on shared supply circuits.
- Triplen harmonics (3rd, 9th, 15th) add in the neutral of three-phase wye systems, sometimes overloading neutrals.
- Interference with communication circuits.

Utilities limit total harmonic distortion (THD) via standards like IEEE 519. Harmonic filters and active front ends on VFDs mitigate the effects.

## Power Electronics and the AC/DC Boundary

Semiconductor switching devices — diodes, thyristors, IGBTs, MOSFETs — can rectify AC to DC, invert DC to AC, and convert between AC frequencies. This reopens design choices that were settled by the transformer in the 1890s.

- **HVDC transmission**: long overhead lines or submarine cables use DC to avoid reactive power issues, skin effect losses, and phase synchronization problems at line length. China, Brazil, Europe have major HVDC links.
- **Variable-frequency drives**: rectify 60 Hz AC to DC, then invert to adjustable-frequency AC to control motor speed. Ubiquitous in industrial drives, pumps, fans, HVAC, electric vehicles.
- **Grid-tied solar and battery inverters**: produce AC synchronized to grid voltage and frequency from a DC source (PV panels, batteries).
- **Switch-mode power supplies**: the ubiquitous low-voltage DC power supplies in electronics use high-frequency switching (10 kHz to MHz) to allow small, light transformers.

Power electronics have transformed how generation and consumption interface with the grid. Inverter-based resources (wind, solar, batteries) are replacing synchronous generators in much of the grid, which changes grid dynamics — inertia, fault response, voltage support — in ways utilities are still adapting to.

## Measurement and Instrumentation

Electrical measurement is a substantial technical domain:

- **Multimeters**: DC and AC voltage, current, resistance. RMS measurement accuracy varies — cheap units assume sinusoidal waveforms; "true RMS" meters integrate actual waveform.
- **Oscilloscopes**: visualize waveforms in time. Essential for electronics work, power quality diagnosis, signal integrity.
- **Clamp meters**: measure current without breaking the circuit, using magnetic coupling.
- **Power analyzers**: measure real, reactive, and apparent power, power factor, harmonics, and energy.
- **Network analyzers**: characterize impedance, transfer functions, and S-parameters at RF frequencies.

Calibration traceable to national standards (NIST, NPL, PTB) provides accuracy for commercial and scientific applications.

## Safety

AC at line voltage is lethal. Key hazards:

- **Electric shock**: current through the body, not voltage, causes harm. 1-10 mA is painful; 10-30 mA causes muscular contraction preventing release ("can't let go"); 100+ mA through the chest causes ventricular fibrillation.
- **Arc flash**: a short circuit in a power system can produce an arc with tens of kA, temperatures hotter than the sun's surface, and pressure blast waves. Arc-flash PPE is required for work near energized equipment.
- **Ground-fault circuit interrupters (GFCIs)**: detect imbalance between line and neutral currents, indicating leakage (possibly through a person), and trip within milliseconds. Required in wet locations.
- **Arc-fault circuit interrupters (AFCIs)**: detect arcing from loose connections or damaged insulation, a major fire cause.
- **Lockout/tagout**: before working on equipment, de-energize, verify dead, and lock out the disconnect with personal locks.

Safe work practices are codified in standards (NFPA 70E in US, IEC 60364 internationally). Violations cause the preventable deaths of hundreds of workers per year in the US alone.

## Why This Level Matters

AC circuit analysis is the foundation of the entire electric power system and most of electrical engineering practice. The phasor and impedance machinery enables clean analysis of circuits that would be intractable in the time domain. Three-phase systems, transformers, and power factor are central to how electricity is actually generated and delivered. Power electronics are increasingly the interface between renewable energy, batteries, motors, and the grid — all of which depend on understanding AC at its fundamentals.

Electronics, too, uses AC analysis throughout — signal processing, radio, audio, feedback amplifiers, filters — all rely on frequency-domain thinking with phasors and transfer functions. An engineer who can compute impedance, understand resonance, distinguish real and reactive power, and trace power flow through a three-phase system is equipped to analyze systems across a huge swath of the electrical world.

## The Transition to Level 3

L3 turns to **power electronics and motor drives** — how semiconductor switches convert between AC and DC, between voltage levels and frequencies, and how they drive the motors that do most of the useful mechanical work of modern civilization.

Next: [L3 — Power Electronics & Motors](./L3_Power_Electronics_and_Motors.md) *(deferred)*
