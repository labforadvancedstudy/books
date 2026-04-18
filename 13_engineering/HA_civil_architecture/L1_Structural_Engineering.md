# Level 1: Structural Engineering
*Loads, beams, columns, frames — the art of making buildings stand up*

<!-- Evidence Tier: Textbook -->

## The Core Problem

A structural engineer answers one question about every part of a building or structure: will this piece carry its loads safely, without breaking and without deflecting too much, for its design life?

That question is hard because:

- **Loads** are uncertain (dead, live, wind, snow, seismic, thermal, construction).
- **Materials** vary (concrete strength, steel yield, wood grade).
- **Analysis** is approximate (beam theory, plate theory, finite element models — all simplifications of reality).
- **Construction** introduces defects (concrete voids, misaligned rebar, missed welds).
- **Service life** exposes the structure to fatigue, corrosion, impact, and neglect.

Codes, experience, and factors of safety bridge the gap between what the math predicts and what the real world delivers. A competent engineer knows not just the formulas but the places where reality diverges from the formulas, and designs accordingly.

## Loads

Structural design begins with enumerating loads:

- **Dead loads**: the weight of the structure itself and permanent attachments. Well-known because they don't change.
- **Live loads**: occupancy-related loads — people, furniture, stored goods. Code-prescribed minimums based on occupancy: offices, 2.4 kPa; residential, 1.9 kPa; parking garages, 2.4 kPa; public assembly, 4.8 kPa.
- **Snow loads**: depend on location, roof geometry, exposure. From under 0.5 kPa in warm climates to over 5 kPa in mountain regions.
- **Wind loads**: depend on wind speed, building shape, height, exposure. Modeled as pressures that vary over the building surface.
- **Seismic loads**: lateral forces from ground motion. Depend on seismic hazard, soil type, building period, structural system.
- **Thermal loads**: stresses from temperature changes, especially in long bridges or steel structures.
- **Construction loads**: loads that apply only during construction (form pressure, lifting forces, crane loads).

Engineers combine these with load factors. Dead load might be factored 1.2, live load 1.6, wind 1.0 — in a combination like 1.2D + 1.6L + 0.5W, and several other combinations are checked. The controlling combination is the worst case for each element.

## Materials

Three dominant structural materials:

**Steel**: high strength (yield 250–700 MPa depending on grade), ductile, predictable, fire-vulnerable (requires fireproofing in buildings). Made in rolled shapes (I-beams, wide-flange beams, channels, angles, tubes). Connected by welding, bolting, or riveting (rivets obsolete in buildings but still used in heavy industry and historic aviation). Life-cycle: indefinite if protected from corrosion.

**Reinforced concrete**: strong in compression (20–80 MPa typical), weak in tension, so embedded steel reinforcing bars (rebar) carry the tension. Can be cast in place (formwork at the site) or precast (cast off-site, trucked in, connected). Connected by lap splices, mechanical couplers, or post-tensioning tendons. Life-cycle: long if reinforcement is well-protected from corrosion; dramatic failure if chloride or carbonation reaches the rebar.

**Timber**: lower strength and stiffness per cross-section than steel or concrete, but cheap, renewable, and constructable with ordinary skills. Traditional in residential construction; increasingly used for mid-rise with engineered wood products (glulam, cross-laminated timber) capable of multi-story buildings. Fire-vulnerable but surprisingly fire-resistant in large sections (char layer protects inner wood for a while).

Other materials have specific niches: masonry (historical, load-bearing walls), aluminum (lightweight, corrosion-resistant, expensive), composites (aerospace, marine, specialty).

## Structural Systems

The arrangement of structural elements is the **structural system**:

- **Load-bearing walls**: walls themselves carry gravity and sometimes lateral loads. Traditional for low-rise masonry and wood buildings.
- **Post-and-beam**: columns carry gravity loads; beams span between them; walls are non-structural cladding. Standard for mid- and high-rise steel and concrete.
- **Frames**: beams and columns rigidly connected to form moment-resisting assemblies. Carry gravity and lateral loads.
- **Braced frames**: beam-column frames plus diagonal braces for lateral stiffness.
- **Shear walls**: solid walls (concrete or braced) that resist lateral loads by acting as large vertical cantilevers.
- **Core-and-outrigger**: tall buildings with a concrete core for stiffness, with outrigger arms connecting the core to perimeter columns for additional lateral resistance.
- **Tubular structures**: outer walls act together as a large tube, efficient for very tall buildings.

Each system has trade-offs: economy, architectural flexibility, construction speed, seismic performance. Taller buildings require stiffer lateral systems because wind and seismic forces scale faster with height than gravity does.

## Beams, Columns, Frames

**Beams** carry loads in bending. Cross-sections are designed so that the resulting bending stress plus shear stress stays below material limits. Standard design equations, combined with codes, determine allowable spans and required cross-sections.

**Columns** carry axial loads in compression. Design must check both stress (compressive yield) and stability (buckling). Long slender columns fail by buckling; short stubby columns fail by crushing. The **slenderness ratio** (effective length / radius of gyration) tells you which regime controls.

**Frames** combine beams and columns with rigid connections. Lateral loads produce combined axial, shear, and bending in frame members. Moment-resisting frames are compact but less stiff than braced frames; braced frames are stiffer but take up more architectural space.

Real analysis is done with software — STAAD, ETABS, SAP2000, RAM, and similar structural analysis packages. The engineer still has to understand what the software is doing, catch analysis errors, and translate analysis output into buildable details.

## Seismic Design

In seismic regions (California, Japan, Chile, New Zealand, Mexico, parts of China, Turkey, Indonesia, and many others), lateral loads from earthquakes often control design.

Key concepts:

- **Base shear**: the total horizontal force at the base of the building during an earthquake. Depends on building mass, fundamental period, soil conditions, and seismic hazard.
- **Response spectrum**: a curve showing how buildings of different natural periods respond to the design earthquake. Short-period buildings (stiff, low-rise) feel higher accelerations; long-period buildings (flexible, tall) feel lower accelerations but larger displacements.
- **Ductility**: the ability to deform plastically without breaking. Ductile systems dissipate seismic energy and survive much larger earthquakes than their elastic capacity alone would suggest.
- **Capacity design**: designing specific elements to fail first in a controlled, ductile manner, while protecting critical elements from damage.
- **Base isolation**: placing the building on flexible bearings that decouple it from ground motion. Expensive but effective; used for hospitals, data centers, historical buildings, and some critical residential.

Modern seismic codes are sophisticated and work. The 2010 Chile earthquake (M8.8) and 2011 Japan earthquake (M9.0) both caused substantial damage but building collapses were rare in well-coded regions. The 2023 Turkey earthquake (M7.8), by contrast, saw widespread collapses in areas where code enforcement had been weak — a political-economic failure, not a scientific one.

## Foundations

Every structure transfers loads to the ground through its **foundation**. Types:

- **Spread footings**: enlarged pads under columns, spreading the load over enough soil area that bearing pressure is acceptable.
- **Mat (raft) foundations**: a continuous slab under the entire building. Used in poor soils or for heavy structures.
- **Piles**: driven or drilled deep into firm strata to transfer loads past soft surface soil. Used for tall buildings, bridges, and any structure on weak soil.
- **Caissons**: large-diameter drilled shafts, often used for bridge piers.

Foundation design is geotechnical engineering — a related discipline covering soil mechanics, settlement, bearing capacity, slope stability. Structural engineers work with geotechnical engineers to specify foundations. Structural failures with "structural" in the news are often foundation or geotechnical failures.

## Codes

Structural design follows codes — documents published by professional organizations or governments that specify minimum requirements. In the US, the International Building Code (IBC) incorporates material-specific standards (ACI 318 for concrete, AISC 360 for steel, NDS for wood). Other countries have equivalents: Eurocodes in Europe, various national codes.

Codes are updated every few years based on new research, lessons from failures, and material developments. A building designed to the 2006 IBC is different from one designed to the 2024 IBC — mostly stricter seismic, wind, and energy requirements.

Codes are minimums. An engineer can (and sometimes should) design more conservatively. An engineer may not design less conservatively without very good reason and clear documentation, because code is the legal floor.

## Construction and Quality

The best design is worthless if construction doesn't follow it. Structural quality assurance involves:

- **Submittals**: contractor submits shop drawings, material certificates, and testing plans; engineer reviews and approves.
- **Inspections**: special inspection of specific elements (welds, high-strength bolts, concrete placement) by independent inspectors.
- **Testing**: concrete cylinder strength, steel coupon tests, weld ultrasonic testing.
- **Field observation**: engineer visits site periodically to check progress and catch problems.
- **Record drawings**: documentation of as-built conditions, often differing in small ways from the original design.

Failures like the Surfside condo collapse (Florida, 2021) typically trace to a combination of long-term deterioration (corrosion, design deficiencies, deferred maintenance) rather than a single construction error. Prevention is maintenance plus inspection over decades, not just good construction.

## Why This Level Matters

Structural engineering is the discipline that lets you build things taller, longer-span, and more complex than gravity would naturally allow. Every modern building, bridge, tower, and dam is a specific arrangement of materials and forms chosen to resist loads with acceptable safety margins.

A structural failure is usually catastrophic — people die, property is destroyed, liability is enormous, the public loses confidence. A well-executed structural design is invisible to the public — the building just works, year after year, through weather and use.

The vast majority of structures work precisely because this discipline works.

## The Transition to Level 2

L2 turns to the **architectural** side — how buildings are designed for human use and experience, not just structural adequacy. Materials, light, circulation, program, and the relationship between structure and architectural intent.

Next: [L2 — Building Design](./L2_Building_Design.md) *(Phase 2D)*
