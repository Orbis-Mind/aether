# Roadmap

This roadmap lists **proposed** functionality. None of it is implemented unless
also marked ✅ in [`FEATURE_INVENTORY.md`](FEATURE_INVENTORY.md). It states the
product direction toward a full multi-operation workload compiler and runtime;
the current single-workload compiler/runtime is the single-node case of this.

## Direction

> Extend the compiler from a linear, single-operation plan to an **execution
> graph (DAG)** of operations, and the runtime from single-operation execution to
> **dependency-aware scheduling with recovery and plan optimization** — without
> changing the local-first, zero-dependency constraints.

## Milestones (proposed, unordered priority)

### M1 — Intermediate Representation + Execution Graph (Compiler)
- Define an IR distinct from the final plan.
- Accept multi-operation Workload Definitions.
- Dependency analysis between operations; build a validated DAG.
- Detect parallelizable operations.

### M2 — Operation Scheduler (Runtime)
- Execute a DAG: schedule independent operations, honor dependencies, pass
  intermediate results along edges.
- Bounded concurrency and resource management across operations.

### M3 — Recovery Engine (Reliability)
- Checkpoint completed operations; resume without repeating successful work.
- Preserve execution state across restarts; structured recovery semantics.

### M4 — Adaptive Optimizer (Optimization)
- Compare alternative execution plans using collected metrics.
- Optimize scheduling / resource allocation.
- A hard validation gate: never adopt a candidate that has not passed correctness
  and performance checks against the baseline.

### M5 — Observability depth
- In-process CPU/RAM sampling.
- Optional OpenTelemetry export; trace a full graph execution.

## Near-term (smaller, already-scoped)

- Native SSE streaming for cloud providers.
- Read model metadata directly from local model files.
- Atomic (temp-file + rename) persistence for cache/memory durability.

## Explicitly out of scope

- Running model weights in-process (Aether delegates inference to engines).
- Becoming an agent framework or an API gateway.

Each milestone, when delivered, moves its items from this roadmap into the
feature inventory with tests and documented behavior — not before.
