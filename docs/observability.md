# Execution Observability

Pillar E. What Aether records about execution.

## Existing functionality

### Execution traces
- **Run records** (`internal/observability`): per-run model, tokens, cost,
  latency, success, cache status, retries, error, timestamp — persisted to
  `runs.json` under the data directory.
- **Routing dataset** (`routing.jsonl`): task type, difficulty, risk, strategy,
  chosen model, outcome — appended per run as JSON Lines for later analysis.

### Metrics
- Operation latency including **P50/P90/P99** percentiles and max.
- Success rate, retries, failures, cache hit rate, cost and token totals.
- `aether stats` renders a summary report.

### Structured logging (`internal/logging`)
- JSON (or text) logs to **stderr** via the standard library `log/slog`.
- Level from `AETHER_LOG_LEVEL` (debug|info|warn|error, default `warn` — quiet by
  default so stdout/`--json` output is never polluted); format from
  `AETHER_LOG_FORMAT`.
- Each run is tagged with an **execution ID** (request tracing); the ID is also
  written to the result metadata. Lifecycle events: `run.start`, `run.complete`,
  `run.failed`.

### Reproducible benchmarks
`aether evaluate` produces deterministic, machine-readable results (see
[optimization](optimization.md) and the README benchmarks section).

## Partial / not yet implemented

- 🟡 **Resource consumption**: tokens and cost are tracked; **in-process CPU/RAM
  sampling** is not. (External profiling works; Aether does not self-report it.)
- ⬜ OpenTelemetry export / distributed tracing across operations (relevant once
  the multi-operation runtime exists).

## Example

```bash
AETHER_LOG_LEVEL=debug aether run agent.ae.yaml --mock --message "hi" --json
# stderr: {"level":"INFO","msg":"run.complete","execution_id":"…","model":"mock","latency_ms":…}
# stdout: human-readable progress lines FOLLOWED BY the JSON result object
#         (see the known inconsistency below — the JSON is not yet isolated on stdout)
aether stats   # summary report with P50/P90/P99
```

> **Known inconsistency (documented, not silently patched):** with `--json`,
> `aether run` currently writes progress lines (`→ Loading artifact…`,
> `⚠ Using mock model…`, `→ Executing…`) to **stdout before** the JSON object, so
> piping stdout straight into `jq` fails. The structured logs (stderr) are clean.
> Recommended fix (tracked in the inventory): suppress the banner/progress lines
> on stdout when `--json` is set, leaving stdout as a single JSON document.
