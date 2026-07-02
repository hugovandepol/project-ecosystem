# EnerGNN

**Last Updated:** 2026-07-01

## Table of Contents

- [Basic Info](#basic-info)
- [Description](#description)
- [Overview](#overview)
- [Technical Profile](#technical-profile)
- [Grid Context](#grid-context)
- [Related Projects](#related-projects)
- [Maturity & Adoption](#maturity--adoption)
- [Learn More](#learn-more)
- [Additional Notes](#additional-notes)

## Basic Info

- LF Energy webpage: TODO: not yet set up (project onboarding as of July 2026)
- Website:
- Code: https://github.com/energnn
- Documentation: https://energnn.readthedocs.io/en/stable/
- Calendar:
- LinkedIn:
- Community:
	- Mailing List:
	- Slack:
- LFX Insights:
- Other:

## Description

A Graph Neural Network library for real-life energy networks.

## Overview

EnerGNN is a Python library, built on JAX and Flax, for applying graph neural networks (GNNs) to full-scale energy networks. Its central idea is a data representation called the Hyper Heterogeneous Multi Graph (H2MG), which captures the way a real transmission grid is actually wired: connections that join more than two components at once (hyper), many different component types such as lines, transformers, generators, and loads (heterogeneous), and multiple components sharing the same location (multi). General-purpose GNN libraries assume simpler graphs, so EnerGNN exists to give power system engineers a modeling toolkit that matches the structure of their networks rather than forcing the network into a simplified form.

The problem EnerGNN targets is the growing gap between the number of operational studies a transmission operator needs to run and the time that conventional numerical solvers take to run them. Many grid problems — screening thousands of contingencies, choosing voltage setpoints, deciding switch positions — are optimization problems that must be re-solved for every new operating condition. EnerGNN uses "amortized optimization": rather than solving each case from scratch, a GNN is trained once on many cases so that, for any new operating condition, it produces a good solution almost instantly. Because the models are built on the H2MG representation, they remain valid when the grid's structure varies — line outages, network expansion, or the renaming and reordering of equipment that happens continuously in a real control environment.

EnerGNN was developed at RTE, the French transmission system operator, in collaboration with academic partners led by the University of Liège. It is used internally at RTE across several use cases on full-scale French transmission data (7,000+ buses), including contingency screening, tertiary voltage control, and substation topology optimization. The library is organized so that grid engineers describe their business problem once (as a "problem" object) and reuse the shared GNN modeling and training machinery, keeping domain logic separate from the neural network internals. Transmission network data can be imported directly from PyPowSyBl.

## Technical Profile

### What It Does

Provides a graph neural network modeling library — data representation, model architectures, a problem interface, and a training loop — for building GNN models that quickly approximate the solution to power system optimization and analysis problems on full-scale, structurally varying networks.

### Problem(s) Solved

Conventional solvers (e.g., MILP, AC power flow) produce robust answers but can be too slow when an operator must evaluate large numbers of operating conditions or needs a decision in near real time. EnerGNN lets a transmission operator train a fast surrogate model that returns a good solution in milliseconds and — unlike off-the-shelf GNN tooling — stays valid as the grid's topology changes with outages, expansions, and equipment relabeling. It also gives grid researchers and operators a shared, reusable foundation for applying GNNs to their own problems instead of rebuilding data-handling and training infrastructure for each one.

### Key Capabilities

- Hyper Heterogeneous Multi Graph (H2MG) data representation (`energnn.graph`) that models hyper-edges, multiple component types, and collocated components, with serialization, padding, and statistical utilities
- Modular, structure-robust GNN models (`energnn.model`) assembled from interchangeable `flax.nnx` building blocks — Normalizer, Encoder, Coupler (message passing, neural ODE, etc.), and Decoder
- A problem interface (`energnn.problem`) that decouples use-case business logic (data sampling, objective and gradient definition) from the GNN forward/backward pass, so each use case implements its own problem and loader against shared model machinery
- Support for supervised and self-supervised (label-free) training, enabling amortized optimization for problems where no solved-example labels exist
- A training framework (`energnn.trainer`) with an optax-based training loop, experiment-tracker integration, and orbax checkpointing
- Direct import of transmission grid data from PyPowSyBl, with GPU-accelerated execution via JAX

### Relevant Standards

None. EnerGNN is a graph neural network modeling library and does not directly implement grid communication or data model standards. It can ingest transmission network data through PyPowSyBl, which handles grid data formats, but EnerGNN itself does not implement those formats.

## Grid Context

### Grid Segment

Transmission

### Function

Planning & Analysis

### Industry Solution Categories

#### Solution Type

- Power System GNN Library: A graph neural network modeling and training framework that produces fast surrogate models for power system optimization and analysis problems on full-scale, structurally varying networks.

#### Component of

None. EnerGNN is a standalone modeling library. Models built with it could be embedded within an EMS (for security analysis or voltage control) or offline study tooling, but the library itself is not a component of those systems.

### Cross-Cutting Tags

- **Project Intent:** Applied
- **AI/ML:** Yes
- **Deliverable Type:** Software

## Related Projects

- **PowSyBl**: Integration — EnerGNN imports transmission network data directly from PyPowSyBl (PowSyBl's Python interface), which serves as the grid-data source that populates EnerGNN's H2MG representation.
- **OpenGridFM**: Similar approach, different scope — both are open source toolkits for building GNN models of power grids. OpenGridFM focuses on pre-training and fine-tuning foundation models for power system *analysis* tasks; EnerGNN is a general-purpose GNN library emphasizing the H2MG representation and amortized optimization across analysis *and* control use cases.
- **AINETUS**: Similar methods, different layer — both apply GNNs to transmission grid problems and originate in RTE-led research. AINETUS is an operator decision-support stack whose agent loop includes a graph neural power flow solver; EnerGNN is a general-purpose GNN modeling library of the kind used to build such models. No asserted direct integration.

## Maturity & Adoption

### LF Energy Stage

Sandbox

### Deployment Maturity

R&D

<!-- RTE use cases are at TRL 3–5 on full-scale real data — validated in a relevant environment, not yet running in production control-room operations. -->

### Supporting / Adopting Organizations

- RTE (transmission system operator, France — project lead)
- Université de Liège / ULiège (university — academic lead)
- INRIA (research institute, France)

Academic partnerships also span Université Paris-Saclay, Mines Paris-PSL, and University College Dublin, with an upcoming collaboration with InstaDeep (as stated in the April 2026 TAC presentation).

## Learn More

- [EnerGNN TAC proposal (lf-energy/tac#753)](https://github.com/lf-energy/tac/issues/753)
	- Date: 2026-02-13
	- Type: TAC Proposal
- [EnerGNN – A Graph Neural Network library for real-life complex Energy systems (LF Energy TAC presentation)](https://tac.lfenergy.org/meetings/2026/2026-04-14/EnerGNN_LFE_TAC.pdf)
	- Date: 2026-04-14
	- Type: Presentation
- [EnerGNN documentation](https://energnn.readthedocs.io/en/stable/)
	- Type: Documentation

## Additional Notes

**Function: a modeling library, not an operational system.** EnerGNN's RTE use cases include operational-sounding tasks (tertiary voltage control, substation topology optimization), which can make it look like an Operations project. It is not. EnerGNN is a GNN modeling library — its activity content is building surrogate models that approximate the solution to optimization and analysis problems, not operating the grid. The use cases are R&D demonstrations (TRL 3–5) of what the library can build, and their outputs (risky-contingency lists, recommended setpoints, switch actions) feed operational decisions rather than executing them. By the taxonomy's activity-content principle — and consistent with the other GNN and modeling libraries in the portfolio (OpenGridFM, Power Grid Model, PowSyBl) — it belongs in Planning & Analysis. This mirrors the OpenSTEF precedent: a tool whose output feeds operations is still P&A when its activity content is analytical.

**Speed-vs-optimality trade-off.** EnerGNN's value proposition is speed, not beating conventional solvers on solution quality. In RTE's topology-optimization study reported in the April 2026 TAC presentation, the GNN reached ~9.3% mean inter-region capacity improvement in ~200 ms, versus ~13.7% for a MILP baseline that took ~10 minutes — a lower-quality answer delivered orders of magnitude faster, which is the point for near-real-time and large-batch use.

**General-purpose library, transmission use today.** EnerGNN is described as a library for energy networks and large complex industrial infrastructures generally, and the H2MG representation is not transmission-specific. It is classified as Transmission because every current use case comes from RTE's transmission operations. The methods could in principle extend to distribution or other networked infrastructure if such use cases joined the project.
