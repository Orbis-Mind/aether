# Aether — Feature Inventory (source-verified)

This inventory is derived from the **actual source code**, not from marketing
claims. It is the ground truth the README and architecture docs are written
against. Status legend:

- ✅ **Implemented** — exists and is tested.
- 🟡 **Partial** — a meaningful subset exists; clearly bounded below.
- ⬜ **Planned** — part of the product direction, **not built yet**.

Audited at commit on `main`, 2026-11-16.

## Packages (what actually exists)

| Package | Responsibility | Status |
|---|---|---|
| `internal/compiler` | Validate a `.ae.yaml` spec (strict YAML + schema), classify the task, route a model, build an `ExecutionPlan`, write a versioned `.ae` artifact | ✅ |
| `internal/runtime` | Execute a compiled plan: render prompt, cache lookup, model call with retry/backoff/fallback, memory, streaming, observability | ✅ |
| `internal/router` | Intelligent model-selection **pipeline** (task → policy filter → scoring → selection) | ✅ |
| `internal/policy` | Cost/quality/latency/privacy constraint engine applied before selection | ✅ |
| `internal/classifier` | Offline heuristic task analysis (type, difficulty, risk, tools) | ✅ |
| `internal/models` | Model registry: capabilities, pricing, quality with measured/estimated provenance | ✅ |
| `internal/provider` | `ModelProvider` abstraction (Ollama, OpenAI-compatible, Anthropic, mock) + health checks | ✅ |
| `internal/discovery` | Detect locally installed Ollama models, map to catalogue | ✅ |
| `internal/cache` | Bounded exact + semantic response cache (TTL, eviction, safety guards) | ✅ |
| `internal/memory` | Local vector store + entity/relationship memory, data deletion | ✅ |
| `internal/observability` | Run records, latency percentiles, routing dataset, report rendering | ✅ |
| `internal/logging` | Structured `log/slog` logging + execution-ID request tracing | ✅ |
| `internal/eval` | Offline, deterministic model evaluation (keyword/regex + numeric answer grading) | ✅ |
| `internal/registry` | Local artifact registry (publish/pull/search) | ✅ |
| `internal/yamlx` | Zero-dependency strict YAML subset parser | ✅ |

## Pillars (the product identity) vs. reality

### A. AI Workload Compiler
- ✅ Accept a structured workload definition (`.ae.yaml`).
- ✅ Static validation (strict YAML, schema, unknown-key rejection, size limits).
- ✅ Generate an executable plan (`core.ExecutionPlan`) — prompt optimization, token estimation, model selection, cache strategy.
- ✅ Deterministic compilation + integrity-verified `.ae` artifact (v2, full SHA-256).
- 🟡 Execution steps: a **fixed linear sequence** (`retrieve → inject → render → execute → store → cache`), not a general graph.
- ⬜ **Dependency analysis**, **execution graphs (DAGs)**, **parallel-execution detection**, **multi-operation workloads** — not built.

### B. Intelligent Execution Runtime
- ✅ Execute a compiled plan; manage per-run state; context-aware execution with cancellation.
- ✅ Bounded single-operation concurrency via context timeouts; resource cleanup.
- ✅ Predictable semantics for one operation (render → cache → model → persist).
- ⬜ **Scheduling independent operations**, **cross-operation dependency handling**, **intermediate-result passing**, **DAG execution** — not built (a plan is one operation today).

### C. Reliability and Fault Recovery
- ✅ Operation failure detection; typed permanent vs. transient errors.
- ✅ Configurable retries with exponential backoff + jitter (bounded, no storms).
- ✅ Timeouts and cancellation propagation (Ctrl-C/SIGTERM).
- ✅ Fallback model chain, bounded by policy.
- ✅ Structured failure information on the result.
- ⬜ **Checkpointing / recovery across operations** — not built (single operation; nothing to resume between steps).

### D. Adaptive Runtime Optimization
- ✅ Collect execution metrics (latency, tokens, cost, success) + routing dataset.
- ✅ Measured-quality overlay: `aether evaluate` results feed future routing (a real feedback loop).
- 🟡 Static compile-time optimization: prompt compression, cache-strategy selection.
- ⬜ **Comparing alternative execution plans**, **scheduling/resource optimization**, **validated optimization candidates** — not built.

### E. Execution Observability
- ✅ Execution traces (run records, routing dataset as JSONL).
- ✅ Operation latency (incl. P95), retries, failures recorded.
- ✅ `aether stats` report; structured JSON logs with execution IDs.
- ✅ Reproducible offline benchmarks (`aether evaluate`).
- 🟡 Resource consumption: token/cost tracked; CPU/RAM not sampled in-process.

## CLI commands (verified to exist)
`init`, `build`, `run`, `test`, `models`, `doctor`, `evaluate` (alias `eval`),
`inspect`, `benchmark`, `publish`, `pull`, `search`, `stats`, plus `--version`.

## Terminology mapping (identity term → real artifact)

| Identity term | Exists today as | Status |
|---|---|---|
| Workload Definition | `.ae.yaml` spec (`core.AetherSpec`) | ✅ |
| Aether Compiler | `internal/compiler` | ✅ |
| Execution Plan | `core.ExecutionPlan` | ✅ |
| Execution Trace | `internal/observability` records + routing dataset | ✅ |
| Execution Runtime | `internal/runtime` | ✅ |
| Intermediate Representation (IR) | — | ⬜ planned |
| Execution Graph (DAG) | — (linear steps only) | ⬜ planned |
| Operation Scheduler | — | ⬜ planned |
| Recovery Engine | — (retry/fallback only) | ⬜ planned |
| Adaptive Optimizer | — (metrics + routing feedback only) | ⬜ planned |

## Inconsistencies found during this audit (documented, not silently changed)

1. **`aether run --json` output is not cleanly machine-parseable.** Progress
   lines (`→ Loading artifact…`, `⚠ Using mock model…`, `→ Executing…`) are
   written to **stdout before** the JSON object, so `aether run … --json | jq`
   fails. Structured logs on stderr are unaffected.
   *Recommended fix:* when `--json` is set, suppress the banner/progress lines on
   stdout so stdout is a single JSON document. (Not applied here — this is a
   documentation/positioning task; filed for a follow-up code change.)

## Honest summary
Aether today is a **single-workload compiler + reliable single-operation runtime**
with strong routing, evaluation, and observability. The multi-operation
compiler/runtime (DAGs, dependency scheduling, recovery, plan optimization) is
the **architectural direction**, not current functionality, and is labelled as
such everywhere in the docs.
