# Qubic ANN Explorer: Make Useful Proof of Work Visible

## Proposal

Send **25.58 billion QUBIC** to
`JETSKIYUBUPFUCODLCBAJNFVCINABCOQIEXEZWNWMAFGBKPBOFTUCECCCSXG`
to launch Qubic ANN Explorer publicly and fund six months of operation and
active development:
live solution ingestion, Ant Colony integration, research metrics, and
continuous improvement of the product.

The project budget is **$11,000**. It includes the build of the explorer
itself, its public release, and six months of server costs and server
management starting from the funding date. The requested 25.58 billion QUBIC
is the direct conversion of this budget at the price snapshot of $0.00000043
per QUBIC (July 18, 2026). 

> Option 0: No
>
> Option 1: Yes, transfer 25.58 billion QUBIC

## What Is Qubic ANN Explorer?

Every week, Qubic miners submit millions of solutions. A solution is not an
abstract hash puzzle: each one trains a small artificial neural network (ANN)
under the exact scoring rules of that epoch Core. This is Qubic Useful
Proof of Work, and today it is essentially invisible. The evidence sits in
raw solution data, historical Core versions, scoring parameters, and
low-level C++ code that almost nobody can understand.

Qubic ANN Explorer makes that work visible. It is a public website that does
for mining solutions what a block explorer does for transactions:

```text
computor + mining seed + nonce + historical Core rules
    -> verified score
    -> reconstructed ANN
    -> visualization, replay, metrics, and research data
```

Anyone can search the archive, open a solution, look at the actual network it
produced, replay how that network evolved, and confirm that its score was
reproduced under the matching Core rules. Nothing has to be taken on trust:
everything shown is recomputed and checkable.

## Why Qubic Needs This

A conventional explorer can show that mining activity happened. It cannot
show what the mining computed. When someone asks "what does Qubic AI mining
actually produce?", the honest answer today is: read the scoring source code.

This proposal replaces that answer with a product. It gives investors,
researchers, and the wider community real, checkable proof of what Qubic
Useful Proof of Work produces, instead of abstract claims. Newcomers can see
the ANNs behind mining with their own eyes. Computors get permanent
recognition for their contribution. Developers and researchers get serious,
reproducible data to build on.

## What the Public Gets

- **Verifiable proof.** Every solution is connected to its epoch, computor,
  algorithm, score, threshold, and verification state. Historical results are
  reproduced with the matching historical Core rules.
- **Interactive ANN discovery.** Users can inspect input, evolution, and
  output neurons, synapses and their weights, structural changes, and sampled
  mutation replay, without reading C++ scoring code.
- **A living algorithm history.** The explorer covers every stage of Qubic
  mining evolution: Legacy ANN, HyperIdentity, Addition, the current Detour
  iteration, and the future Ant Colony algorithm.
- **Live mining visibility.** New solutions appear as pending and move to
  verified after deterministic processing. Completed epochs are rescanned to
  recover any missed solutions.
- **Computor recognition.** Computor and epoch pages provide a permanent
  record of participation and contributed solution genomes.
- **Developer access.** A documented public API and versioned research
  exports let others build analysis, education, and visualization tools on
  top of the archive.

The result is a product that makes Qubic AI work understandable to normal
users while preserving reproducible data for developers and researchers.

## Already Working

The explorer is already running. Open the preview and check everything
yourself: **<https://annexplorer.jetskipool.ai/>**

Every HyperIdentity and Addition solution has a fully reconstructed ANN.
Search by solution, Genome ID, epoch, or computor; interactive reconstruction
and mutation replay run directly in the browser, backed by educational
documentation and a versioned public API. The production archive passes exact
data-integrity checks and the complete browser test suite.

This proposal funds the public launch and continued development of a product
that is already built and operational.

## What the Funding Unlocks

Six months of active work:

1. Public launch: live ingestion, verification, and reconstruction running
   openly for everyone.
2. Continued verification and reconstruction of the historical archive.
3. Ant Colony integration and adaptation to new mining algorithms.
4. Research metrics: structural, behavioral, similarity, and novelty views,
   where each algorithm data supports them.
5. Public API improvements and versioned research exports.
6. Constant database, query, API, and visualizer optimization as the archive
   grows.
7. New features and research views driven by Core changes, archive
   discoveries, and community feedback.

## Ant Colony and Future Algorithms

Core is already moving toward its next algorithm: the Detour iteration
running on mainnet today is the announced stepping-stone to **Ant Colony**.
The explorer already handles Detour solutions, and this proposal includes
Ant Colony support. As the algorithm reaches production Core, the
explorer will integrate its parameters, verification path, visualization, and
research metrics. Ant Colony will be an explorable algorithm, not just a name
in a table.

The same architecture will be maintained for whatever comes after. Continuous
algorithm adaptation is part of the funded product, so the explorer evolves
with Qubic instead of freezing into a historical dashboard.

## Research Layer

Reconstructed networks make new kinds of analysis possible: how structures
evolve across epochs, how complexity relates to score, how algorithms differ,
and which solutions are genuinely unusual. The explorer will publish research
metrics in these areas step by step, for example graph structure (neurons,
synapses, density, connectivity), behavior (stability, convergence), and
novelty (genome and graph distance, outliers).

Not every metric applies to every algorithm. A metric is published only where
the algorithm reconstruction data supports a reproducible definition, and
each definition is documented next to the number. This creates a solid
technical foundation for future Aigarth-related research.

## Delivery Commitment

During the funded six months, the public will receive:

- A monitored public explorer with live and historical solutions
- Explicit pending and verified states backed by deterministic processing
- Ant Colony support after its production Core specification is available
- Published research metrics with documented definitions
- A stable, rate-limited developer API and research exports
- Direct dataset downloads in different formats, added soon after the public
  launch
- Continuous performance, visualization, and usability improvements

The explorer presents reproducible results: every graph and score shown is
recomputed from real solution data under the matching Core rules.

## Budget

| Item | Calculation | Total |
| --- | ---: | ---: |
| Explorer build, public release, and development | fixed | $8,000 |
| Production server | $250 x 6 months | $1,500 |
| Server management and algorithm adaptation | $250 x 6 months | $1,500 |
| **Total** |  | **$11,000** |

The development line covers the already-built explorer, its public release,
and the seven work items listed above. The server lines cover a dedicated
machine capable of running the verification and reconstruction workers next
to the public API, and six months of its management starting from the funding
date.

## Conclusion

Qubic ANN Explorer makes Qubic ANN mining output visible, verifiable, and
useful. It gives the public an interactive product, gives computors permanent
recognition, gives developers a stable data layer, and gives researchers
millions of reproducible ANN artifacts to study.

The core product and archive already exist, and the preview is open today.
This proposal funds the part that creates lasting public value: public
launch, live discovery, Ant Colony and future algorithm support, research
metrics, constant optimization, and six months of reliable operation.

---
<p align="center"><sub> LICENSED UNDER ANTI-MILITARY LICENSE</sub></p>
