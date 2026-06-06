# ADR-0009: Stateful Accumulator Metric Protocol

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

The pipeline reports metrics (loss, MAE, RMSE, …). Metrics are computed per batch but must be
correctly aggregated to a single per-epoch figure — and a naive average of per-batch averages is
wrong when batches differ in size. We must decide whether metrics are stateful accumulators or pure
functions, and whether the library ships defaults or relies entirely on injection. This resolves
Q-9 and supports SC-1 (results reaching the caller correctly).

## Decision

Define a **stateful accumulator `Metric` protocol** with `update(preds, targets)` / `compute()` /
`reset()`. The library ships a few defaults relevant to time series (MAE, RMSE, MSE); apps may
override or inject their own. The protocol is intentionally compatible in spirit with
`torchmetrics`, so apps can plug those in directly.

## Rationale

Stateful accumulators compute size-weighted epoch aggregates correctly and cheaply — they tally
running state and reduce once at `compute()`, avoiding both the wrong-average bug and the memory cost
of buffering all predictions. Shipping a few time-series defaults makes the common case zero-effort
while injection keeps the library from owning domain-specific metric choices. Mirroring the
`torchmetrics` shape means a large existing metric ecosystem works without an adapter, reinforcing
the inject-don't-own stance of ADR-0001.

## Consequences

**Positive:**
- Correct, memory-efficient epoch aggregation by construction (supports SC-1, guards FM-4).
- `torchmetrics` and custom metrics drop in via the same protocol.

**Negative:**
- Slightly more to implement for a custom metric (three methods vs. one function).
- State means metrics must be `reset()` between epochs — a contract the engine must honor.

**Neutral:**
- Built-in defaults are minimal and time-series-oriented; broader metric sets come from injection.

## Alternatives Considered

### Option A: Pure functions over collected predictions/targets
Simpler mental model, but forces buffering every batch's outputs in memory and makes correct
size-weighted aggregation the caller's problem. Rejected on memory and correctness grounds.

### Option B: Library owns a large fixed metric catalog
Convenient, but pulls domain-specific choices into the library and bloats it against the
inject-don't-own principle. Rejected; defaults are kept minimal.

## References

- Resolves [open-questions.md](../open-questions.md) Q-9
- Artifact: [SC-1, FM-4](../artifact.md)
- Related: ADR-0001 (injection), ADR-0010 (metrics surfaced via logger)
