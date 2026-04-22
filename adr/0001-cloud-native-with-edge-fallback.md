# 0001 — Cloud-native primary with edge fallback

**Status:** Accepted

**Date:** 2026-04-16

## Context

AURA's original framing was edge-first: a resource-constrained AI copilot running on a Raspberry Pi inside a single vehicle, targeting embedded and edge deployment scenarios for programmable-ECU products.

Reassessing the problem space, the broader AI engineering landscape is dominated by **scale-oriented systems**: multi-tenant services, LLM serving at throughput, RAG over large corpora, observability, cost-aware inference. Edge AI is a legitimate specialization but a narrow one, and an edge-only architecture leaves most of these disciplines unexercised.

The original edge framing also created product-design awkwardness: a single-vehicle, single-user device has unclear distribution, update, and monetization paths.

## Decision

AURA is architected as a **cloud-native multi-tenant service** with an **offline-capable edge runtime** as a feature, not the primary deployment target. The same codebase, prompts, corpus, and agent logic serve both paths. The cloud is the default; the edge runtime activates when connectivity is lost or the user prefers local operation.

## Alternatives Considered

- **Edge-only** (original plan) — rejected: narrow technical scope, awkward product story, harder to demo without physical hardware
- **Cloud-only** — rejected: loses the in-vehicle hardware demo, forfeits the "works offline in the tunnel" product benefit, discards existing Pi hardware investment
- **Two separate products (cloud app + edge device)** — rejected: doubles the maintenance surface, splits the brand and corpus, makes neither product strong

## Consequences

### Positive

- Exercises the scale-engineering disciplines (multi-tenant serving, streaming, observability, cost-aware inference) that an edge-only architecture cannot
- Produces a live public URL that any visitor can try without hardware
- Keeps the in-vehicle hardware video as a differentiating secondary artifact
- The "pull the cable, keeps working" moment is a stronger demo *because* there's a sophisticated cloud system it falls back from

### Negative

- Larger total scope than either cloud-only or edge-only
- Introduces consistency and sync concerns between cloud and edge runtimes
- Edge runtime work is deferred to Phase 8, late in the timeline

### Neutral

- Hardware investment (Pi 5) is preserved but deferred
- Repository structure must accommodate both deployment targets cleanly

## Revisit When

- Edge-specific customer demand appears and justifies reprioritization
- Cloud costs make a self-hosted-per-vehicle model more attractive
- A partner with edge hardware distribution emerges
