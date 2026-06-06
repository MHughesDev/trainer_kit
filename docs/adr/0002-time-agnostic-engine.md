# ADR-0002: Keep the Engine Time-Agnostic; DataSource Yields Pre-Windowed Batches

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

Time series is the first target domain, but the engine is meant to stay domain-agnostic so the
same core serves later domains (artifact: "agnostic core + target a few domains"). Time-series
data needs windowing (slice a long series into input/target spans) and a strictly *temporal*
train/val split (no shuffling across the time boundary, or the model sees the future). Where this
logic lives decides whether the core stays clean or quietly accretes domain logic (FM-2), and a
leaky split would silently corrupt every consumer's results (FM-4). This resolves Q-2.

## Decision

The engine consumes plain `(input, target)` batches and **knows nothing about time**. The injected
`DataSource` is responsible for windowing and for producing the temporal train/val split. A
**separate, optional** `trainer_kit.timeseries` helper layer may provide sliding-window and
temporal-split utilities that apps can use *behind* their `DataSource` — but the engine never
depends on it.

## Rationale

Pushing windowing and splitting behind the `DataSource` keeps the core genuinely agnostic
(defends FM-2) and means adding a future domain requires no engine change. Offering an optional
helper layer keeps ergonomics reasonable (apps don't all reinvent sliding windows) without
coupling the engine to time-series concepts. The leak-prone temporal split lives in one auditable,
reusable place (the helper) rather than being re-implemented per app, reducing FM-4 exposure while
keeping the engine ignorant of it.

## Consequences

**Positive:**
- Engine remains domain-agnostic; new domains need no core changes.
- Temporal-split correctness can be tested once in the helper layer.

**Negative:**
- Apps that don't use the helper must implement windowing themselves.
- A two-layer story (agnostic engine + optional helper) is slightly more to document.

**Neutral:**
- The helper is versioned with the library but is strictly opt-in; using torch `Dataset`/`DataLoader`
  directly is fully supported.

## Alternatives Considered

### Option A: Engine owns windowing and temporal splitting
Most convenient for callers, but injects domain logic into the agnostic core (FM-2) and would need
re-litigating for every future domain. Rejected.

### Option B: DataSource yields windows, no helper at all
Keeps the core pure but forces every app to re-implement (and risk getting wrong) the temporal
split — raising FM-4 exposure. Rejected in favor of an optional shared helper.

## References

- Resolves [open-questions.md](../open-questions.md) Q-2
- Artifact: [FM-2, FM-4](../artifact.md)
- Related: ADR-0001 (DataSource protocol), ADR-0008 (leakage guard)
