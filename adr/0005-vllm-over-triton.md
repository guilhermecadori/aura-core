# 0005 — vLLM over Triton for LLM serving

**Status:** Accepted
**Date:** 2026-04-17

## Context

The primary LLM serving path uses a self-hosted open-weights model. The goals:

- **High throughput** under concurrent load — continuous batching is the key technique
- **Low time-to-first-token** (TTFT) for responsive streaming
- **GPU-efficient** — portfolio budget, scale-to-zero when idle
- **OpenAI-compatible API** — simplifies client code and allows trivial swap to hosted APIs for the fallback path
- **Portfolio legibility** — the choice should be defensible and explainable

## Decision

Use **vLLM** as the inference server, deployed on **Modal** (GPU host with scale-to-zero). Qwen2.5-14B-Instruct as the primary model, with Llama-3.1-8B-Instruct as a benchmarked alternative (decision deferred to Phase 2).

## Alternatives Considered

- **NVIDIA Triton Inference Server** — rejected: excellent for heterogeneous model serving (CV, recommender, LLM together) in enterprise deployments with mixed hardware; for pure LLM serving, vLLM's specialized optimizations (PagedAttention, continuous batching) produce better throughput with less operational complexity. Triton's use case isn't AURA's use case.
- **TGI (Text Generation Inference by Hugging Face)** — rejected: viable alternative; comparable performance; lost on ecosystem momentum and vLLM's stronger continuous batching implementation at time of decision
- **SGLang** — rejected: promising, potentially faster for structured workloads; ecosystem and community still smaller; revisit later
- **Ollama** — rejected: excellent for local development; not designed for multi-tenant serving with batching
- **llama.cpp server** — rejected for the cloud path: CPU-focused, not competitive with GPU-backed vLLM on throughput. *Retained for edge path* (Phase 8) where CPU/quantized inference is the right fit.
- **Hosted API only (Anthropic, OpenAI, Together, Groq)** — rejected as the *primary* path: forfeits the demonstration of "I can serve LLMs efficiently." *Retained as the premium-tier fallback path* for hard queries and demo reliability.

## Consequences

### Positive

- Continuous batching produces strong throughput without manual request batching
- PagedAttention enables efficient KV cache reuse
- OpenAI-compatible API means clients are portable between vLLM and hosted APIs
- Modal provides scale-to-zero economics ($0 idle)
- Strong community, frequent releases, broad model support

### Negative

- vLLM version churn is real; pinning matters
- Some newer models lag on vLLM support at release time
- GPU cold-start latency on Modal scale-up (~30–60s) affects first-request latency
- CUDA-specific; no easy path to other accelerators without work

### Neutral

- Model choice (Qwen vs Llama) is independent of server choice
- Hosted API fallback provides resilience and handles cold-start edge cases

## Revisit When

- SGLang or a successor demonstrates materially better throughput on the target workload
- A role specifically requires Triton experience and the gap matters
- Modal's economics shift such that dedicated GPU hosting becomes competitive
- A non-NVIDIA accelerator becomes attractive for cost reasons
