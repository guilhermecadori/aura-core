# 0004 — Qdrant over Pinecone for vector storage

**Status:** Accepted
**Date:** 2026-04-16

## Context

The RAG pipeline requires a vector database for storing corpus embeddings and performing similarity search. Requirements:

- **Hybrid retrieval**: must support BM25 (or sparse vector) alongside dense retrieval, because exact-match queries on part numbers and DTC codes fail on pure dense retrieval
- **Metadata filtering**: must filter by document type, vehicle subsystem, source — filter predicates are used in nearly every query
- **Cost-aware**: hobby/portfolio budget, scale-to-low-cost when idle
- **Self-hostable**: eventual control over infrastructure, no single-vendor lock-in
- **Production credibility**: the choice itself is a portfolio signal

## Decision

Use **Qdrant**, starting on Qdrant Cloud's free tier (1GB) and migrating to self-hosted if/when scale requires it. Leverage Qdrant's built-in hybrid search (dense + sparse vectors in the same collection) rather than running a separate BM25 index.

## Alternatives Considered

- **Pinecone** — rejected: expensive at small scale, closed-source (no self-host path), hybrid search is more recent and less mature than Qdrant's, vendor lock-in concerns
- **Weaviate** — rejected: heavier operational footprint, module system adds complexity, filtering syntax less ergonomic
- **Milvus** — rejected: operational complexity is high (multi-component deployment), overkill for v1 corpus size, stronger fit for billion-scale vectors which AURA doesn't need
- **pgvector** — rejected for the primary RAG path: performance walls at serious scale, hybrid search requires extensions and manual work, would preclude demonstrating "chose the right specialized tool"; *may still be used* for secondary lightweight vector needs (e.g., user conversation semantic search)
- **Chroma** — rejected: excellent developer experience but not positioned as a production-grade system, portfolio optics matter
- **LanceDB** — rejected: promising and embedded-friendly but would pair better with an edge-first architecture (see ADR 0001); for cloud-first, Qdrant is better matched

## Consequences

### Positive

- Free tier covers entire Phase 1 development
- Hybrid search, reranking, and filtering all first-class
- Self-host path exists; no vendor lock-in
- Rust-based performance is strong without tuning
- Well-documented Python client with async support

### Negative

- Less name recognition in job posts than Pinecone (mitigated by documenting the evaluation in this ADR)
- Qdrant Cloud's free tier has usage limits that could force an early migration
- Operational experience on self-hosted Qdrant is net-new work

### Neutral

- Embedding model choice (bge-m3) is independent of vector store choice
- Could add pgvector alongside Qdrant for lightweight secondary uses without architectural conflict

## Revisit When

- Corpus exceeds Qdrant Cloud free tier and self-hosted operations become burdensome
- A specific role targets Pinecone experience and the gap matters
- Qdrant's development stalls or hybrid search regresses
