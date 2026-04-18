# Level 1: Epidemiology
*Measuring disease in populations; how we know what causes what*

<!-- Evidence Tier: Textbook -->

## The Discipline That Counts the Dead

Epidemiology is the study of how disease, injury, and health outcomes are distributed in populations, and what causes the distribution. It is the scientific backbone of public health.

Epidemiology's basic questions:

- How many people have this disease right now (**prevalence**)?
- How many new cases appear per unit time (**incidence**)?
- Who is getting it, and who isn't (by age, sex, place, occupation, exposure)?
- Why is the rate rising or falling?
- What can we do about it?

Epidemiology is where medicine meets statistics. A clinician sees patients one at a time. An epidemiologist sees the whole distribution and infers what's producing it.

## Counting Right

**Incidence rate** = new cases per person-time at risk. A useful rate for acute conditions. "The incidence of measles in unvaccinated preschoolers in an outbreak is 5 cases per 100 child-weeks of exposure" is an incidence-rate statement.

**Prevalence** = proportion of the population with the condition at a point in time. More useful for chronic conditions. "The prevalence of hypertension in US adults over 60 is about 60%" is a prevalence statement.

**Mortality rate** = deaths per unit population per unit time. Usually expressed per 100,000 person-years. Age-adjusted rates allow comparison across populations with different age structures — crucial because most diseases are age-dependent.

**Case fatality rate** = proportion of people with a disease who die from it. COVID-19 had a CFR of roughly 1–2% in the first wave, varying widely by age, and dropped substantially with vaccines and better treatment. Ebola CFR is 30–90% depending on strain and care quality.

Getting the denominator right is the main epidemiological skill. "10,000 deaths from X this year" is meaningless without knowing whether the population at risk is 100 million or 100 thousand, and whether the rate is rising or falling.

## Study Designs

Epidemiology uses a toolbox of study designs, each with its own strengths and pitfalls.

**Ecologic studies**: compare disease rates across places or time periods. Cheap and fast. Prone to the **ecologic fallacy** — a relationship at the group level may not hold at the individual level. Example: the famous observation that U.S. states with higher gun ownership have more firearm deaths is valid at the state level but doesn't immediately prove individual gun owners are at higher risk.

**Cross-sectional studies**: survey a population at one time, measuring exposure and disease simultaneously. Good for prevalence and associations. Cannot establish temporal order — which came first, the exposure or the disease.

**Case-control studies**: start with cases (people with the disease) and controls (people without), and look backward at their exposure history. Efficient for rare diseases. Vulnerable to **recall bias** (cases remember exposures differently from controls) and **selection bias** (were cases and controls drawn from comparable populations?).

**Cohort studies**: follow a group of exposed and unexposed people over time; measure who develops disease. More expensive but stronger causal inference. The Framingham Heart Study, Nurses' Health Study, UK Biobank are landmark long-running cohorts. They produced most of what we know about cardiovascular risk factors and many other chronic-disease associations.

**Randomized controlled trials (RCTs)**: randomly assign people to exposure or control. This is the gold standard for causation because randomization breaks confounding. Expensive, sometimes unethical (you can't randomize smoking or childhood abuse), sometimes impractical (you can't randomize city-level policies).

**Natural experiments and quasi-experimental designs**: use real-world events to approximate randomization. When a state changes a policy and neighboring states don't, you can compare outcomes in a "difference-in-differences" design. Not as clean as an RCT but useful for questions that aren't randomizable.

## The Problem of Confounding

A **confounder** is a third variable associated with both exposure and outcome, creating a spurious association. Classic example: coffee drinking is associated with lung cancer. The confounder is smoking — smokers drink more coffee, and smokers get lung cancer. Adjust for smoking and the coffee-lung cancer association disappears.

Epidemiology is largely a toolkit for dealing with confounding: stratification, matching, regression adjustment, propensity scoring, instrumental variables, Mendelian randomization. Each has assumptions and limits. A good epidemiologist is paranoid about what they might be missing.

Two concepts worth absorbing:
- **Internal validity**: the study's conclusions about the study population are correct (no bias, no confounding, correct analysis).
- **External validity**: the conclusions generalize to other populations. A trial of a drug in 30-year-old healthy volunteers may not apply to 75-year-olds with multiple conditions.

Both matter. A study can have great internal validity (well-run, proper analysis) but poor external validity (the findings don't apply to the populations we care about).

## The Hill Criteria

Epidemiologist Austin Bradford Hill, in a 1965 address, proposed criteria for inferring causation from association. None is sufficient on its own; together they build the case:

- **Strength**: a large association is harder to explain away.
- **Consistency**: reproducible across studies and populations.
- **Specificity**: a specific exposure produces a specific outcome (less useful for complex multifactorial diseases).
- **Temporality**: cause precedes effect. This one is mandatory.
- **Biological gradient**: dose-response relationship.
- **Plausibility**: mechanism consistent with biology.
- **Coherence**: consistent with other known facts.
- **Experiment**: experimental evidence, when available.
- **Analogy**: similar exposure-disease relationships established elsewhere.

Smoking-lung cancer in the 1950s: the tobacco industry argued for years that correlation didn't prove causation. It turned out to be causation — by every Hill criterion. This is the textbook success story of epidemiology, led by Doll, Hill, and others in the UK, and by Hammond and Horn in the US. By the 1964 Surgeon General's report, the case was settled and policy followed.

## Outbreak Investigation

When a cluster of disease appears, epidemiologists investigate:

1. **Confirm the outbreak**: is the rate actually elevated over baseline?
2. **Define the case**: clinical and/or laboratory criteria for counting.
3. **Count cases**: find them through surveillance systems, healthcare providers, labs.
4. **Describe by time, place, person**: classic epidemic curve, map of cases, demographic breakdown.
5. **Generate hypotheses**: what exposure could plausibly explain the pattern?
6. **Test hypotheses**: often via case-control study comparing cases with matched controls.
7. **Implement control measures**: remove the source, vaccinate the exposed, trace contacts, etc.
8. **Communicate** to public, press, policymakers.

John Snow's 1854 investigation of cholera in London — tracing cases to the Broad Street pump — is the founding case. Modern outbreak response follows the same logic with modern tools. Contact tracing apps, genomic sequencing of pathogen samples, and integrated surveillance data systems are recent additions.

## The Data Systems

Modern public health runs on data systems most people never see:

- **Vital statistics**: births, deaths, cause of death. Required legal reporting in most countries.
- **Disease surveillance**: mandatory reporting of specified diseases (TB, measles, HIV, many others). Passive (physicians report) and active (health departments search for cases).
- **Syndromic surveillance**: aggregates of ER visits, OTC medication sales, absence from work/school — early signals of outbreaks before lab diagnosis.
- **Registries**: cancer registries, birth defect registries, immunization registries.
- **Surveys**: BRFSS in the US, similar systems elsewhere — population health behavior and outcomes.

These systems' quality varies by country and region. Rich countries have better data. Poor countries often have vital statistics gaps — a substantial share of deaths in sub-Saharan Africa go unregistered or unattributed to cause. Improving these systems is a slow, unglamorous, enormously important global health priority.

## What Epidemiology Can and Cannot Do

Epidemiology is excellent at:
- Measuring population-level patterns
- Identifying risk factors that are consistently associated with outcomes
- Evaluating interventions (especially at scale and via RCTs)
- Spotting outbreaks and unusual patterns

Epidemiology is limited at:
- Rapidly proving causation for novel exposures (takes years of multiple studies)
- Individual-level prediction (a risk factor that raises group risk from 0.1% to 0.2% gives little guidance for any single person)
- Questions about rare outcomes in small populations (statistical power is limited)
- Disentangling deeply confounded exposures (diet-disease relationships are famously hard)

Understanding the limits is as important as understanding the capabilities. Bad epidemiology fuels bad policy. Good epidemiology, patiently accumulated and synthesized, has saved more lives than any single clinical intervention.

## Why This Level Matters

Public health is the discipline that reduced infectious mortality from the leading cause of death in 1900 to a secondary cause by 2000, that identified smoking and air pollution and trans fats and lead as population-level health hazards, and that responded (with varying competence) to HIV, SARS, H1N1, Ebola, and COVID-19.

None of that happens without the counting. Everything starts with incidence, prevalence, and mortality rates measured properly. A world without epidemiology would be a world where we do not know whether our interventions work.

## The Transition to Level 2

L2 turns from measurement to action: **disease prevention and control** — vaccines, screening, surveillance, behavior change programs, and the policy interventions that actually reduce morbidity and mortality at population scale.

Next: [L2 — Disease Prevention & Control](./L2_Disease_Prevention_and_Control.md) *(Phase 2D)*
