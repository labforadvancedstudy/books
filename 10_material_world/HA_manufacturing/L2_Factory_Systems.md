# Level 2: Factory Systems
*Flow, inventory, quality, and the long march from craft shop to continuous production*

<!-- Evidence Tier: Textbook -->

## Beyond Individual Processes

Level 1 covered the basic processes by which individual parts get made. Real manufacturing is rarely a single process in a corner. It is dozens of processes organized into a sequence, with parts moving through in volume, with buffers and bottlenecks, with workers and machines and tooling coordinating in time. The system is not the sum of the processes — it is the flow of work through them.

Factory systems determine cost, quality, and speed-to-market far more than individual processing choices do. A well-run factory with mediocre equipment beats a poorly run factory with state-of-the-art equipment, most of the time.

## Production Types

Manufacturing systems cluster into distinct types based on volume and variety:

- **Project (one-off)**: ships, bridges, buildings, custom industrial equipment. Each unit is a project.
- **Job shop**: small batches, high variety. Machine shops serving varied customers. Each batch follows a unique path through generic equipment.
- **Batch production**: medium volume, medium variety. Furniture, specialty chemicals, many consumer goods. Runs of hundreds or thousands between changeovers.
- **Mass production**: high volume, low variety. Automobiles, appliances, electronics. Long runs on dedicated equipment.
- **Continuous process**: extremely high volume, no distinct units. Steel, petrochemicals, paper, glass, cement.

The type determines the layout, the equipment, the skill structure of the workforce, and the unit economics. A job shop and a continuous process factory are essentially different species of enterprise even if nominally in the same industry.

## The Assembly Line

The modern factory was invented by Henry Ford between 1908 and 1913, though many pieces existed before. Ford's contribution was the moving assembly line: the workpiece moves to the worker, each worker has a narrow specialized task, the pace is set by the line speed, and the whole sequence is engineered for flow.

The productivity gain was dramatic. The Model T chassis assembly time dropped from 12.5 hours to 93 minutes. The Model T price dropped by half. Mass consumer access to automobiles became possible.

The downsides became clear too. Work was monotonous, accident rates were high, turnover was enormous (Ford had to double wages to stabilize the workforce). The assembly line concept spread to every durable goods industry and defined 20th-century manufacturing, but the social costs drove a century of labor organizing, workplace regulation, and eventually the search for more flexible, humane production systems.

## Lean Manufacturing (The Toyota System)

Toyota, starting in the 1950s under Taiichi Ohno, developed an alternative — the **Toyota Production System**, later popularized as **lean manufacturing**. Its core principles:

- **Eliminate waste (muda)**: seven categories — overproduction, waiting, transportation, overprocessing, inventory, motion, defects. Each is scrutinized and reduced.
- **Just-in-time (JIT)**: parts arrive at the workstation just when needed, not before. Reduces inventory, exposes problems, tightens the production loop.
- **Pull production (kanban)**: each workstation produces in response to downstream demand, not a pushed schedule. Prevents overproduction.
- **Jidoka (autonomation)**: machines stop automatically when something is abnormal. Operators are empowered to stop the line to fix problems (the "andon cord").
- **Continuous improvement (kaizen)**: small, daily improvements driven by workers, not just engineers or managers.
- **Standardized work**: the current best method is documented and followed until someone proves a better one.
- **Respect for people**: long-term employment, investment in worker skills, shared problem-solving.

The Toyota system achieves lower inventory, higher quality, faster response, and lower cost than a comparable traditional assembly line — when implemented correctly. The difficulty is the "correctly." Many factories have adopted lean terminology without the underlying discipline; the results are mixed.

## Single-Piece Flow

Classic batch production works in lots of 100, 1000, or 10,000 parts. Each workstation does its job on the whole lot, then passes the lot on.

**Single-piece flow** — one unit at a time through a sequence of connected stations — has surprising advantages:

- **Faster feedback**: a defect is caught at the next station, not after 10,000 defective parts are made.
- **Lower inventory**: only work in progress is on the line.
- **Faster throughput time**: the first part is complete much sooner than with batches.
- **Smaller footprint**: less storage needed for in-process parts.

The trade-off is setup time. If changing from product A to product B takes 2 hours on a given machine, you can't economically run lots of 1. But lean manufacturing focused heavily on setup reduction (**SMED** — single-minute exchange of die), enabling lots to shrink to 1 or to a few.

Modern assembly lines for cars, appliances, and electronics approximate single-piece flow or very small lots, producing many product variants on the same line with automated tool changeovers.

## Bottlenecks and the Theory of Constraints

Eliyahu Goldratt's **Theory of Constraints** observes that every system has a bottleneck — the slowest step that limits throughput. Improving non-bottleneck steps does not increase throughput; only improving the bottleneck does.

Practical implications:
- Find the bottleneck. Usually obvious — it's where work piles up and workers wait.
- Exploit it — never let it idle, never let it be starved, never let it work on defective material.
- Subordinate other operations to it — pace everything else to what the bottleneck can absorb.
- Elevate the bottleneck — add capacity, run it longer, offload some of its work.
- When the bottleneck moves, start over with the new one.

The insight is that most manufacturing improvement effort is wasted on non-bottleneck steps. A factory focused on bottleneck improvement can usually double throughput with little capital investment.

## Quality Management

Three evolutionary stages in quality thinking:

**Inspection**: check the finished product, scrap or rework the defects. Treats quality as an output. Expensive because all defects come after all the work is done, and the inspection itself misses some defects.

**Statistical Process Control (SPC)**: monitor the process in real time. Use control charts to detect when a process shifts out of its normal range, before a lot of defective product is made. Developed by Walter Shewhart at Bell Labs in the 1920s; popularized in Japan by W. Edwards Deming after WWII.

**Total Quality Management / Quality at the Source**: build quality into the process so defects don't happen in the first place. Design products and processes for robustness. Train every worker to identify and solve quality problems. Treat quality as everyone's responsibility.

Modern quality standards (ISO 9001, IATF 16949 for automotive, AS9100 for aerospace) certify that a facility has management systems aligned with these ideas. Certification is required for doing business in most industries. Whether certification produces actual quality depends on execution.

**Six Sigma** added statistical rigor to defect reduction — targeting 3.4 defects per million opportunities. In practice, Six Sigma's structured problem-solving methodology (DMAIC: Define, Measure, Analyze, Improve, Control) has been more influential than the 3.4 DPMO target.

## Automation and Robotics

Industrial automation spans a spectrum:

- **Hard automation**: dedicated equipment for a specific task. Cheap per unit at high volume, inflexible to product change.
- **Flexible automation**: programmable equipment (CNC machine tools, industrial robots) that can run different products with software changes. Higher capital cost, flexibility in return.
- **Fixed station robotics**: robots doing welding, painting, assembly at specific points on an otherwise conveyor-based line.
- **Mobile robots (AMRs)**: autonomously navigating factory floor for material movement.
- **Collaborative robots (cobots)**: designed to work alongside humans without safety cages. Smaller, slower, lower payload but enables new applications.

As of 2025, about 4 million industrial robots are in service globally, most in China, Japan, South Korea, Germany, and the US. Automotive and electronics dominate; food, pharmaceutical, and general manufacturing growing.

Full automation ("lights-out" factories) is rare and usually limited to specific processes (semiconductor fabs, some chemical plants). Most "automated" factories still depend heavily on human operators for setup, inspection, troubleshooting, and anything non-routine. The ratio is shifting over decades but slowly.

## Supply Chain

A modern factory does not make everything. It assembles what comes from a network of suppliers. A car has 10,000–30,000 parts from hundreds of first-tier suppliers, each with their own suppliers, extending several tiers deep.

**Tier 1 suppliers**: deliver to the final assembler. Often major global firms themselves (Bosch, Denso, Continental, Magna).

**Tier 2 suppliers**: deliver to Tier 1s. Specialize in components (bearings, fasteners, electronics).

**Tier 3 and below**: raw materials, commodity parts.

Supply chain management handles ordering, inventory, logistics, quality, contracts, risk. Modern supply chains are **just-in-time** — parts arrive at the assembly plant hours before use, not weeks. Inventory is minimized; responsiveness is critical.

This creates fragility. The 2011 Japan earthquake disrupted global automotive supply for months because of specialized semiconductors, pigments, and parts from affected suppliers. The COVID-19 pandemic disrupted countless supply chains through factory closures, port congestion, and demand shifts. The 2021 Suez Canal blockage by the Ever Given affected supply chains globally.

Post-pandemic, many firms are **nearshoring** (moving production closer to end markets), **diversifying suppliers** (reducing single-source risk), and **increasing inventory** in critical items — reversing decades of globalization-era optimization for absolute cost minimization.

## Industry 4.0

The latest wave of manufacturing digitization is sometimes called **Industry 4.0**:

- **IoT (Internet of Things)**: machines, tools, and products networked with sensors reporting status continuously.
- **Big data and analytics**: machine learning applied to production data to predict failures, optimize schedules, reduce defects.
- **Digital twins**: virtual replicas of physical systems, used for simulation, training, and optimization.
- **Additive manufacturing** (integrated with traditional processes for hybrid production).
- **Cobots and AI**: more flexible automation.
- **Cloud and edge computing**: distributed intelligence across the factory.

Hype outruns reality on Industry 4.0 in most places. Actual adoption is uneven. Leading firms in automotive, aerospace, and semiconductors are genuinely operating at high digital integration. Most factories still run on a mix of old and new systems with substantial manual integration.

## Safety

Factories are hazardous environments. Common risks:

- **Mechanical**: moving parts, lifted loads, pinch points. Machine guarding required.
- **Electrical**: lockout/tagout required before maintenance.
- **Thermal**: hot metals, furnaces, welding. Burn protection.
- **Chemical**: solvents, cutting fluids, process chemicals. Ventilation, PPE, exposure limits.
- **Ergonomic**: repetitive motion, awkward postures, manual lifting. Leading cause of chronic injury.
- **Confined spaces, work at heights, heavy vehicle traffic**: specific procedures.

Industrial fatality rates in rich countries have dropped from dozens per 100,000 workers in 1900 to roughly 2 per 100,000 in 2020 (US and EU averages). Most of the improvement comes from safer machinery, better process design, protective equipment, training, and enforcement. Developing countries with weaker regulation and older equipment still have substantially higher rates.

## Why This Level Matters

A product's cost, quality, delivery, and sometimes safety are decided far more by factory systems than by individual process choices. A well-designed factory is as much an engineered object as any product it makes. A poorly-designed factory kills its parent company slowly through cost overruns, delivery failures, and quality escapes.

Understanding factory systems is essential for anyone building or sourcing physical products. You cannot design a product properly without knowing how it will be made; you cannot negotiate with a supplier properly without understanding their factory economics; you cannot run a manufacturing company without understanding flow, quality, and bottlenecks.

## The Transition to Level 3

L3 turns to **materials and energy in manufacturing** — the inputs, their supply, their environmental footprints, and how the manufacturing system fits into the larger circular or linear economy.

Next: [L3 — Materials & Energy Flow](./L3_Materials_and_Energy_Flow.md) *(deferred)*
