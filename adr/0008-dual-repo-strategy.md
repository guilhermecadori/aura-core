# 0008 — Dual public/private repository strategy

**Status:** Accepted

**Date:** 2026-04-17

## Context

AURA has two conflicting goals:

1. **Build in public** — a visible open-source repository lets engineers read the code, ADRs, and commit history, learn from the architecture, and contribute
2. **Protect commercially valuable IP** — the curated Amarok corpus, tuned prompts, fine-tuning datasets, and real telemetry recordings represent months of domain work and would be trivial to fork wholesale

A fully open repository exposes the moat. A fully private repository forfeits the value of being publicly readable. A permissive license (MIT, Apache) on everything invites bad-faith commercial forks. A copyleft license alone doesn't prevent clones by entities that don't care about license obligations.

## Decision

Split the project into two repositories:

- **`aura-core`** — public, licensed under **AGPL-3.0**. Contains the generic architecture, all infrastructure code (Terraform, Helm, Docker), the Next.js frontend, the FastAPI backend skeleton, the LangGraph agent implementation, retrieval pipeline code, evaluation harness, synthetic/mock corpus samples, ADRs, documentation, and benchmarks.
- **`aura-pro`** — private. Contains the curated Amarok-specific corpus, tuned prompt templates, real telemetry recordings from the developer's vehicle, any fine-tuning datasets, and any prompt/retrieval configurations that represent commercially valuable tuning.

The public repo depends on the private repo's data through a clean interface (corpus is loaded from a configured S3 bucket path, prompts are loaded from configured templates). The public repo runs end-to-end against mock data; the private repo's contents are what make the production deployment actually good.

## Alternatives Considered

- **Single fully public repo, permissive license (MIT)** — rejected: invites wholesale commercial cloning with no protection
- **Single fully public repo, AGPL** — rejected: AGPL deters good-faith commercial forks but doesn't stop bad-faith ones; also exposes the curated corpus and prompts which are the actual moat
- **Single fully private repo** — rejected: forfeits the value of public code; engineers and contributors can't read it
- **Public repo with private Git submodule** — rejected: operationally clunky, depends on access control propagating correctly, exposes metadata about the private repo's existence and shape
- **Public repo with a paid/gated "pro" tier (dual licensing)** — rejected: implies commercial positioning the project doesn't yet have; over-complicates the story

## Consequences

### Positive

- Public repo is a clean artifact that anyone can read, clone, and run against mocks
- AGPL-3.0 provides reciprocity: commercial forks must open-source their modifications under the same terms
- Private repo protects the domain-specific work that represents the actual competitive advantage
- Clear architectural boundary between "the system" (public) and "the data that makes it good" (private)
- If the project is commercialized later, the licensing structure already supports it

### Negative

- Two repos to maintain, with a clean interface between them that must be actively preserved
- Some context-switching cost when a change spans both repos

### Neutral

- Eval sets may need to split: a public subset using mocks, a private authoritative set using real corpus
