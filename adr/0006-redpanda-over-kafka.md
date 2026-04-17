# 0006 — Redpanda over Apache Kafka for event streaming

**Status:** Accepted

**Date:** 2026-04-16

## Context

The telemetry ingestion pipeline needs an event streaming backbone. Vehicles streaming OBD-II telemetry at 1–10Hz per vehicle, with simulated fleet load for testing, is the canonical Kafka-shaped workload. The system must support:

- Consumer groups (hot path for chat context, cold path for archival)
- Partitioning strategy (by vehicle ID) for scale
- Backpressure handling when consumers lag
- Deployability on both Railway (dev) and Kubernetes (prod demo)

Portfolio consideration: **"Kafka" is the dominant keyword in data engineering job posts**. The system must credibly claim Kafka experience.

## Decision

Use **Redpanda** — a Kafka-protocol-compatible streaming platform written in C++. Application code speaks the standard Kafka protocol and uses standard Kafka clients; nothing is Redpanda-specific above the broker.

## Alternatives Considered

- **Apache Kafka** — rejected: operational overhead is substantial. ZooKeeper or KRaft consensus to manage, JVM tuning, broker tuning, higher resource footprint. For a portfolio project where one developer operates everything, Redpanda delivers the same API with dramatically less operational surface. The resume legitimately reads "Kafka (Redpanda)"; this is standard industry language.
- **AWS MSK / Confluent Cloud** — rejected: expensive at portfolio scale, less control, less visible to reviewers (managed service means less interesting infrastructure to show)
- **NATS JetStream** — rejected: excellent technology but the keyword signal is weaker; the industry standard for "streaming events" in job descriptions is Kafka
- **RabbitMQ** — rejected: different model (message queue vs log), doesn't fit the append-only telemetry stream pattern, weaker hiring signal for data/ML roles
- **Kinesis** — rejected: AWS-specific, costs accumulate at portfolio scale, smaller hiring signal than Kafka broadly
- **No streaming backbone** (direct WebSocket → DB) — rejected: forfeits the streaming architecture story; simple but undersells the engineering

## Consequences

### Positive

- Single binary to operate, no ZooKeeper or separate consensus layer
- ~1/10 the memory footprint of equivalent Kafka cluster
- Standard Kafka clients work unchanged
- Faster to stand up locally in Docker Compose for development
- Strong defaults; less tuning needed

### Negative

- Smaller community than Apache Kafka proper
- Some Kafka ecosystem tools (e.g., specific Connect plugins) may have compatibility caveats at edges
- Resume phrasing requires clarifying "Kafka (Redpanda)" for precision

### Neutral

- Migration to Apache Kafka is trivial should the need arise (same protocol)
- Consumer code, topic design, and partitioning strategy are all reusable knowledge

## Revisit When

- A Kafka-specific ecosystem tool is needed that Redpanda doesn't support
- Scale exceeds single-broker Redpanda tier in a way where clustered Kafka is cheaper
- A role specifically values JVM Kafka operational experience
