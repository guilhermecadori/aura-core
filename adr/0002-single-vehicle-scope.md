# 0002 — Single-vehicle scope for v1

**Status:** Accepted
**Date:** 2026-04

## Context

A diagnostic AI copilot could in principle serve any vehicle. Broader coverage appears more commercially valuable and demonstrates more general architecture. However, corpus quality, evaluation coverage, and demo credibility all scale inversely with breadth — shallow multi-vehicle coverage is commodity (many existing OBD-II apps do this poorly), while deep single-vehicle expertise is both differentiating and verifiable.

The developer owns a 2021+ VW Amarok V6 Extreme (EA897 3.0 TDI, stock OEM ECU). This vehicle offers high diagnostic complexity (variable geometry turbo, common-rail injection, DPF with active regeneration, EGR, AdBlue), an active Brazilian owner community (good corpus sources), and — most importantly — first-hand ground truth.

## Decision

v1 supports **only the 2021+ VW Amarok V6 Extreme**. The corpus, evaluation set, telemetry parsing, and tuning decisions are all specific to this vehicle. The architecture is designed to generalize (corpus partitioning by vehicle, per-vehicle eval sets, tenant-scoped telemetry) but v1 ships with one vehicle fully supported.

## Alternatives Considered

- **Top-10 Brazilian vehicles** — rejected: no ground truth for vehicles the developer doesn't own, corpus quality would be forum-scraped and shallow, evals would be impossible to author credibly, demo video loses its emotional weight
- **VW platform broadly (Amarok / Tiguan / T-Cross)** — rejected: EA897 engine family is the deep-knowledge area; branching into other powertrains dilutes expertise with limited added coverage
- **Any OBD-II vehicle generically** — rejected: becomes a DTC lookup tool with a chat wrapper, no moat

## Consequences

### Positive

- Corpus can be curated to expert depth; grounding and evals are credible
- Demo video features real vehicle with real faults; emotionally and technically stronger than any generic demo
- Scope discipline keeps Phase 1 (RAG) tractable and high-quality
- "I built the best possible AI copilot for this one vehicle" is a stronger portfolio story than "I built a generic tool"

### Negative

- Limits the addressable user base for the live product — most reviewers who try it do so via demo mode rather than with their own vehicle
- Reviewers without domain knowledge may undervalue the depth of expertise
- If the project is ever commercialized, multi-vehicle expansion is non-trivial work

### Neutral

- Demo mode (recorded drives from the developer's vehicle) is mandatory infrastructure for reviewer access
- Architecture must still be designed multi-tenant and multi-vehicle-ready even if only one vehicle is populated

## Revisit When

- v1 ships and real user feedback justifies specific next-vehicle priorities
- A partnership or dataset emerges that provides ground truth for another vehicle cheaply
- Commercial traction requires breadth for market reasons
