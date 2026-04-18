# L4 — Electronics and Signal Processing

<!-- Evidence Tier: Textbook -->

L3 built the world of power — the kilowatts and megawatts that spin motors and deliver electricity. L4 turns to the other half of electrical engineering: **electronics and signal processing**. Where power electronics treats electricity as energy to be moved, signal electronics treats electricity as information to be sensed, amplified, shaped, transmitted, decoded, and computed with. This is the domain of integrated circuits, op-amps, filters, digital logic, analog-to-digital converters, radio frequency systems, and the algorithms that turn voltage waveforms into meaning. Nearly every machine built after 1960 contains a signal chain somewhere in it; the modern information economy is largely a monument to semiconductor electronics and digital signal processing.

## Analog Electronics: The Foundation

Signal electronics begins with **analog circuits** — circuits operating on continuous voltages and currents. The workhorses are resistors, capacitors, inductors, diodes, bipolar junction transistors (BJTs), and MOSFETs. Combinations form amplifiers (inverting, non-inverting, differential), filters (low-pass, high-pass, band-pass, notch), oscillators (Colpitts, Hartley, ring, crystal), mixers, phase-locked loops, and voltage references. The **operational amplifier** (op-amp) is the fundamental building block of modern analog design — a high-gain differential amplifier with near-infinite input impedance and near-zero output impedance that, with negative feedback, implements virtually any linear signal operation. Early op-amps were built with vacuum tubes; the LM741 (1968) and later the LF356, OP07, AD8051, and their successors democratized precision analog design. Contemporary op-amps offer picovolt-scale offsets, nanoampere input currents, and gigahertz bandwidths.

Analog design is deceptively hard. Ideal component equations are only a first approximation; real design must handle **noise** (thermal, shot, flicker), **offset and drift** (with temperature and time), **distortion** (harmonic, intermodulation), **bandwidth limits** (gain-bandwidth product, slew rate), **stability** (phase margin, compensation), and **power supply rejection**. A good analog designer thinks simultaneously about small-signal response, large-signal behavior, stability criteria, noise budget, and layout parasitics — the capacitance of PCB traces and the inductance of ground paths matter at high frequencies.

## Diodes, Transistors, and Semiconductor Physics

The underlying devices rest on semiconductor physics. **Silicon** — the second most abundant element in Earth's crust — dominates because it has a usable bandgap (1.12 eV), forms an excellent native oxide (SiO₂), and can be grown as nearly perfect single crystals. Doping silicon with phosphorus or boron creates n-type and p-type regions. Junctions between them form diodes; stacked junctions or field-effect structures form transistors.

The **MOSFET** (metal-oxide-semiconductor field-effect transistor), especially the CMOS (complementary MOSFET) pair, is the dominant transistor in digital and much analog design. CMOS dissipates power mainly during switching, enabling billions of transistors per chip. Moore's Law — the observation that transistor density doubles roughly every two years — drove a half-century of exponential progress in compute. As dimensions shrink, leakage currents, short-channel effects, and quantum tunneling increase; beyond the 7 nm node, transistor structures have shifted from planar to FinFET to gate-all-around (GAA) architectures. EUV lithography, multiple patterning, strained silicon, high-κ/metal gate stacks, and copper interconnects have each been required breakthroughs.

**Wide-bandgap semiconductors** — silicon carbide (SiC, 3.2 eV) and gallium nitride (GaN, 3.4 eV) — are replacing silicon in high-power and high-frequency applications. GaN transistors now dominate phone fast chargers and 5G base stations. SiC is taking share in EV traction inverters and industrial motor drives. Gallium arsenide (GaAs) and indium phosphide (InP) remain important for optoelectronics and millimeter-wave RF. Silicon photonics integrates optical waveguides on silicon dies, enabling 400 Gbps and 800 Gbps transceivers for data centers.

## Digital Electronics and Logic

**Digital electronics** reduces signals to binary — high/low voltage levels representing 1/0 — and processes them with logic gates. Basic gates (NAND, NOR, AND, OR, XOR, inverter) compose into combinational logic (adders, multipliers, decoders, multiplexers) and sequential logic (flip-flops, registers, counters, state machines). Digital design gains enormous leverage from **abstraction**: the physics of transistors can be forgotten once gate-level timing is characterized; gate-level timing can be forgotten once RTL (register-transfer level) behavior is verified; RTL can be forgotten once the architectural model is correct.

Digital ICs fall into several families. **Microprocessors** (CPUs) execute stored programs — x86 and ARM dominate, with RISC-V rising. **GPUs** handle massively parallel floating-point work, now the substrate of AI inference and training; Nvidia's H100 and successors are the economic engine of the current AI boom. **Application-specific integrated circuits (ASICs)** encode a fixed function — Bitcoin miners, Google TPUs, Broadcom switch silicon. **Field-programmable gate arrays (FPGAs)** — primarily AMD/Xilinx and Intel/Altera — are reconfigurable digital fabrics used for network processing, defense, and prototyping. **System-on-chip (SoC)** designs combine CPU, GPU, memory controller, radio, and specialized accelerators on a single die; Apple's M-series and Qualcomm's Snapdragon exemplify the approach.

## Signal Processing Fundamentals

**Digital signal processing (DSP)** is the mathematical manipulation of sampled signals. The **Nyquist-Shannon sampling theorem** — a signal band-limited to frequency B can be perfectly reconstructed from samples taken at rate ≥ 2B — is the foundational result. Real ADCs approach this ideal with anti-alias filters, sample-and-hold circuits, and quantization. Common ADC architectures include successive approximation (SAR, high resolution, moderate speed), pipelined (high speed, moderate resolution), sigma-delta (very high resolution via oversampling and noise shaping, used in audio and instrumentation), and flash (fastest, power-hungry, used in oscilloscopes and radars).

Once signals are digitized, a toolkit of algorithms shapes them. The **Fourier transform** decomposes signals into frequency components; the fast Fourier transform (FFT), rediscovered by Cooley and Tukey in 1965, is the workhorse of spectrum analysis, modulation, compression, and filtering. **Digital filters** come in two families: FIR (finite impulse response, always stable, linear phase if symmetric, used in audio and communications) and IIR (infinite impulse response, computationally efficient, can match classical analog filter responses — Butterworth, Chebyshev, elliptic). Adaptive filters (LMS, RLS) adjust their coefficients to track changing signal statistics, used in echo cancellation, equalization, and noise cancellation.

**Compression** algorithms — a core application of signal processing — exploit statistical structure. MP3, AAC, and Opus compress audio by discarding imperceptible frequency content (psychoacoustic masking); JPEG and H.264/H.265/AV1 compress images and video with discrete cosine transforms and motion compensation; FLAC and PNG compress losslessly. Modern codecs require billions of operations per second, implemented on dedicated hardware blocks in phones and GPUs.

## Communications and Modulation

Most signal processing exists to enable **communications**. Information theory (Shannon, 1948) establishes the channel capacity — the maximum error-free data rate for a given bandwidth and signal-to-noise ratio: C = B log₂(1 + S/N) bits per second. Modern systems approach the Shannon limit remarkably closely through sophisticated modulation and coding.

**Modulation** maps information onto carrier waveforms. Amplitude, frequency, and phase modulation (AM, FM, PM) date from the early 20th century. Digital modulations — QAM (quadrature amplitude modulation, carrying multiple bits per symbol), OFDM (orthogonal frequency-division multiplexing, dividing bandwidth into many narrow subcarriers), CDMA (code-division multiple access, spreading signals across a wider band) — power modern wireless. 4G LTE used OFDM on the downlink; 5G extends this with massive MIMO (multiple-input multiple-output, using many antennas coherently), millimeter-wave bands, and beamforming. Wi-Fi 6 and 7 apply similar techniques in unlicensed bands.

**Error correction coding** — Reed-Solomon, convolutional codes with Viterbi decoding, turbo codes, LDPC (low-density parity check), and polar codes — adds redundancy to detect and correct bit errors. Modern systems get within a dB or two of Shannon capacity. Optical communications add dense wavelength-division multiplexing (DWDM), coherent detection with digital signal processing at the receiver, and forward error correction to push undersea cables past 20 Tbps per fiber pair.

## RF and Microwave Engineering

**Radio-frequency (RF) engineering** addresses signals roughly from 3 kHz to 300 GHz. Above ~100 MHz, lumped-element circuit models break down and transmission-line and wave thinking takes over. Characteristic impedance, standing waves, Smith charts, S-parameters, and impedance matching networks are the core toolkit. PCBs become microwave structures; connectors, cables, and packaging are critical.

Modern RF design integrates low-noise amplifiers (LNAs), mixers, oscillators (VCOs and PLL synthesizers), power amplifiers (PAs, the power-hungry component in cellphones), filters (SAW, BAW, MEMS-based), and antennas. Integration into CMOS (RF-CMOS) has enabled entire smartphone radio front-ends on single dies. **Millimeter-wave** (30–300 GHz) design for 5G, automotive radar, and satellite communications requires GaAs, SiGe, or GaN technologies. **Phased-array** systems steer beams electronically by controlling phase across many antenna elements — the basis of modern radar, satellite communications, and 5G base stations.

## Mixed-Signal Design

Real systems straddle analog and digital domains. A sensor produces an analog voltage; an ADC digitizes it; DSP processes it; a DAC converts back to analog for a speaker or actuator. **Mixed-signal design** is notoriously tricky because digital switching creates noise that couples into analog circuits via shared supplies, grounds, and substrates. Good practice includes separate analog and digital power domains, star grounding, guard rings, and careful PCB layout.

**Data converters** — ADCs and DACs — are often the system bottleneck. High-speed converters for radar and communications (e.g., Analog Devices' AD9082 or RFSoC-integrated converters) sample at tens of gigasamples per second. High-precision converters for instrumentation resolve 24 bits (one part in 16 million) or more. Sigma-delta converters trade sample rate for resolution, using oversampling and feedback to push quantization noise out of the band of interest.

## Sensors and Instrumentation

The front end of a signal chain is the **sensor** — the device that converts a physical quantity (temperature, pressure, acceleration, light, sound, magnetic field, chemical concentration) into an electrical signal. Modern MEMS (micro-electromechanical systems) fabrication puts accelerometers, gyroscopes, microphones, and pressure sensors on silicon dies at low cost — smartphones contain dozens. CMOS image sensors have displaced CCDs for nearly all imaging applications. Optical sensors (photodiodes, photomultipliers, SPADs for LiDAR), magnetic sensors (Hall effect, GMR, TMR for automotive and industrial sensing), and biosensors (glucose monitors, electrochemical sensors) round out the landscape.

Sensor signals are weak and noisy. **Instrumentation amplifiers** — differential amplifiers with very high common-mode rejection — pull signals out of ground loops and electromagnetic interference. Lock-in amplifiers extract signals buried deep in noise by synchronous detection. Precision measurement remains a mix of clever analog and skilled DSP.

## PCB Design and EMI/EMC

Getting a signal chain to work on the bench is only half the battle; getting it to work in a product requires **PCB design** and **EMI/EMC** (electromagnetic interference / compatibility) engineering. PCBs route signals through copper traces on layered fiberglass substrates. Modern boards have 4–20 layers, with carefully controlled impedance for high-speed signals, dedicated power and ground planes, and thermal management vias. Signal integrity issues — reflections, crosstalk, ground bounce, power integrity — dominate above a few hundred megahertz. Tools like high-speed SPICE, IBIS models, and 3D electromagnetic simulators are standard.

Products must also pass regulatory testing: FCC Part 15 in the US, CE marking in Europe, and similar regulations elsewhere. Emissions limits (how much noise a product radiates) and immunity requirements (how much interference it must tolerate) shape design from chip selection to enclosure material. Shielded cables, filtering at I/O, careful clock tree design, and metal enclosures are routine mitigations.

## Power for Electronics

Though L3 covered power electronics, **point-of-load power** is integral to electronic design. Modern digital chips need multiple precise voltage rails (0.6 V core, 1.0 V SRAM, 1.8 V I/O, 3.3 V interface) at currents from milliamps to hundreds of amps, with tight tolerance and fast transient response. Switching regulators (buck, boost, buck-boost) dominate above a few hundred milliwatts; linear regulators (LDOs) remain important for low-noise analog supplies and final filtering. Power integrity — ensuring voltages stay within spec under load changes — is now a core part of chip and board design.

Batteries and energy harvesting complicate the problem. Smartphones manage dozens of power domains with dedicated PMICs (power management ICs). IoT sensors operating on harvested microwatts must gate circuits aggressively and use ultra-low-power microcontrollers (e.g., ARM Cortex-M0+, RISC-V sub-threshold designs).

## Emerging Frontiers

Several frontiers are reshaping electronics:

**Silicon photonics**: integrating optical waveguides, modulators, and detectors on silicon dies. Short-reach data center interconnects increasingly use photonic integration; AI training clusters need petabits of optical bandwidth. Co-packaged optics — photonic dies in the same package as compute dies — is an active area.

**Neuromorphic and in-memory computing**: von Neumann architectures move data between memory and compute, burning energy on data movement. Neuromorphic chips (Intel Loihi, IBM TrueNorth) and analog in-memory compute (various startups) perform computation where data lives, promising efficiency gains for sparse, event-driven AI workloads.

**Quantum electronics**: superconducting qubits (Google, IBM), trapped ions (IonQ, Quantinuum), silicon spin qubits (Intel), and photonic approaches (PsiQuantum, Xanadu) are competing architectures for quantum computers. Electronics to control qubits — dilution refrigerator cables, cryogenic CMOS, microwave pulse generation — is as challenging as the qubits themselves.

**AI-assisted design**: ML is entering chip design (floorplanning, placement, timing optimization) and analog design (synthesizing amplifier topologies). Google's RL-based floorplanner contributed to TPU designs. Cadence and Synopsys both integrate ML optimizers. Design productivity, not just transistor count, now limits progress.

**Flexible and printed electronics**: thin-film transistors on plastic and paper substrates, printed with inkjet or roll-to-roll processes, enable displays, wearables, and disposable medical sensors. Organic LEDs (OLEDs) dominate premium smartphone displays and are taking share in TVs.

## The Semiconductor Industry

Behind electronics sits one of the world's most concentrated and consequential industries. **TSMC** produces roughly half of the world's leading-edge logic chips, with Samsung Foundry and Intel Foundry Services competing. **ASML** has a monopoly on EUV lithography machines — each system costs $200M+ and comprises 100,000 parts. **Applied Materials, Lam Research, KLA, Tokyo Electron** dominate deposition, etch, and metrology equipment. Design tools consolidate around **Cadence, Synopsys, Siemens EDA**. Memory (DRAM and NAND flash) is an oligopoly of Samsung, SK Hynix, Micron, with emerging Chinese players. The **CHIPS Act** (US, $52B), EU Chips Act (€43B), and parallel programs in Japan, Korea, and China reflect the strategic importance of regaining or maintaining semiconductor manufacturing capacity. Geographic concentration of advanced fabrication in Taiwan, with ASML in the Netherlands and EDA in the US, creates systemic risks that motivate current policy and capacity investments.

## Why This Level Matters

Signal electronics is the substrate of the information age. Every measurement, communication, computation, and perception system — from the voltage sensor in a pacemaker to the transceiver in a cell tower to the GPUs training AI models — is ultimately built from the building blocks described here. The field sits at the interface of physics (semiconductor devices, electromagnetic waves), mathematics (Fourier analysis, information theory, linear algebra), and engineering (design, manufacturing, testing). Progress has compounded across decades because abstractions hold — transistors, gates, blocks, and systems each provide clean interfaces — even as underlying physics grows stranger at the nanoscale.

The macroscale consequence is that the information technology industry has become both economically dominant and strategically decisive. Advanced semiconductors are the modern equivalent of steel in the 20th century. Countries that design and fabricate them shape the world's economic and military capabilities. The craft covered here — from analog circuit design to digital architecture to signal processing algorithms — is what turns physics into product, and product into civilizational infrastructure.

## The Transition to Level 5

L5 will bring together power and signal electronics into an integrated view of **intelligent electrical systems** — the co-design of power, sensing, communication, and computation that characterizes modern electric vehicles, renewable grids, autonomous machines, and smart infrastructure. At that level the division between "power" and "signal" electronics blurs: a grid inverter is also a communicating, self-monitoring, software-defined device; an EV is a computer on wheels; a modern wind turbine is a distributed control system. L5 explores how electrical engineering increasingly becomes systems engineering.

Next: [L5 — Intelligent Electrical Systems](./L5_Intelligent_Electrical_Systems.md) *(deferred)*
