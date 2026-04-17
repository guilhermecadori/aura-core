# 0007 — Selective Kubernetes usage

**Status:** Accepted
**Date:** 2026-04-17

## Context

Kubernetes is a dominant keyword in DevOps, platform, and senior engineering job posts. A portfolio project without any Kubernetes experience forfeits a meaningful signal. However, *everything* on Kubernetes is a common anti-pattern — it introduces substantial operational complexity that's only justified for workloads that genuinely benefit.

Most of AURA's services (stateless FastAPI, Next.js frontend, batch orchestrator) run perfectly well on simpler platforms (Railway, Fly.io, Vercel, Modal). Forcing them onto Kubernetes would be resume-driven architecture and would produce worse operational outcomes than the simpler platforms provide.

Conversely, **the telemetry ingestion pipeline is a genuinely Kubernetes-shaped workload**: stateful services (Redpanda), consumer groups requiring careful pod lifecycle management, horizontal scaling driven by a queue depth metric (Kafka consumer lag), and multi-pod coordination.

## Decision

Use Kubernetes selectively, not universally:

- **AWS EKS** hosts the telemetry ingestion pipeline: Redpanda brokers, WebSocket ingest workers, hot/archive consumers, KEDA autoscaler driven by Kafka consumer lag
- **All other services** stay on simpler purpose-built platforms: Vercel (frontend), Railway/Fly.io (backend services, Airflow, MLflow, Langfuse), Modal (GPU inference)
- **Infrastructure as code**: Terraform provisions the EKS cluster and supporting AWS resources; Helm charts deploy the workloads

This gives one credible, non-trivial Kubernetes deployment while keeping the rest of the system operationally lean.

## Alternatives Considered

- **Everything on Kubernetes** — rejected: operational complexity overwhelms a single-developer portfolio project; services like Vercel, Modal, and Railway are objectively better tools for their respective workloads; demonstrating judgment about *when* to use k8s is a stronger signal than using it everywhere
- **No Kubernetes** — rejected: forfeits a major job-market signal, and the ingestion pipeline is genuinely the right shape for k8s
- **k3s or k0s instead of EKS** — rejected: lighter to run but weaker market signal; EKS demonstrates managed-k8s experience that more closely mirrors enterprise environments
- **GKE instead of EKS** — rejected: AWS has stronger presence in the target market (Brazil); GKE coverage is handled separately via Cloud Run in ADR-adjacent decisions
- **ECS/Fargate** — rejected for this role: AWS-native container orchestration but weaker keyword signal than Kubernetes; Fargate *does* appear as the execution target for non-ingestion AWS workloads (e.g., one-off compute for eval runners)

## Consequences

### Positive

- One high-quality Kubernetes deployment is a stronger artifact than many low-quality ones
- Rest of the system remains operationally manageable by a single developer
- Interview answer "I used Kubernetes for X because it's the right shape; I used simpler tools for Y because they were" signals seniority
- Terraform + Helm + KEDA stack is industry-standard and portable

### Negative

- EKS cluster has a minimum baseline cost (~$70/month when running); mitigated by `terraform destroy` when not demoing
- Two operational models to maintain (k8s + simpler platforms)
- Some debugging complexity bridges the two worlds (service-to-service calls between EKS and Railway, for instance)

### Neutral

- Helm chart portability means the ingestion pipeline can run on any k8s flavor if EKS becomes unattractive
- KEDA-based autoscaling on consumer lag is a genuinely useful pattern worth learning

## Revisit When

- Cost optimization forces consolidation to a single platform
- A specific role requires end-to-end k8s experience across all services
- Managed services change their pricing or capabilities in a way that shifts the tradeoff
- Operational burden of running two platforms exceeds the benefit of using the right tool for each
