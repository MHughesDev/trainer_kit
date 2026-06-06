# ADR-0011: Single Typed `fit()` Entry Point with a TrainConfig Dataclass

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

SC-1 promises "a single pipeline entry point." This ADR fixes what that call literally looks like:
how an app configures and starts a run. The forces: discoverability and type-safety vs. low
ceremony, and keeping the front door stable as options grow (a clunky front door makes apps feel the
FM-1 "too much glue" pain even if internals are good). This resolves Q-11.

## Decision

Expose a single entry point: `fit(model_provider, data_source, config)` returning a result object,
where `config` is a **typed dataclass `TrainConfig`** (epochs, lr, device, seed, etc.). Provide a
matching `evaluate(...)` for validation-only runs. The signature stays **device-agnostic** (per
ADR-0007) — `device` in the config selects placement without changing the API shape.

## Rationale

A single `fit()` is the literal realization of SC-1's "one entry point." A typed `TrainConfig`
dataclass is self-documenting and IDE-completable, and it absorbs new options as fields without
changing the call signature — so the front door stays stable as the library grows, supporting the
ADR-0003 stability goal and reducing the FM-1 friction consumers feel first. Returning a structured
result object (final metrics, history) gives callers their outcome without the library having to log
or persist anything (consistent with ADR-0010 and the owns-nothing stance).

## Consequences

**Positive:**
- Clear, discoverable, type-checked front door directly satisfying SC-1.
- New options are added as config fields without breaking the signature.

**Negative:**
- Tiny scripts pay slight ceremony (construct a `TrainConfig`) vs. loose kwargs.
- A growing config dataclass needs sensible defaults and documentation to stay approachable.

**Neutral:**
- `fit()` and `evaluate()` share the same provider/data/config inputs for symmetry.

## Alternatives Considered

### Option A: Loose `**kwargs` on `fit(...)`
Lowest ceremony for quick scripts, but no type-safety or IDE discoverability, and it ages badly as
options multiply — exactly the glue/friction that feeds FM-1. Rejected.

### Option B: Builder / staged API (`prepare()` then `run()`)
More flexible for advanced flows, but adds surface and contradicts the "single entry point" framing
of SC-1. Rejected for v1; can be layered on later without breaking `fit()`.

## References

- Resolves [open-questions.md](../open-questions.md) Q-11
- Artifact: [SC-1, FM-1](../artifact.md)
- Related: ADR-0001 (injected inputs), ADR-0007 (device-agnostic), ADR-0010 (result vs. logging)
