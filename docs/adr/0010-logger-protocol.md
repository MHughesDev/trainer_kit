# ADR-0010: Lifecycle-Hook Logger Protocol with Structured Payloads

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

The library must surface training metrics and progress without owning any tracker or external
account (SC-4). The decision (from intake) was an injected logger protocol: the library emits
events, the app routes them to stdout / TensorBoard / MLflow / W&B / etc. This ADR fixes the
*shape* of that protocol — what events fire and what each carries. This resolves Q-10.

## Decision

Define a `Logger` protocol as **coarse lifecycle hooks with structured payloads**:
`on_run_start`, `on_epoch_start`, `on_batch_end`, `on_epoch_end`, `on_run_end`. Each hook receives a
structured record (e.g. step/epoch index, split, metrics dict, timestamp). The library ships a
trivial stdout logger; all richer destinations are app-provided implementations of the protocol.

## Rationale

Named lifecycle hooks map directly onto how trackers like TensorBoard and MLflow expect to be driven
(scalars at step/epoch boundaries), so writing an adapter is a few lines — keeping SC-4's "route
anywhere" promise cheap. Structured payloads (a metrics dict plus context) avoid string-parsing and
let adapters pick what they need. Coarse hooks are more discoverable and easier to implement than a
single generic event method, which matters because every richer destination is consumer-written. The
library depending on no tracker keeps the inject-don't-own contract (ADR-0001) intact.

## Consequences

**Positive:**
- Trivial to adapt to TensorBoard/MLflow/W&B/stdout; SC-4 satisfied with no tracker dependency.
- Structured records are self-describing and stable to consume.

**Negative:**
- Adding a genuinely new event type later is a protocol change (mitigated by ADR-0003's versioning).
- Fixed hook set may not fit an exotic logging need without a workaround.

**Neutral:**
- A no-op/stdout logger is the default when none is injected.

## Alternatives Considered

### Option A: Single `log(event)` method over a typed event union
More extensible — new events don't change the method signature — but less obvious to implement and
pushes event-type dispatch onto every adapter. Rejected for worse DX given adapters are app-written.

### Option B: Library integrates trackers directly (built-in TensorBoard/MLflow adapters)
Most convenient out of the box, but makes the library depend on (and effectively own a relationship
with) external trackers, violating SC-4's "no tracker dependency". Rejected; adapters live in apps.

## References

- Resolves [open-questions.md](../open-questions.md) Q-10
- Artifact: [SC-4](../artifact.md)
- Related: ADR-0001 (injection), ADR-0009 (metrics carried in payloads)
