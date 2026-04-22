# 0006 — Redpanda over Apache Kafka for event streaming

**Status:** Accepted

**Date:** 2026-04-16

## Context

The telemetry ingestion pipeline needs an event streaming backbone. Vehicles streaming OBD-II telemetry at 1–10Hz per vehicle, with simulated fleet load for testing, is the canonical Kafka-shaped workload. The system must support:

- Consumer groups (hot path for chat context, cold path for archival)
- Partitioning strategy (by vehicle ID) for scale
- Backpressure handling when consumers lag
- Deployability on both Railway (dev) and Kubernetes (prod demo)

Kafka is the de-facto industry standard for event streaming in data engineering. The system speaks the Kafka protocol so existing tooling, client libraries, and operational patterns apply directly.

## Decision

Use **Redpanda** — a Kafka-protocol-compatible streaming platform written in C++. Application code speaks the standard Kafka protocol and uses standard Kafka clients; nothing is Redpanda-specific above the broker.

## Alternatives Considered

- **Apache Kafka** — rejected: operational overhead is substantial. ZooKeeper or KRaft consensus to manage, JVM tuning, broker tuning, higher resource footprint. For a single-developer project where one person operates everything, Redpanda delivers the same API with dramatically less operational surface. "Kafka (Redpanda)" is standard industry phrasing.
- **AWS MSK / Confluent Cloud** — rejected: expensive at project scale, less control, and the managed-service surface is less instructive to work with directly
- **NATS JetStream** — rejected: excellent technology but outside the dominant Kafka-protocol ecosystem; less reusable knowledge for contributors
- **RabbitMQ** — rejected: different model (message queue vs log), doesn't fit the append-only telemetry stream pattern
- **Kinesis** — rejected: AWS-specific, costs accumulate at project scale, narrower ecosystem than Kafka broadly
- **No streaming backbone** (direct WebSocket → DB) — rejected: loses the streaming-architecture capability the system needs for archive paths, anomaly detection, and multi-consumer fan-out

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
- External descriptions need to clarify "Kafka (Redpanda)" for precision

### Neutral

- Migration to Apache Kafka is trivial should the need arise (same protocol)
- Consumer code, topic design, and partitioning strategy are all reusable knowledge

## Revisit When

- A Kafka-specific ecosystem tool is needed that Redpanda doesn't support
- Scale exceeds single-broker Redpanda tier in a way where clustered Kafka is cheaper
- A workload requirement specifically benefits from JVM-based Kafka broker behaviour
