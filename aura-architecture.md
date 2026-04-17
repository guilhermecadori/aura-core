# AURA — Automotive Understanding & Reasoning Agent

**A cloud-native AI copilot for vehicle diagnostics, with offline edge fallback.**

---

## 1. Project Concept

### 1.1 What AURA Is

AURA is an AI copilot for vehicle diagnostics. A user connects their vehicle (via OBD-II dongle, log upload, or demo mode) and has a natural-language conversation about its behavior — diagnostic trouble codes, telemetry anomalies, maintenance questions, troubleshooting walkthroughs. The system reasons over live telemetry, a curated technical corpus, and the vehicle's history to produce grounded, cited answers.

The product is intentionally narrow in scope: **deep expertise for one specific vehicle — the 2021+ Volkswagen Amarok V6 Extreme (EA897 3.0 TDI, OEM ECU)**. This narrowness is the moat. Shallow multi-vehicle coverage is commodity; deep single-vehicle expertise that a real owner can validate is differentiating.

The architecture is designed to scale to many vehicles later, but the v1 product knows one vehicle exceptionally well.

### 1.2 Why This Project

This is a portfolio project targeting AI Engineering, ML Platform, and Applied AI roles. It was originally inspired by FuelTech's job descriptions but is no longer narrowly aimed at FuelTech specifically. The compensation reality of the broader AI engineering market — where scale-oriented work dominates demand — drove a deliberate pivot from edge-only to cloud-native-with-edge-fallback.

The project is designed to demonstrate, in priority order:

1. **AI systems at scale** — RAG, LLM serving, multi-tenancy, observability, cost-aware inference
2. **Agent design** — ReAct loop, tool use, grounded reasoning, controlled failure modes
3. **Data and platform engineering** — streaming pipelines, batch analytics, orchestration, IaC
4. **Edge AI competence** — quantized inference, offline-first design, hardware integration
5. **Product engineering** — a real product a stranger can use, not a notebook demo

### 1.3 Why It Has To Be a Real Product

A live URL beats a benchmark report. A video of a real vehicle beats a slide deck. The portfolio strategy is built around two artifacts a reviewer can engage with directly:

- **A live web product** at a public URL where any visitor can try AURA in demo mode within seconds
- **A video** demonstrating in-vehicle voice interaction, ReAct reasoning, offline fallback, and bilingual operation

Everything else — the architecture decisions, the benchmark dashboard, the open-source repository, the ADRs — supports these two artifacts.

### 1.4 The Vehicle

The 2021+ VW Amarok V6 Extreme is the development and demo vehicle, owned by the developer. Specifications:

- **Engine**: EA897 evo2, 3.0L V6 TDI, 258 hp / 580 Nm
- **Drivetrain**: 4MOTION permanent AWD, 8-speed automatic
- **ECU**: Stock OEM (Bosch EDC17), no aftermarket tuning
- **Diagnostic interface**: Standard OBD-II, plus VW-specific UDS PIDs accessible via ELM327-class dongles for enhanced telemetry

The choice is deliberate: the EA897 is genuinely complex (variable geometry turbo, common-rail injection, DPF with active regeneration, EGR, AdBlue), the Brazilian Amarok owner community is active (good RAG corpus sourcing), and the developer's ownership means real ground-truth data and credible demo material.

### 1.5 Naming

- **AURA** — the product. Selected for sound and feel (Siri/Alexa/NeMo tier), with a defensible technical backronym (Automotive Understanding & Reasoning Agent — "Reasoning" validated against actual ReAct agent loop usage)
- **AURA Core** — the inference and orchestration backend
- **AURA Sense** — the telemetry ingestion subsystem
- **AURA RAG** — the retrieval and corpus subsystem
- **AURA Edge** — the offline-capable runtime for in-vehicle deployment
- **AURA Voice** — the STT/TTS subsystem

### 1.6 Repository Strategy

**`aura-core`** — public, AGPL-3.0. Generic architecture, mock data, synthetic corpus samples, all infrastructure code, all eval harnesses, the full open-source product surface. This is what reviewers see.

**`aura-pro`** — private. Curated Amarok corpus, tuned prompts, real telemetry recordings, fine-tuning datasets, any commercially valuable IP. AGPL on `aura-core` prevents bad-faith forks while allowing recruiters and engineers to read and learn from the architecture.

### 1.7 Language Strategy

- **Internal reasoning**: English (better LLM performance, larger evaluation literature)
- **User-facing primary**: Brazilian Portuguese (target market, developer's native language, less competition)
- **User-facing secondary**: English (larger global reviewer audience)
- **Deferred to v2**: Spanish (Latin American market expansion)

The system detects user input language and responds in kind. The corpus is bilingual where source documents permit.

---

## 2. Product Design

### 2.1 Primary User Flow

A user arrives at the URL, signs in, and lands on their vehicle dashboard. First-time users go through an onboarding flow with three options:

1. **Connect via OBD-II** — install the `aura-bridge` desktop client, pair an ELM327 dongle, get live telemetry
2. **Upload a log** — drop a saved diagnostic log file for post-hoc analysis
3. **Demo mode** — skip vehicle connection, use a recorded drive from the developer's Amarok

The third option is critical: it lets any reviewer experience the full product in 10 seconds without owning a vehicle.

### 2.2 Main Interface

The primary screen is a **split layout**: telemetry panel on the left, conversation panel on the right. Gauges and chat are not separate features — they are the same feature, tightly coupled.

#### 2.2.1 Telemetry Panel (Left)

Live gauges sized by importance:

- **Primary cluster (always visible)**: RPM, boost pressure, EGT, coolant temperature
- **Secondary cluster (expandable)**: oil temperature, DPF soot load, DPF regeneration status, fuel rail pressure, injector balance rates (IBR per cylinder), turbo vane position, MAF, MAP, intake air temperature, battery voltage, AdBlue level, EGR position

Interaction model:

- **Click a gauge** → adds a context chip to the chat input (e.g., *"EGT: 743°C"*)
- **Drag a gauge to chat** → same effect, more discoverable
- **Timeline scrubber** below gauges → 30-minute rolling buffer; scrub a range and ask about it
- **DTC badges** appear as a row when codes are present; clicking pre-fills *"Explain P0299"* in the chat

#### 2.2.2 Conversation Panel (Right)

- **Token-streamed responses** (non-negotiable for perceived latency)
- **ReAct trace** visible but collapsed by default; "show reasoning" toggle expands inline to reveal Thought → Action → Observation steps
- **Context chips** at top of input show what's attached to the next message (vehicle, current telemetry snapshot, scrubbed range, DTC); user can remove individually
- **Voice input**: hold-to-talk microphone button; transcription appears in input box for review before send
- **Voice output**: per-message speaker icon (no auto-play); hands-free mode toggle for continuous voice-to-voice
- **Message actions**: copy, share (public read-only conversation link), add to vehicle notes

### 2.3 Secondary Surfaces

- **Vehicle history** — chronological log of conversations, observed DTCs, maintenance events, flagged anomalies
- **Maintenance view** — service intervals, upcoming items by mileage and date, items flagged from past conversations
- **Settings** — language (PT-BR / EN), units (metric default), connected devices, data export

### 2.4 Mobile

PWA (Progressive Web App). Responsive design, service worker for offline, installable to home screen, microphone access via WebRTC. Works on iOS and Android without app store approval. No native apps in v1.

### 2.5 Explicitly Out of Scope

- Multi-vehicle support (by design, scope discipline)
- Social features, gamification, marketplaces
- ECU tuning or programming (different domain, no hardware)
- Native mobile apps
- Workshop/fleet operator features

---

## 3. Technical Architecture

### 3.1 Architectural Principles

- **Async by default** — every I/O call non-blocking; one FastAPI worker handles many concurrent streams
- **Streaming over polling** — WebSockets for telemetry, SSE for LLM responses
- **Observable everything** — every request gets a trace ID, every LLM call is logged with full context
- **Right tool for the job** — Modal for GPU, Railway for stateless services, EKS only where it earns its keep
- **Eval before deploy** — no prompt or model change reaches production without passing the eval suite

### 3.2 System Topology

```
┌──────────────────────────────────────────────────────────────────┐
│ PRESENTATION                                                     │
│ Next.js 15 PWA on Vercel (Edge Network)                          │
└─────────────────────────────┬────────────────────────────────────┘
                              │ HTTPS / WSS
┌─────────────────────────────▼────────────────────────────────────┐
│ APPLICATION (Railway / Fly.io — region: GRU São Paulo)           │
│ ┌──────────────────┐ ┌──────────────────┐ ┌───────────────────┐  │
│ │ FastAPI Chat API │ │ Telemetry Ingest │ │ ReAct Orchestrator│  │
│ │ Streaming SSE    │ │ WS → Kafka       │ │ LangGraph         │  │
│ └────────┬─────────┘ └────────┬─────────┘ └─────────┬─────────┘  │
└──────────┼────────────────────┼─────────────────────┼────────────┘
           │                    │                     │
           │           ┌────────▼──────────┐          │
           │           │ Redpanda (Kafka)  │          │
           │           │ Topics:           │          │
           │           │  telemetry.raw    │          │
           │           │  telemetry.events │          │
           │           │  dtc.observed     │          │
           │           └─┬──────────────┬──┘          │
           │             │              │             │
           │     ┌───────▼─────┐ ┌──────▼─────────┐   │
           │     │Hot consumer │ │Archive consumer│   │
           │     │→ Redis      │ │→ S3 (Parquet)  │   │
           │     └─────────────┘ └────────┬───────┘   │
           │                              │           │
┌──────────▼──────────────────────────────▼───────────▼────────────┐
│ DATA                                                             │
│ ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────────┐    │
│ │ Postgres │  │ Upstash  │  │ Qdrant   │  │ AWS S3          │    │
│ │ (Neon)   │  │ Redis    │  │ Cloud    │  │ telemetry       │    │
│ │ users,   │  │ cache,   │  │ vectors  │  │ archive,        │    │
│ │ history  │  │ buffer   │  │          │  │ corpus sources  │    │
│ └──────────┘  └──────────┘  └──────────┘  └─────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
           │                                          │
┌──────────▼─────────────────┐  ┌──────────────────── ▼────────────┐
│ INFERENCE                  │  │ ANALYTICS & ORCHESTRATION         │
│ ┌────────────────────────┐ │  │ ┌──────────────────────────────┐  │
│ │ Modal                  │ │  │ │ Airflow (self-hosted)        │  │
│ │ vLLM + Qwen2.5-14B     │ │  │ │ DAGs:                        │  │
│ │ (primary, scale-to-0)  │ │  │ │  - corpus refresh            │  │
│ └────────────────────────┘ │  │ │  - eval suite                │  │
│ ┌────────────────────────┐ │  │ │  - analytics trigger         │  │
│ │ Anthropic API          │ │  │ └────────────┬─────────────────┘  │
│ │ (premium fallback)     │ │  │              │                    │
│ └────────────────────────┘ │  │ ┌────────────▼─────────────────┐  │
└────────────────────────────┘  │ │ Databricks Community Edition │  │
                                │ │ PySpark notebooks:           │  │
                                │ │  - fleet analytics           │  │
                                │ │  - training data generation  │  │
                                │ └────────────┬─────────────────┘  │
                                │              │                    │
                                │ ┌────────────▼─────────────────┐  │
                                │ │ DuckDB (ad-hoc analytics)    │  │
                                │ └──────────────────────────────┘  │
                                └───────────────────────────────────┘

EKS (AWS): Redpanda + telemetry consumers, KEDA autoscaling on lag
Observability: OpenTelemetry → Grafana Cloud + Langfuse + MLflow
IaC: Terraform for AWS portions
Secondary cloud: GCP Cloud Run (aura-bridge simulator)
```

### 3.3 Component Choices and Rationale

#### 3.3.1 Frontend

| Choice | Rationale |
|---|---|
| **Next.js 15 + TypeScript** | Largest ecosystem, best AI streaming support, strong hiring signal |
| **Tailwind + shadcn/ui** | Fast, professional, doesn't look AI-generated |
| **Vercel AI SDK** | Industry standard for streaming chat, tool calls, structured outputs |
| **TanStack Query + Zustand** | Server state + client state without Redux overhead |
| **Recharts** (gauges, scrubber) | Faster to build than visx; swap if specific gauges need more control |
| **Vercel** (deployment) | Free tier sufficient initially; portable to any Node host |

#### 3.3.2 Backend

| Choice | Rationale |
|---|---|
| **FastAPI + Python 3.12** | ML ecosystem requires Python; FastAPI is the standard async web framework |
| **Pydantic v2** | Schema validation, structured outputs |
| **Async everywhere** | Non-negotiable for concurrent streaming |
| **Railway / Fly.io** (deployment) | Cheaper and simpler than k8s for stateless services; São Paulo region available |

#### 3.3.3 LLM Serving

| Choice | Rationale |
|---|---|
| **vLLM** (inference server) | Continuous batching, PagedAttention, OpenAI-compatible API; the correct choice for LLM serving |
| **Qwen2.5-14B-Instruct** (primary) | Strong PT-BR capability, good instruction following, permissive license |
| **Llama-3.1-8B-Instruct** (alternative) | Benchmarked against Qwen during Phase 2; whichever wins on the eval set ships |
| **Modal** (GPU host) | Scales to zero (no idle cost), excellent vLLM support, deploys from Python |
| **Anthropic API** (premium fallback) | Smarter routing layer for hard queries; demonstrates routing/fallback design |

#### 3.3.4 RAG Stack

| Choice | Rationale |
|---|---|
| **Qdrant** (vector DB) | Rust-based performance, hybrid search support, metadata filtering, self-hostable |
| **bge-m3** (embeddings) | Multilingual including PT-BR, strong performance, runs cheaply |
| **bge-reranker-v2-m3** | Cross-encoder reranker, retrieve-then-rerank is industry standard |
| **Hybrid retrieval (BM25 + dense)** | BM25 catches exact-match cases (DTCs, part numbers) dense misses |
| **Semantic + structural chunking** | Parse by document structure first, semantic within sections |

#### 3.3.5 Agent Layer

| Choice | Rationale |
|---|---|
| **LangGraph** | Explicit state, explicit edges; correct abstraction for ReAct loop with tools |
| **NOT LangChain** | Leaky abstractions; senior reviewers see avoidance as a positive signal |
| **Tools**: `retrieve_corpus`, `get_telemetry_snapshot`, `explain_dtc`, `search_history`, `compute_metric` | Minimum viable tool set; expand only when prompted by real failures |

#### 3.3.6 Data Layer

| Choice | Rationale |
|---|---|
| **Postgres (Neon)** | Users, vehicles, conversations, maintenance logs, DTCs; serverless, cheap |
| **Redis (Upstash)** | Semantic cache, session state, rate limiting, hot telemetry buffer |
| **Auth: Supabase Auth** | Free, integrates with Postgres, OAuth providers |
| **AWS S3** | Telemetry archive (Parquet), corpus source documents, model artifacts |

#### 3.3.7 Streaming and Telemetry

| Choice | Rationale |
|---|---|
| **Redpanda** (Kafka-compatible) | Kafka protocol, single binary, no ZooKeeper, 1/10 the memory footprint |
| **Topics**: `telemetry.raw`, `telemetry.events`, `dtc.observed` | Clean separation of raw stream, derived events, observed faults |
| **`aura-bridge`** (client) | Local Python client, BT/USB to ELM327, WebSocket to backend |
| **WebSocket → Ingest service → Kafka** | Standard pattern; ingest service handles auth, validation, partitioning |

#### 3.3.8 Analytics and Orchestration

| Choice | Rationale |
|---|---|
| **Apache Airflow** | DAGs for corpus refresh, eval suite, analytics; strongest job-market keyword |
| **Databricks Community Edition** | Free real Databricks; PySpark notebooks for analytics and training data |
| **DuckDB** | Ad-hoc analytics; honest comparison: "Spark for fleet scale, DuckDB for this scale" |
| **MLflow** | Experiment tracking for prompt/retrieval/model variants; model registry |

#### 3.3.9 Cloud and Infrastructure

| Choice | Rationale |
|---|---|
| **AWS** (primary cloud) | Dominant in Brazilian market; S3 + EKS + IAM with Terraform |
| **AWS EKS** | Real Kubernetes for Redpanda + consumers (one credible k8s deployment) |
| **GCP Cloud Run** | Single deployed service for legitimate multi-cloud claim |
| **Terraform** | IaC for all AWS resources; major hiring signal |
| **Docker** | Every service containerized; multi-stage builds; Compose for local dev |
| **Helm** | Charts for the EKS deployment |
| **KEDA** | HPA driven by Kafka consumer lag |

#### 3.3.10 Observability

| Choice | Rationale |
|---|---|
| **OpenTelemetry** | End-to-end traces; vendor-neutral instrumentation |
| **Grafana Cloud** | Free tier; metrics, logs, traces; public dashboard link |
| **Langfuse** | LLM-specific observability: prompt/response/cost/latency per call |
| **Promptfoo** | Eval harness; integrates with MLflow |

### 3.4 Excluded Tools (with prepared answers)

- **LangChain** — leaky abstractions; LangGraph + direct LLM clients give better control and observability
- **Pinecone** — Qdrant chosen for hybrid search, self-hosting option, cost; ADR documents the evaluation
- **Triton Inference Server** — vLLM is the specialized tool for LLM serving with continuous batching; Triton excels for heterogeneous model serving which AURA doesn't need
- **Custom CUDA kernels** — vLLM provides the optimized kernels; custom kernel work is overkill for this use case

### 3.5 Cost Profile

| Resource | Monthly Cost (idle / active) |
|---|---|
| Vercel (frontend) | $0 / $0 (hobby tier) |
| Railway/Fly.io (backend) | $5 / $20 |
| Neon Postgres | $0 / $0 (free tier) |
| Upstash Redis | $0 / $0 (free tier) |
| Qdrant Cloud | $0 / $0 (free tier 1GB) |
| Modal GPU inference | $0 / $30–80 |
| Anthropic API (fallback) | $0 / $10–30 |
| AWS (S3 + EKS demos) | $5 / $30 |
| GCP Cloud Run | $0 / $0 |
| Grafana Cloud | $0 / $0 |
| Langfuse (self-hosted) | $5 / $5 |
| MLflow (self-hosted) | $5 / $5 |
| Domain | $1 / $1 |
| **Baseline** | **~$20 / ~$170** |

EKS cluster runs only during demos and load tests (`terraform apply` / `terraform destroy`). Benchmark spikes can hit $250–300 short-term.

---

## 4. Build Phases

Each phase has a **scope**, **stop criterion** (the minimum that makes the phase credible standalone), and **expand criterion** (what to add later).

### Phase 0 — Foundation Knowledge (12 weeks)

**Scope.** Background study before diving into the project specifics. The areas where the developer's existing knowledge needs strengthening to build AURA without expensive false starts:

- Modern LLM application architecture (RAG patterns, agent loops, eval methodology)
- Production Python async patterns (FastAPI, async DB, async LLM clients)
- TypeScript / Next.js 15 / streaming UI patterns
- Kafka fundamentals (topics, partitions, consumer groups, offsets)
- Vector databases and embedding model fundamentals
- Docker and Compose proficiency
- Basic Kubernetes literacy (enough to be productive in Phase 5)
- AWS fundamentals: IAM, S3, EKS basics, Terraform basics
- Observability fundamentals: traces, metrics, logs, OpenTelemetry concepts

**Approach.** Mix of structured courses, official documentation, and small disposable projects. The goal is *literacy*, not *mastery* — depth comes from building AURA. A useful heuristic: spend ~60% of this phase reading/watching, ~40% writing throwaway code that exercises one concept at a time.

**Stop criterion.** Comfortable starting Phase 1 without needing to learn fundamental concepts mid-build.

---

### Phase 1 — RAG Foundation (8 weeks)

**Scope.** Corpus ingestion pipeline for Amarok V6 Extreme 2021+. Sources: VW shop manual, TSBs, OBD-II DTC databases, curated forum threads, independent shop procedures. Qdrant + bge-m3 + bge-reranker + BM25 hybrid retrieval. Eval set of ~200 golden questions with reference answers. MLflow tracking experiments across chunking/embedding/retrieval configurations. Promptfoo harness. FastAPI service exposing `/query` endpoint, containerized.

**Stop criterion.** Service answers diagnostic questions in PT-BR or EN with grounded citations. Recall@10 benchmarked and published. MLflow shows ≥3 retrieval-strategy experiments. One Airflow DAG for corpus refresh.

**Expand criterion.** Fine-tuned embeddings on Amarok queries, query rewriting, multi-hop retrieval.

---

### Phase 2 — LLM Serving + ReAct Agent (8 weeks)

**Scope.** vLLM on Modal serving Qwen2.5-14B-Instruct (benchmark vs Llama-3.1-8B). OpenAI-compatible API. Streaming. Semantic cache (Redis). ReAct loop in LangGraph with the tool set: `retrieve_corpus`, `get_telemetry_snapshot`, `explain_dtc`, `search_history`, `compute_metric`. Premium-tier fallback to Anthropic API with simple routing logic. Langfuse instrumentation on every LLM call.

**Stop criterion.** End-to-end query works: question → ReAct decides retrieval → retrieves → grounded answer streams back. Langfuse shows full traces. TTFT and throughput measured for vLLM (Modal) vs hosted API.

**Expand criterion.** Fine-tuned model on PT-BR automotive data, speculative decoding, prefix caching benchmarks, smarter multi-model router.

---

### Phase 3 — Product Surface (10 weeks)

**Scope.** Next.js PWA. Split telemetry/chat UI. Auth (Supabase). Multi-tenant data isolation. Rate limiting. Demo mode with recorded telemetry playback (this is what reviewers without an Amarok use). Message streaming with context chips. DTC badges. Timeline scrubber. Vehicle history view. Public URL goes live.

**Stop criterion.** A stranger can visit the URL, sign up, enter demo mode, have a meaningful conversation, and understand AURA in under 5 minutes. PWA installable on mobile. Telemetry WebSocket bridge working with at least simulated stream.

**Expand criterion.** Voice (folded forward from Phase 7 if scope allows), maintenance scheduler, conversation sharing, advanced visualizations.

---

### Phase 4 — Streaming Data Pipeline (6 weeks)

**Scope.** Redpanda deployed (Railway initially). `aura-bridge` desktop client produces telemetry to `telemetry.raw`. Hot consumer writes to Redis + Postgres. Archive consumer writes Parquet to S3. Anomaly detection consumer flags events to `dtc.observed`. Load simulator produces N virtual Amaroks for testing.

**Stop criterion.** Real telemetry from the developer's Amarok flows end-to-end through Kafka, hot path (chat) and cold path (S3) both working. Load test with 100 simulated vehicles sustains throughput.

**Expand criterion.** Exactly-once semantics, schema registry, CDC from Postgres, Kafka Connect integrations.

---

### Phase 5 — Observability, Load Testing, Production Ops (8 weeks)

**Scope.** OpenTelemetry instrumentation end-to-end. Grafana Cloud dashboards (public link). k6 load test suite: baseline, spike, soak, chaos. Per-tenant token accounting. Airflow DAGs for scheduled eval runs, corpus refresh, analytics triggers. **EKS deployment**: migrate Redpanda + consumers to EKS via Terraform + Helm + KEDA autoscaling on consumer lag. Benchmark report with reproducible scripts and "what breaks first" analysis.

**Stop criterion.** Public Grafana dashboard link in repo README. Published benchmark report. EKS deployment live with autoscaling demonstrated under load. Weekly eval runs producing MLflow-tracked results.

**Expand criterion.** Cost optimization deep-dive, multi-region, canary deployments, formal SLO/SLI with error budgets.

> **🎯 PRIMARY STOP POINT — END OF PHASE 5**
>
> After Phase 5, AURA covers the high-priority skills (RAG, LLM serving, multi-tenancy, streaming, observability, IaC, k8s, cost engineering) and has a live product reviewers can use. **If the job search accelerates, stop here.** Phases 6–9 are extensions, not baseline.

---

### Phase 6 — Analytics and Training Data (4 weeks)

**Scope.** Databricks Community Edition notebooks. PySpark jobs processing the S3 Parquet archive: per-vehicle aggregates, fleet anomaly patterns (across simulated vehicles), training-data generation for future fine-tuning. DuckDB for ad-hoc analytics with a notebook explicitly comparing both ("Spark for fleet scale, DuckDB for this scale"). MLflow tracking any model experiments.

**Stop criterion.** 1–2 polished Databricks notebooks linked from the repo. A "data platform" architecture diagram showing batch and stream paths converging.

**Expand criterion.** Real fine-tuning of bge-m3 or a small LLM on generated data, Delta Lake features, Spark Structured Streaming.

---

### Phase 7 — Voice (6 weeks)

**Scope.** STT: faster-whisper on GPU for cloud, whisper.cpp for edge. TTS: Piper or XTTSv2. Voice-in and voice-out on web. Hands-free continuous mode. Bilingual switching mid-conversation.

**Stop criterion.** Voice-to-voice chat works on web in PT-BR and English with acceptable latency (<2s end-to-end target).

**Expand criterion.** Voice fine-tuning for automotive vocabulary, wake word detection, engine-bay noise cancellation.

---

### Phase 8 — Edge Runtime (8 weeks)

**Scope.** Raspberry Pi 5 8GB (or Jetson Orin Nano upgrade if budget permits). Local llama.cpp or vLLM with quantized Qwen. Local Qdrant with corpus subset. Sync protocol with cloud. Offline-first PWA mode. `aura-bridge` runs on the Pi and serves the vehicle directly when offline. Pi boots into kiosk mode automatically.

**Stop criterion.** Pull network in vehicle, conversation continues seamlessly, DTCs still explained, telemetry still interpreted.

**Expand criterion.** Jetson upgrade for voice latency, hardware integration (display, steering wheel controls).

---

### Phase 9 — The Video (3 weeks)

**Scope.** In-vehicle demo video. Split-screen with ReAct trace HUD. Pull-the-cable offline moment. Bilingual interaction. Fault injection (unplug a sensor, simulate a boost leak, trigger a DPF event). Tight editing.

**Stop criterion.** Video published; linked from repo, LinkedIn, resume.

---

## 5. Detailed Timeline

### 5.1 Assumptions

- **Working capacity**: ~15 hours/week sustained (part-time alongside other commitments). Conservative for a real human with a full life.
- **Calendar adjustment**: estimates include buffer for context-switching cost, debugging surprises, and life events. Real software projects under-deliver against optimistic estimates by 30–50%; conservative estimates account for this.
- **No vacations or holidays modeled** — add real-world calendar buffer separately.
- **Phases are mostly sequential** with limited parallelism (one developer). Some Phase 0 study can overlap with Phase 1 start.

### 5.2 Phase Durations and Gates

| Phase | Weeks | Months | Cumulative (months) | Status After Phase |
|---|---|---|---|---|
| **0. Foundation Knowledge** | 12 | 3.0 | 3.0 | Ready to build |
| **1. RAG Foundation** | 8 | 2.0 | 5.0 | Narrow RAG portfolio possible |
| **2. LLM Serving + ReAct** | 8 | 2.0 | 7.0 | Agent + serving story complete |
| **3. Product Surface** | 10 | 2.5 | 9.5 | Live product URL exists |
| **4. Streaming Data Pipeline** | 6 | 1.5 | 11.0 | Streaming/data story complete |
| **5. Observability & Ops** | 8 | 2.0 | 13.0 | **Primary stop point reached** |
| **6. Analytics & Training Data** | 4 | 1.0 | 14.0 | Spark/Databricks signal added |
| **7. Voice** | 6 | 1.5 | 15.5 | Voice product complete |
| **8. Edge Runtime** | 8 | 2.0 | 17.5 | Offline/edge story complete |
| **9. Video** | 3 | 0.75 | 18.25 | Final demo artifact published |

### 5.3 Calendar View

```
Month  1  2  3  | 4  5  | 6  7  | 8  9  10 | 11 12 | 13 14 15 | 16 | 17 18 | 19
       ─────────|───────|───────|──────────|───────|──────────|────|───────|────
P0  ▓▓▓▓▓▓▓▓▓
P1           ▓▓▓▓▓▓▓▓
P2                 ▓▓▓▓▓▓▓▓
P3                       ▓▓▓▓▓▓▓▓▓▓
P4                                 ▓▓▓▓▓▓
P5                                       ▓▓▓▓▓▓▓▓ ← PRIMARY STOP
                                                  ▼
P6                                                ▓▓▓▓
P7                                                    ▓▓▓▓▓▓
P8                                                         ▓▓▓▓▓▓▓▓
P9                                                                ▓▓▓
```

### 5.4 Decision Gates

Every phase boundary is a decision gate. At each gate, ask:

1. **Has this phase met its stop criterion?** If no, complete it before moving on.
2. **Has the job market situation changed?** Active interview pipeline = consider stopping at the next natural break.
3. **Is the next phase still aligned with the goal?** Skills demand shifts; revisit priorities.
4. **Is anything in the architecture obviously wrong?** Sunk cost shouldn't preserve bad decisions.

Hard gate: **end of Phase 5 (~Month 13)**. This is the explicit "stop or continue" decision point. The portfolio is strong here; everything beyond is upside.

### 5.5 Realistic Caveats

- **The 18-month estimate assumes nothing is learned that requires major rework.** It will. Add 10–20% mental buffer.
- **Phase 0 (foundation) is the most uncertain.** Three months is the planned ceiling; if learning goes faster, Phase 1 can start earlier and the whole timeline compresses. If it goes slower, the rest of the plan suffers — be honest about readiness rather than rushing into Phase 1.
- **Phases 1 and 2 compound risk.** RAG quality and LLM serving quality together determine whether the rest of the project produces good demos. If either is shaky, fix before continuing.
- **Phase 5 is the "production engineering" phase.** It's the least intrinsically interesting and the most differentiating in interviews. Don't shortcut it.

### 5.6 Public Commitment Schedule

To create accountability and build a public following alongside the project:

- **Month 3** (end of Phase 0): blog post or LinkedIn writeup — *"Why I'm building AURA"* (concept, target market, architecture overview)
- **Month 5** (end of Phase 1): demo video of RAG service answering Amarok questions
- **Month 7** (end of Phase 2): blog post — *"Building a ReAct agent without LangChain"* with code excerpts
- **Month 9.5** (end of Phase 3): live URL goes public, announcement post
- **Month 11** (end of Phase 4): blog post on the streaming architecture
- **Month 13** (end of Phase 5): benchmark report published, public Grafana dashboard link, **active job search begins if not earlier**
- **Month 14+**: continued posts as Phases 6–9 ship

---

## 6. What Recruiters and Hiring Teams Will See

By end of Phase 5:

1. **Live product URL** — `aura.app` (or similar), demo mode usable in 30 seconds
2. **Public GitHub repo** — `aura-core` with comprehensive README, ADRs, Terraform, Helm charts, eval results
3. **Public Grafana dashboard** — live metrics from the running system
4. **Benchmark report** — `BENCHMARKS.md` with reproducible k6 scripts, honest "what breaks first" section, $/1k-requests numbers
5. **MLflow tracking** — public experiment history showing iteration discipline
6. **Architecture Decision Records** — `adr/` folder with short opinionated decisions and tradeoffs

By end of Phase 9:

7. **Demo video** — 4–6 minutes, in-vehicle, voice, ReAct trace HUD, offline moment, bilingual
8. **Edge artifact** — Pi-based runtime, documented build instructions

The recruiter experience: read the resume bullet, click the URL, try the product, watch the video, browse the repo. Each step deepens conviction. No PDF gates, no "request a demo," no walls.

---

## 7. Key Decisions Locked

These decisions are load-bearing. Changing them is expensive.

- Cloud-native primary, edge as fallback feature
- Single vehicle scope: 2021+ Amarok V6 Extreme
- Stack: Next.js + FastAPI + Postgres + Redis + Qdrant + vLLM on Modal
- Agent: LangGraph-based ReAct loop
- Streaming: Redpanda (Kafka-compatible)
- Orchestration: Airflow
- Cloud: AWS primary (EKS for one component), GCP minimal presence
- Observability: OpenTelemetry + Grafana + Langfuse + MLflow
- Repo strategy: `aura-core` (AGPL public) + `aura-pro` (private)
- Languages: PT-BR primary, EN secondary, internal reasoning EN

## 8. Open Decisions Deferred

- Specific LLM choice (Qwen2.5-14B vs Llama-3.1-8B) — decide via Phase 2 benchmarking
- Specific Pi tier (Pi 5 8GB vs Jetson Orin Nano) — decide based on Phase 7 voice latency requirements and budget
- Whether to add Kafka Streams / Flink for stream processing — defer until Phase 4 reveals if there's a real need
- Domain name and brand identity for public launch — decide before Phase 3 ships
