# Architecture Decision Records

Short, dated records of significant architectural decisions. Each ADR captures:

- **Context** — the situation that required a decision
- **Decision** — what was chosen
- **Alternatives** — what else was considered and why it was rejected
- **Consequences** — what this decision implies, positive and negative

ADRs are append-only. If a decision is reversed, a new ADR supersedes the old one — the original is never deleted or edited in substance.

## Format

Based on Michael Nygard's original template, lightly adapted.

## Index

| # | Title | Status | Date |
|---|---|---|---|
| [0001](./0001-cloud-native-with-edge-fallback.md) | Cloud-native primary with edge fallback | Accepted | 2026-04-16 |
| [0002](./0002-single-vehicle-scope.md) | Single-vehicle scope for v1 | Accepted | 2026-04-16 |
| [0003](./0003-langgraph-over-langchain.md) | LangGraph over LangChain for agent orchestration | Accepted | 2026-04-16 |
| [0004](./0004-qdrant-over-pinecone.md) | Qdrant over Pinecone for vector storage | Accepted | 2026-04-16 |
| [0005](./0005-vllm-over-triton.md) | vLLM over Triton for LLM serving | Accepted | 2026-04-16 |
| [0006](./0006-redpanda-over-kafka.md) | Redpanda over Apache Kafka for event streaming | Accepted | 2026-04-16 |
| [0007](./0007-selective-kubernetes.md) | Selective Kubernetes usage | Accepted | 2026-04-17 |
| [0008](./0008-dual-repo-strategy.md) | Dual public/private repository strategy | Accepted | 2026-04-17 |

## Template

See [`_template.md`](./_template.md) for new ADRs.
