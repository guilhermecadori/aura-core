# 0003 — LangGraph over LangChain for agent orchestration

**Status:** Accepted

**Date:** 2026-04-16

## Context

AURA's reasoning layer is a ReAct agent: the model alternates between Thought (reasoning), Action (tool call), and Observation (tool result) steps until it produces a final answer. This requires explicit state management, conditional routing between steps, and tool invocation.

Three categories of framework exist:

1. **High-level orchestration frameworks** (LangChain, LlamaIndex) — wrap the entire LLM interaction in abstractions
2. **Graph-based state machines** (LangGraph, Haystack, Burr) — explicit nodes, edges, and state
3. **Direct implementation** — write the loop by hand against an OpenAI-compatible client

Portfolio considerations matter here: the agent layer is inspected closely by technical reviewers. Control, observability, and legibility are higher-value than convenience.

## Decision

Use **LangGraph** for the agent orchestration layer. Direct vLLM / Anthropic API clients for LLM calls beneath it. No LangChain anywhere in the stack.

## Alternatives Considered

- **LangChain** — rejected: leaky abstractions, frequent API churn between versions, hides the prompts and requests actually sent to the model, harder to instrument cleanly, increasingly regarded as an anti-signal by senior practitioners. Industry mood has shifted from "use LangChain by default" to "justify LangChain if you use it."
- **Hand-rolled loop** — rejected: viable but the state management for multi-tool ReAct with checkpointing and human-in-the-loop pauses becomes non-trivial; LangGraph provides the right primitives cheaply
- **LlamaIndex** — rejected: strong for retrieval-heavy workflows but less natural for general agent state machines; overlap with Qdrant-based retrieval is poor
- **Haystack** — rejected: smaller ecosystem, weaker streaming and async support at evaluation time
- **Burr** — rejected: promising but smaller community; LangGraph has stronger momentum and tooling

## Consequences

### Positive

- Agent state is explicit and inspectable — the ReAct trace is a first-class artifact, not reconstructed from logs
- Checkpointing comes for free, enabling conversation resume and human-in-the-loop flows later
- Tool calls are routed via explicit edges, easy to reason about and trace
- Interview answer "I evaluated LangChain and chose LangGraph because..." signals thoughtfulness

### Negative

- LangGraph itself is relatively young and its API may shift; version pinning is important
- Learning curve for graph-based state management
- Some LangChain integrations (loaders, tools) would need to be reimplemented or imported piecemeal

### Neutral

- Langfuse integration works equivalently with either framework
- Team members familiar with LangChain need to learn a new mental model

## Revisit When

- LangGraph project stalls or is deprecated
- A specific LangChain integration becomes so valuable that importing it piecemeal is unviable
- The agent logic becomes simple enough that direct implementation is cleaner
