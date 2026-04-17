# AURA

**Automotive Understanding & Reasoning Agent** — an AI copilot for vehicle diagnostics.

Connect your vehicle, ask a question in natural language, get a grounded answer backed by live telemetry and a curated technical corpus. Cloud-native with offline edge fallback.

> **Status:** In active development. First public release expected end of Phase 3. See [Roadmap](#roadmap).

---

## What It Does

AURA combines three things most vehicle diagnostic tools keep separate:

- **Live telemetry** — OBD-II sensor streams with visual gauges and a scrubbable timeline
- **Technical knowledge** — retrieval-augmented reasoning over service manuals, TSBs, DTC databases, and community knowledge
- **Conversational interface** — voice or text, bilingual (PT-BR / EN), with visible agent reasoning

Point at a gauge, ask what you're seeing. Scrub back on the timeline and ask what happened. Get a diagnostic code explained in context of *your* vehicle's current state, not a generic lookup.

v1 targets the **2021+ Volkswagen Amarok V6 Extreme** (EA897 3.0 TDI). The architecture is designed to scale to more vehicles; v1 goes deep on one.

---

## Architecture at a Glance

```
Browser (Next.js PWA)
    │
    ├─ Chat + telemetry UI, token streaming, voice I/O
    ▼
FastAPI backend ── LangGraph ReAct agent
    │                   │
    │                   ├─→ Qdrant (hybrid retrieval: BM25 + bge-m3, reranked)
    │                   ├─→ vLLM on Modal (Qwen2.5-14B, scale-to-zero GPU)
    │                   └─→ Anthropic API (premium fallback)
    │
    ├─→ Postgres (users, history) · Redis (cache, hot buffer)
    │
    └─→ Redpanda (Kafka) ─┬─→ Hot consumer → Redis
                          └─→ Archive consumer → S3 (Parquet)
                                                   │
                                                   ▼
                                        Databricks / PySpark (analytics)
                                        DuckDB (ad-hoc)
                                        Airflow (orchestration)

Observability: OpenTelemetry → Grafana Cloud · Langfuse · MLflow
Infra: Terraform · Docker · Helm · AWS EKS (ingest pipeline)
```

Full architecture document and ADRs in [`/docs`](./docs).

---

## Stack

| Layer | Choice |
|---|---|
| Frontend | Next.js 15, TypeScript, Tailwind, shadcn/ui, Vercel AI SDK |
| Backend | FastAPI, Python 3.12, async throughout |
| Agent | LangGraph (ReAct loop) |
| LLM serving | vLLM on Modal · Anthropic API (fallback) |
| Retrieval | Qdrant · bge-m3 · bge-reranker-v2-m3 · BM25 hybrid |
| Data | Postgres (Neon) · Redis (Upstash) · S3 |
| Streaming | Redpanda (Kafka-compatible) |
| Analytics | Databricks (PySpark) · DuckDB |
| Orchestration | Airflow |
| Observability | OpenTelemetry · Grafana Cloud · Langfuse · MLflow |
| Infra | Docker · Terraform · AWS EKS · Helm · KEDA |

Notable exclusions with rationale in [`docs/adr/`](./docs/adr): LangChain, Pinecone, Triton.

---

## Roadmap

Phases are sequential. Each has an explicit stop criterion. Target cadence is part-time development; calendar estimates are deliberately conservative.

- [ ] **Phase 0 — Foundation** *(3 mo)*
      Background study: async Python, streaming UI, Kafka, vector DBs, k8s literacy, AWS/Terraform, OTel.
- [ ] **Phase 1 — RAG Foundation** *(2 mo)*
      Corpus ingestion, Qdrant hybrid retrieval, eval harness (~200 golden questions), MLflow tracking.
- [ ] **Phase 2 — LLM Serving + ReAct Agent** *(2 mo)*
      vLLM on Modal, LangGraph ReAct loop, tool use, semantic cache, Langfuse traces.
- [ ] **Phase 3 — Product Surface** *(2.5 mo)*
      Next.js PWA, split telemetry/chat UI, auth, multi-tenancy, demo mode. **Public URL goes live.**
- [ ] **Phase 4 — Streaming Data Pipeline** *(1.5 mo)*
      Redpanda, `aura-bridge` OBD-II client, hot + archive consumers, load simulator.
- [ ] **Phase 5 — Observability & Production Ops** *(2 mo)*
      OTel end-to-end, public Grafana dashboard, k6 load suite, EKS deployment with KEDA, benchmark report.
      → **Primary checkpoint.** Main development phase complete here.
- [ ] **Phase 6 — Analytics & Training Data** *(1 mo)* — Databricks notebooks, DuckDB comparisons.
- [ ] **Phase 7 — Voice** *(1.5 mo)* — Whisper + Piper, hands-free mode, bilingual.
- [ ] **Phase 8 — Edge Runtime** *(2 mo)* — Raspberry Pi 5 offline mode, cloud sync, kiosk boot.
- [ ] **Phase 9 — Video** *(0.75 mo)* — In-vehicle demo, ReAct HUD, offline moment.

---

## Repository Layout

This repo (`aura-core`) contains the open-source architecture, infrastructure code, evaluation harnesses, and mock corpus samples — everything needed to understand, reproduce, and learn from the system.

The curated vehicle-specific corpus, tuned prompts, and real telemetry recordings live in a separate private repo (`aura-pro`) to keep sensitive IP out of the public fork path.

```
aura-core/
├── apps/
│   ├── web/              Next.js PWA
│   ├── api/              FastAPI backend
│   ├── bridge/           OBD-II client
│   └── edge/             Pi runtime (Phase 8)
├── packages/
│   ├── agent/            LangGraph ReAct implementation
│   ├── rag/              Retrieval pipeline
│   └── telemetry/        Shared schemas and parsers
├── infra/
│   ├── terraform/        AWS (EKS, S3, IAM)
│   ├── helm/             k8s charts for ingest pipeline
│   └── docker/           Compose for local dev
├── pipelines/
│   ├── airflow/          DAGs (corpus refresh, eval, analytics)
│   └── databricks/       PySpark notebooks
├── evals/
│   ├── golden/           Reference Q&A set
│   └── promptfoo/        Eval configurations
└── docs/
    ├── architecture.md   Full design document
    ├── adr/              Architecture Decision Records
    └── benchmarks.md     Performance report (Phase 5)
```

---

## Try It

- **Live demo** *(Phase 3+)*: link goes here when the product ships
- **Benchmark dashboard** *(Phase 5+)*: public Grafana link
- **Video demo** *(Phase 9)*: in-vehicle walkthrough

Until then, the architecture document and ADRs are the thing to read.

---

## Why This Exists

AURA is a project demonstrating end-to-end AI engineering: retrieval systems, agent design, LLM serving, streaming data, multi-tenant product engineering, observability, and cost-aware production ops. It is built in public.

The vehicle specificity is deliberate — deep domain grounding is the moat that distinguishes this from the dozens of generic RAG demos. The developer drives the target vehicle daily, which makes ground truth and demo material first-hand rather than fabricated.

---

## License

`aura-core` is released under **AGPL-3.0**. You're welcome to read, learn from, fork for non-commercial use, and contribute. Commercial use requires reciprocal open-sourcing under the same license.

---

## Acknowledgments

Built on the work of many open-source projects — vLLM, Qdrant, LangGraph, BGE models, Redpanda, Airflow, OpenTelemetry, and the broader Python and TypeScript ecosystems. Also on the knowledge shared in the Brazilian tuned vehicles owner community.
