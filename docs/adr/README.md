# Architecture Decision Records (ADRs)

This folder contains the project's Architecture Decision Records — a chronological log of significant technical decisions, the context that drove them, and the consequences of each choice.

> To add a new ADR, follow the skill: `skills/create-adr.md` and procedure: `procedures/add-adr.md`.

---

## What Is an ADR?

An ADR documents a single architectural decision. It captures:
- The **context** (forces, constraints, options considered)
- The **decision** that was made
- The **status** of the decision (proposed, accepted, deprecated, superseded)
- The **consequences** (what becomes easier, harder, or different)

ADRs are immutable historical records. When a decision changes, a new ADR is created that supersedes the old one — the old ADR is updated only to reflect its new status.

---

## Index

| # | Title | Status | Date |
|---|-------|--------|------|
| 0001 | [Inject Dependencies via Typing Protocols](./0001-inject-dependencies-via-protocols.md) | Accepted | 2026-06-06 |
| 0002 | [Keep the Engine Time-Agnostic; DataSource Yields Pre-Windowed Batches](./0002-time-agnostic-engine.md) | Accepted | 2026-06-06 |
| 0003 | [Start in 0.x, Commit to SemVer at 1.0 with a Deprecation Cycle](./0003-versioning-policy.md) | Accepted | 2026-06-06 |
| 0004 | [Checkpointing via Caller-Owned Path Behind a Checkpointer Protocol Seam](./0004-checkpoint-seam.md) | Accepted | 2026-06-06 |
| 0005 | [Distribute via VCS Pre-1.0, PyPI at 1.0](./0005-distribution.md) | Accepted | 2026-06-06 |
| 0006 | [Support Python ≥ 3.10 and a Tested PyTorch Range](./0006-version-matrix.md) | Accepted | 2026-06-06 |
| 0007 | [Reserve an Internal Device/Placement Seam for Future Multi-GPU](./0007-device-seam.md) | Accepted | 2026-06-06 |
| 0008 | [Library-Managed Seeding, Golden-Run Tests, and a Leakage Guard](./0008-correctness-strategy.md) | Accepted | 2026-06-06 |
| 0009 | [Stateful Accumulator Metric Protocol](./0009-stateful-metrics.md) | Accepted | 2026-06-06 |
| 0010 | [Lifecycle-Hook Logger Protocol with Structured Payloads](./0010-logger-protocol.md) | Accepted | 2026-06-06 |
| 0011 | [Single Typed `fit()` Entry Point with a TrainConfig Dataclass](./0011-entry-point.md) | Accepted | 2026-06-06 |

---

## Naming Convention

```
adr/NNNN-short-descriptive-title.md
```

Examples:
- `adr/0001-use-postgresql-for-primary-datastore.md`
- `adr/0002-adopt-event-driven-architecture.md`
- `adr/0003-replace-rest-with-graphql.md`

---

## Status Values

| Status | Meaning |
|--------|---------|
| `Proposed` | Under discussion, not yet accepted |
| `Accepted` | Decision is in effect |
| `Deprecated` | No longer recommended but not replaced |
| `Superseded by ADR-NNNN` | Replaced by a newer decision |

---

## ADR Template

```markdown
# ADR-NNNN: [Title]

**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX
**Date:** YYYY-MM-DD
**Deciders:** [names or roles]

## Context

[What forces, constraints, or requirements led to this decision? What options were considered?]

## Decision

[What was decided? State it clearly and directly.]

## Rationale

[Why was this option chosen over the alternatives? What trade-offs were accepted?]

## Consequences

**Positive:**
- [What becomes easier or better?]

**Negative:**
- [What becomes harder or worse?]

**Neutral:**
- [What changes without clear valence?]

## Alternatives Considered

### Option A: [Name]
[Brief description and why it was not chosen]

### Option B: [Name]
[Brief description and why it was not chosen]

## References

- [Links to relevant research, specs, or external resources]
```
