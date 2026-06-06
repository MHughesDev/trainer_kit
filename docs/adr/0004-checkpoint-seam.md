# ADR-0004: Checkpointing via Caller-Owned Path Behind a Checkpointer Protocol Seam

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Project owner

## Context

The library owns no data; the single exception is opt-in cache/checkpoint files (artifact SC-3).
Checkpointing is **out of v1 scope**, but the seam must be reserved now so adding it later is an
extension, not a breaking change. The forces: persisting to the wrong place, or persisting at all
without the caller's say-so, would breach the "owns nothing" contract (FM-2). This resolves Q-4.

## Decision

When checkpointing is implemented, the **caller passes a destination** (a path or handle), and the
library writes through a small **`Checkpointer` protocol** (default implementation: `torch.save`/
`torch.load` to the caller's path). The engine never writes to an implicit or library-chosen
location. In v1 the protocol seam is reserved (defined/typed) but not exercised.

## Rationale

A caller-supplied destination keeps storage ownership with the consumer (SC-3) — the library only
writes where it is told, and only when asked. Routing writes through a `Checkpointer` protocol
rather than calling `torch.save` inline means the destination and format can later be swapped
(e.g. to object storage, or to a fully injected storage interface) without changing the public
pipeline API — the same extensibility discipline as ADR-0007. Defaulting to `torch.save` keeps the
common case trivial.

## Consequences

**Positive:**
- Storage ownership stays with the caller; "owns nothing" holds (SC-3/FM-2).
- Future storage backends are a drop-in behind the protocol, no API break.

**Negative:**
- One more protocol concept exists in the design before it is fully used.
- Callers must supply a path to get checkpoints — no zero-config default location (intentional).

**Neutral:**
- v1 ships the seam but not the behavior; first implementation lands in a later version.

## Alternatives Considered

### Option A: Caller injects a full storage interface; library never touches the filesystem
The purest "owns nothing" stance, but more for every app to wire up for the common local-file case.
Not rejected outright — the `Checkpointer` protocol is the seam that makes this possible later — but
not the default for v1.

### Option B: Library picks a default checkpoint directory automatically
Simplest for users, but the library would be choosing where to persist data, breaching SC-3/FM-2.
Rejected.

## References

- Resolves [open-questions.md](../open-questions.md) Q-4
- Artifact: [SC-3, FM-2](../artifact.md)
- Related: ADR-0007 (same seam-reservation discipline)
