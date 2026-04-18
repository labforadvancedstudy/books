# Level 3: Climate Dynamics
*The energy budget, feedbacks, models, and what drives climate change*

<!-- Evidence Tier: Textbook -->

## From Weather to Climate

L2 covered weather systems — the day-to-day machinery of highs, lows, fronts, storms. L3 zooms out. Weather is the atmospheric state now; climate is the statistics of atmospheric state over decades to millennia. Climate dynamics asks why the global mean temperature is what it is, what makes it change, how regional patterns (monsoons, El Niño, jet streams) are organized, and how to predict future climate under changing conditions.

The distinction matters because the tools differ. Weather forecasting is an initial-value problem — given today's atmosphere, where does it go? Climate projection is a boundary-value problem — given external forcings (solar, CO₂, aerosols) and internal feedbacks, what's the statistical equilibrium of the atmosphere-ocean-ice-land system? Weather is predictable out to ~10-14 days before chaos dominates. Climate is predictable in the statistical sense over much longer horizons, constrained by energy conservation and physics.

## Earth's Energy Balance

The first-order equation of climate is simple. Earth receives solar radiation and radiates infrared back to space. At equilibrium, incoming = outgoing.

Incoming solar at top of atmosphere: ~1361 W/m² (solar constant) × π r² / 4π r² = ~340 W/m² averaged over Earth's surface. Albedo reflects ~30% back; ~240 W/m² is absorbed.

Earth radiates as a near-blackbody at infrared wavelengths. At equilibrium T (SB law), $\sigma T^4 = 240$ W/m² gives T ≈ 255 K = -18°C. But observed surface temperature is ~288 K = 15°C.

The 33 K difference is the **greenhouse effect**. Water vapor, CO₂, methane, other gases absorb outgoing IR and re-emit, partially trapping heat. Without greenhouse gases, Earth would be frozen; with too many, it overheats. This is not a controversy — it's radiative physics, confirmed by satellite, balloon, and surface measurements of IR spectra.

Doubling CO₂ increases radiative forcing by ~3.7 W/m². Climate sensitivity — how much warming results — depends on feedbacks.

## Feedbacks

**Climate feedbacks** amplify or dampen initial forcings. Main ones:

**Water vapor feedback** (strongly positive): warmer air holds more water vapor (Clausius-Clapeyron, ~7% per °C). Water vapor is itself a strong greenhouse gas. Roughly doubles CO₂ sensitivity.

**Ice-albedo feedback** (positive): ice reflects sunlight; loss of ice exposes darker surfaces. Critical at high latitudes — Arctic amplification, paleoclimate transitions.

**Cloud feedback** (uncertain, likely moderately positive): warmer temperatures change cloud amount, altitude, and type. Low clouds cool (reflect sunlight); high clouds warm (trap IR). Net effect is the largest remaining uncertainty in climate sensitivity.

**Lapse rate feedback** (negative in tropics): upper troposphere warms more than surface, increasing outgoing IR efficiency. Partially offsets water vapor feedback.

**Carbon cycle feedback**: warming affects CO₂ and CH₄ release from oceans, soils, permafrost. Operates on multiple timescales.

**Planck feedback**: simply that warmer surface emits more IR. The only unambiguously strong negative feedback, always present.

**Equilibrium climate sensitivity (ECS)** — warming at equilibrium from doubled CO₂ — is estimated at 2.5-4 K by IPCC AR6, with a best estimate near 3 K. The uncertainty range has tightened over the decades but remains substantial, driven mainly by cloud feedback uncertainty.

## General Circulation

The atmosphere redistributes heat from equator to poles via the **general circulation**:

- **Hadley cells** (0-30°): warm air rises at equator, descends at subtropics. Drives trade winds. Expanding northward under warming — a recent observation.
- **Ferrel cells** (30-60°): thermally indirect; driven by eddy fluxes. Contains the storm tracks.
- **Polar cells** (60-90°): weak direct circulation.

The **jet streams** sit at the edges of these cells, especially the polar front jet (~60°) and subtropical jet (~30°). Jet stream waves (Rossby waves) organize much of midlatitude weather.

The **Walker circulation** is the east-west equatorial Pacific circulation — low pressure and ascent over Indonesia, high pressure and descent over the eastern Pacific. El Niño and La Niña are Walker circulation perturbations (see below).

**Monsoons**: seasonal reversal of winds and rainfall over continents heated by solar insolation. Asian, African, North American, South American monsoons are all important regional features. Summer monsoons bring most of the annual rainfall in these regions.

## Ocean Circulation and ENSO

The ocean stores vastly more heat than the atmosphere (~1000× for top few hundred meters). Ocean circulation moves heat, carbon, nutrients, and drives much climate variability.

**ENSO (El Niño-Southern Oscillation)**: coupled ocean-atmosphere oscillation in the tropical Pacific.
- **La Niña**: strong trade winds, warm water pushed west (Indonesia/Australia), cold upwelling in east Pacific. Droughts in Americas, floods in Asia.
- **El Niño**: weakened trade winds, warm water sloshes east, upwelling suppressed. Weather patterns globally shift — drought in Australia, floods in Peru, altered jet streams.

ENSO cycles every 3-7 years, driving the largest interannual climate variability globally. Affects agriculture, fisheries, tropical cyclones, droughts.

Other modes: Pacific Decadal Oscillation (PDO, ~20-30 years), Atlantic Multidecadal Oscillation (AMO, ~60-80 years), North Atlantic Oscillation (NAO), Indian Ocean Dipole, Southern Annular Mode. All contribute to regional climate variability.

**Thermohaline circulation** (the "conveyor belt"): dense, cold, saline water forms in the North Atlantic (Nordic Seas) and Southern Ocean, sinks to abyss, flows slowly through deep oceans, eventually upwells. Transports heat northward in Atlantic; shuts down in some paleoclimate events, with abrupt climate consequences. Evidence suggests Atlantic Meridional Overturning Circulation (AMOC) is weakening under greenhouse warming, though magnitude and future trajectory are uncertain.

## Paleoclimate

Earth has been much warmer and much colder than today over geological time. Paleoclimate data — ice cores, ocean sediment cores, tree rings, speleothems, corals, pollen, fossil assemblages — reconstruct past climates.

**Ice cores** (Antarctic, Greenland): air bubbles preserve ancient atmosphere; isotopes in ice track temperature. Go back ~800,000 years (EPICA Dome C). Show glacial-interglacial cycles with CO₂ varying 180-280 ppm and Antarctic temperature swings of 8-10°C.

**Marine sediments**: foraminifera shells record temperature (Mg/Ca) and ice volume (δ¹⁸O). Extend back tens of millions of years. Document the Cenozoic cooling — from ice-free poles 50 Ma to today's bipolar glaciation.

**Past warm periods**:
- **Paleocene-Eocene Thermal Maximum (PETM, ~56 Ma)**: rapid warming of ~5-8°C, driven by carbon injection of ~3000-7000 GtC over <20 kyr. Analog for anthropogenic emissions; shows what rapid CO₂ release does.
- **Mid-Pliocene (~3 Ma)**: CO₂ ~400 ppm (similar to present), global temperature ~2-3°C warmer, sea level ~10-20 m higher.
- **Last Interglacial (~125 ka)**: ~1°C warmer than pre-industrial, sea level ~6-9 m higher.

**Abrupt climate change**: Dansgaard-Oeschger events (7-15°C Greenland warming in decades), Younger Dryas (1300-year cold reversal during last deglaciation). Evidence that climate has tipping points and can reorganize quickly.

## Climate Models

**General Circulation Models (GCMs)** — more recently called Earth System Models (ESMs) — solve the equations of atmospheric and oceanic fluid dynamics on 3D grids, coupled to land surface, sea ice, biogeochemistry, and increasingly ice sheets.

Modern GCMs have grids of ~25-100 km horizontal resolution, ~40-60 vertical levels, and tens of species of aerosols and chemistry. Run on supercomputers for months to centuries of simulated time.

Model verification: hindcasts of 20th-century warming (including internal variability), paleoclimate simulations, seasonal forecasting, cloud and radiation radiative transfer comparison against satellites.

Intercomparison projects (CMIP6, the current generation) coordinate dozens of models from institutions worldwide. Consistent features (warming, water vapor increase, Hadley expansion, polar amplification) are considered robust; varying features (precipitation patterns, cloud response, ENSO projections) show residual uncertainty.

**Biases** exist — models are systematically wrong about some things (tropical precipitation, Arctic sea ice details, clouds). Constrained by observations and physical principles, models nonetheless remain the best tools for projecting future climate.

## Anthropogenic Climate Change

Human activities are now the dominant climate forcing:

- **CO₂**: up from 280 ppm pre-industrial to 420+ ppm in 2024. Rising ~2-3 ppm/yr.
- **CH₄ (methane)**: up from ~720 ppb to ~1900 ppb.
- **N₂O**: up from ~270 ppb to ~335 ppb.
- **Halocarbons**: entirely anthropogenic; CFCs peaked post-Montreal Protocol, HFCs rising.
- **Aerosols**: net cooling effect, partially offsetting greenhouse warming; declining as air pollution controls improve.

Observed changes:
- **Surface temperature**: ~1.2-1.3°C above pre-industrial (2024). Land warms faster than ocean; Arctic faster than mid-latitudes.
- **Sea level rise**: ~20 cm since 1900, accelerating to >4 mm/yr currently.
- **Glaciers and ice sheets**: nearly all glaciers retreating; Greenland and Antarctic ice sheets losing mass.
- **Arctic sea ice**: September extent has roughly halved since 1979.
- **Ocean heat content**: increasing rapidly; ~90% of excess energy goes into oceans.
- **Weather extremes**: attribution studies show clear human fingerprints on specific heat waves, floods, droughts.

The IPCC AR6 assessment is unequivocal: "It is unequivocal that human influence has warmed the atmosphere, ocean and land." The science on this is settled; debate is about magnitude, pace, regional specifics, and policy response.

## Why This Level Matters

Climate is arguably the single most consequential physical system humans are altering. Decisions about energy, agriculture, infrastructure, migration, finance, public health, and international security all sit downstream of climate trajectories. Understanding climate dynamics — energy balance, feedbacks, internal variability, paleoclimate analogs, model strengths and limitations — is essential for separating robust conclusions from speculation in a domain saturated with both scientific output and public controversy.

The same understanding illuminates cognate questions: what would make Mars habitable? what drove ice ages? what happens at a mass extinction boundary? why are some exoplanets in the habitable zone but unlikely to be habitable? Climate physics is planetary physics.

## The Transition to Level 4

L4 turns to **climate impacts and adaptation** — the translation from physical climate changes to consequences for water availability, agriculture, ecosystems, health, economies, migration, and the infrastructure decisions societies must make across the 21st century.

Next: [L4 — Climate Impacts & Adaptation](./L4_Climate_Impacts_and_Adaptation.md) *(deferred)*
