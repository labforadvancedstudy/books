# L4: Population Genetics
## Evolution as Mathematics

![[evolution_L4_cover.jpg]]

> "Evolution is the change in allele frequencies in a population over time."
> — Theodosius Dobzhansky, Geneticist

Forget organisms for a moment. Forget survival and death, mating and birth. Think instead of genes flowing through populations like water through a landscape.

This is population genetics: evolution viewed as changing frequencies, as mathematics in motion. It transforms Darwin's insights into equations we can solve.

## The Gene Pool

Imagine all the genes in a population as colored marbles in a vast pool. Red marbles are one allele, blue another. Each generation, we reach in, grab random pairs to make individuals, let them live and die according to their colors, then remix the survivors back into the pool.

The question becomes: How do marble frequencies change over time?

Sometimes red increases. Sometimes blue. Sometimes they stay constant. Sometimes one color disappears forever. The rules governing these changes are the laws of evolution.

## Hardy-Weinberg: The Null Hypothesis

In 1908, mathematician G.H. Hardy was annoyed. Biologists claimed that dominant traits should increase over time. Hardy showed they were wrong with a simple equation:

p² + 2pq + q² = 1

Where p and q are allele frequencies, and p², 2pq, and q² are genotype frequencies.

The shocking result: In a large, randomly mating population with no selection, mutation, or migration, allele frequencies never change. Generation after generation, same frequencies.

This is evolution's null hypothesis. Like Newton's first law - objects at rest stay at rest - Hardy-Weinberg says gene frequencies at rest stay at rest. Change requires a force.

## The Forces of Evolution

**Selection**: The directing force. If red marbles survive 1% better than blue, red frequency increases. Small advantages compound over generations. We can calculate exactly how fast.

The equation: Δp = sp(1-p) / (1-2sp)

Where s is selection coefficient, p is allele frequency. Plug in numbers, predict the future.

**Mutation**: The creative force. Blue marbles occasionally turn red, red turns blue. Typical rates: 10⁻⁸ per base pair per generation. Seems tiny, but with billions of base pairs and millions of individuals, new mutations appear constantly.

Mutation alone is weak. It would take 10,000 generations to change an allele frequency from 0.5 to 0.6 by mutation alone. But mutation creates the variation selection needs.

**Gene Flow**: The mixing force. New individuals arrive carrying different marble frequencies. Even a few migrants per generation can overwhelm local selection, homogenizing populations.

Island populations diverge. Connected populations converge. Gene flow is evolution's great homogenizer, fighting against local adaptation.

**Genetic Drift**: The random force. In small populations, marble frequencies change by chance alone. Flip a coin 10 times - you might get 7 heads. Flip 10,000 times - you'll get close to 5,000.

Population genetics has a precise relationship: Variance in allele frequency = p(1-p)/2N

Where N is population size. Smaller populations, bigger random changes.

## The Power of Small Populations

Take 10 red marbles, 10 blue marbles. Each generation, randomly pick 20 to continue. By chance, you might pick 12 red, 8 blue. Do this repeatedly. Eventually, purely by chance, you'll have all red or all blue.

This is genetic drift in action. It's why small populations lose genetic diversity. Why endangered species face genetic problems beyond just low numbers.

The math is brutal: In a population of size N, genetic diversity halves every 1.4N generations. A population of 50 loses half its diversity in 70 generations. Evolution's creativity drains away.

## Effective Population Size

Not everyone reproduces. In elephant seals, one male might father 100 pups while others father none. The genetic population is smaller than the census population.

Effective population size (Ne) accounts for this:
- Unequal sex ratios reduce Ne
- Variable reproductive success reduces Ne  
- Non-random mating reduces Ne

Human Ne is about 10,000 - despite 8 billion of us. Most human genetic diversity traces to a small ancestral population. We're all cousins, more closely related than we appear.

## The Neutral Theory

In 1968, Motoo Kimura shocked evolutionary biology. Most genetic changes, he argued, are neutral - neither helpful nor harmful. They spread by drift, not selection.

The math supported him. Mutation rates, substitution rates, genetic diversity levels - all suggested most evolution happens without selection. Evolution isn't always about adaptation.

This started a war. Selectionists vs neutralists. Adaptationists vs drifters. The resolution: both are right. Much molecular evolution is neutral. Much phenotypic evolution is selected. Reality uses both.

## Mutation-Selection Balance

Harmful mutations constantly appear. Selection constantly removes them. The balance point depends on mutation rate and selection strength:

Equilibrium frequency = μ/s

Where μ is mutation rate, s is selection coefficient.

This explains genetic diseases. Why hasn't selection eliminated cystic fibrosis? Because new mutations replace removed ones. The disease persists at mutation-selection balance.

## The Coalescent

Run evolution backward. Take all gene copies in today's population. Trace their ancestry. Eventually, all lineages coalesce to a single ancestral gene.

The math predicts:
- Average coalescence time: 2N generations
- Variance is huge - some genes coalesce quickly, others slowly
- Pattern depends on population history

This backward view revolutionized population genetics. We can infer history from current genetic patterns. Detect ancient population bottlenecks. Date species splits. Track human migrations.

## Linkage and Recombination

Genes don't evolve independently. They're linked on chromosomes, inherited together. Selection on one gene affects its neighbors.

Recombination breaks linkages, shuffling gene combinations. Too little recombination and bad mutations accumulate near good genes. Too much and favorable combinations break apart.

Sex exists partly to manage this tradeoff. Recombination lets evolution work on genes individually rather than whole genomes. It's evolution's way of not putting all eggs in one basket.

## Population Structure

Real populations aren't well-mixed gene pools. They're structured - by geography, behavior, mate choice. Island models, stepping-stone models, isolation-by-distance - each creates different evolutionary dynamics.

Structure enables local adaptation but reduces overall diversity. It creates clines - gradual geographic changes in allele frequency. Human skin color, lactose tolerance, altitude adaptation - all show geographic structure.

## The Molecular Clock

In 1962, Emile Zuckerkandl and Linus Pauling noticed something odd: Proteins evolve at constant rates. Hemoglobin accumulates changes like a clock ticking.

If most changes are neutral, this makes sense. Neutral mutations appear at rate μ, fix by drift at rate 1/2N. The substitution rate = μ × 1/2N × 2N = μ.

The molecular clock lets us date evolutionary events. Humans and chimps: 6 million years. Humans and mice: 80 million years. All calculated from genetic differences.

## Quantitative Genetics

Most traits aren't single genes but many genes of small effect. Height, intelligence, disease risk - all polygenic. How do we study evolution of such traits?

The breeder's equation: R = h²S

Where R is response to selection, h² is heritability, S is selection differential.

This predicts how fast traits evolve. Why can we breed faster racehorses but not faster cheetahs? Horses had genetic variation for speed. Cheetahs, passing through a population bottleneck, don't.

## The Paradox of Variation

Population genetics revealed a paradox. Selection should eliminate variation, yet populations are full of it. How?

Multiple solutions:
- Mutation-selection balance maintains deleterious variation
- Frequency-dependent selection maintains multiple types
- Environmental heterogeneity maintains local variants
- Sexual selection maintains ornament variation
- Neutral variation persists by drift

Life found many ways to maintain its creative potential.

## From Genes to Phenotypes

The genotype-phenotype map isn't simple. One gene can affect many traits (pleiotropy). Many genes affect one trait (polygeny). The environment modulates everything.

This complexity buffers and channels evolution. Not all genetic changes cause phenotypic changes. Not all phenotypes are equally accessible. Evolution isn't free to wander anywhere in design space.

## Connections
→ [[L5_Speciation_Phylogenetics]] [[gene_flow]] [[genetic_drift]] [[molecular_evolution]]
← [[L3_Selection_Mechanisms]] [[population_genetics]] [[hardy_weinberg]]

---
*Population genetics transformed evolution from natural history to mathematical science. Now we don't just say evolution happens - we calculate how fast.*