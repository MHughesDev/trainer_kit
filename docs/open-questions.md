# Open Questions

A living register of questions the system has not yet resolved. This is not a scratchpad —
it is the durable record of every consequential fork the project has faced, how it was decided,
and what evidence decided it.

## When to add an entry

Add an open question **whenever you compare important parts of the system or weigh trade-offs
between variations** — choosing a data store, an API to integrate, a pattern, a boundary between
components. If a decision is worth making deliberately, the question behind it is worth recording here.

## How entries move

1. **Open** — the question is raised. Record the options on the table and what would decide between them.
2. **Researching** — a research brief is underway (link it).
3. **Resolved** — the question is answered. Record the answer, the evidence that settled it, and the
   ADR or spec that now carries the decision.

Never delete a resolved question. The point of this file is that six months from now, anyone can see
not just *what* was decided but *why*, and which alternatives were already ruled out.

---

## Register

| ID | Question | Status | Options Weighed | Resolution | Evidence / Links |
|----|----------|--------|-----------------|------------|------------------|
| Q-1 | What is the minimal set of injected interfaces, and the exact method signature of each (`ModelProvider`, `DataSource`, optimizer/scheduler/loss factories, `Metric`, `Logger`)? | Open | Few coarse protocols vs. many fine-grained ones; ABCs vs. `typing.Protocol`; required vs. optional methods | — | Crux of [FM-1](./artifact.md); enables [SC-1](./artifact.md). First research brief candidate. See detail below. |
| Q-2 | How do time-series specifics (windowing, forecast horizon, temporal train/val split) divide between the domain-agnostic engine and the injected `DataSource`? | Open | Engine owns windowing/splitting vs. `DataSource` yields ready windows vs. a thin time-series helper layer | — | Determines how "agnostic core" stays true; raised in [artifact](./artifact.md). Second research brief candidate. See detail below. |
| Q-3 | What versioning and compatibility policy governs the injected interfaces (semver line, deprecation path, support window)? | Open | Strict SemVer + deprecation cycle vs. 0.x rapid iteration vs. pinned major with long support | — | Directly defends against [FM-3](./artifact.md); supports [SC-2](./artifact.md). |
| Q-4 | What is the checkpoint / cache contract — who owns the path, the on-disk format, and resume semantics? | Open | Caller passes a path/handle vs. injected storage interface; native `torch.save` vs. format-agnostic blob | — | Seam must be reserved in v1 even though checkpointing is out of scope. Relates to [SC-3](./artifact.md) / [FM-2](./artifact.md). |
| Q-5 | Is the package distributed via public PyPI or a private index? | Open | Public PyPI vs. private/internal index vs. git/VCS install | — | `[unknown — revisit]` in [artifact](./artifact.md); affects how SC-2 multi-app reuse is delivered. |
| Q-6 | What Python and PyTorch version matrix must be supported? | Open | Single pinned torch vs. supported range; min Python (3.10/3.11/3.12) | — | `[unknown — revisit]` in [artifact](./artifact.md); constrains every API choice. |
| Q-7 | How is the engine structured so single-device → multi-GPU/DDP is an extension, not a rewrite (the device/strategy seam)? | Open | Device-strategy abstraction now vs. internal seam only vs. defer entirely | — | Guards [SC-5](./artifact.md) against [FM-5](./artifact.md); design must reserve the seam in v1. |
| Q-8 | What is the correctness strategy for the shared loop — determinism/seeding contract, validation-isolation guarantees, and the test approach that proves them? | Open | Caller-owned seeding vs. library-managed; golden-run tests vs. property tests vs. reference-impl parity | — | A single bug here corrupts every consumer ([FM-4](./artifact.md)). |
| Q-9 | What is the `Metric` interface and the metric-aggregation semantics (per-batch vs. per-epoch, reduction, multi-metric reporting)? | Open | Stateful accumulator objects vs. pure functions; library-provided defaults vs. fully injected | — | Underpins how results are reported; relates to [SC-1](./artifact.md). |
| Q-10 | What is the `Logger` protocol shape and event vocabulary (what events fire, with what payloads, at what granularity)? | Open | Fine-grained event stream vs. coarse start/step/end hooks; structured records vs. free-form | — | Implements the injected-logger decision behind [SC-4](./artifact.md). |
| Q-11 | How does a caller configure and invoke a run (epochs, lr, device, etc.) — the public entry-point shape? | Open | Plain kwargs vs. a typed config/dataclass vs. builder; single `fit()` entry vs. staged API | — | Defines the SC-1 "single pipeline entry point"; affects ergonomics weighed in [FM-1](./artifact.md). |

> Number questions sequentially: `Q-1`, `Q-2`, … Append new ones at the end; never renumber.

---

## Entry Detail (optional)

For questions that need more than a table row, add a section below. Keep the table as the index.

### Q-1: What is the minimal injected-interface set and each method signature?

**Status:** Open
**Raised:** 2026-06-06

**Why it matters:** Interface injection is the entire integration contract — it is how a consuming
app hands its model and data to a pipeline without the library owning either ([SC-1](./artifact.md)).
If the interfaces are too generic, apps write as much glue as a hand-rolled loop and the reuse value
collapses ([FM-1](./artifact.md)); if too prescriptive, they fail to fit a second app
([SC-2](./artifact.md)). This question must be resolved before any spec or code.

**Options weighed:**
- A few coarse protocols (e.g. one `Pipeline` + `ModelProvider` + `DataSource`) — simplest surface,
  but may hide too much or too little.
- Many fine-grained protocols (separate optimizer, scheduler, loss, metric, logger factories) —
  composable and explicit, but more for each app to implement.
- `abc.ABC` (nominal, explicit inheritance) vs. `typing.Protocol` (structural, duck-typed) for the
  contracts — affects coupling and how strictly conformance is enforced.

**What would decide it:** A research brief surveying prevailing PyTorch training-library interfaces
(Lightning's `LightningModule`/`DataModule`, Ignite engine, skorch, plain-torch patterns), extracting
the minimal seam that fits time-series first and generalizes later. Resolve before the first spec.

**Resolution:** [Filled in when resolved — the answer, and the research/ADR that carries it.]

### Q-2: How do time-series specifics divide between the engine and the injected DataSource?

**Status:** Open
**Raised:** 2026-06-06

**Why it matters:** The engine is meant to be domain-agnostic, with time-series as the first concrete
target. Where windowing, forecast horizon, and the *temporal* (non-shuffled, leak-free) train/val
split live determines whether the core stays clean or quietly accretes domain logic
([FM-2](./artifact.md)) — and whether a leak-prone split silently corrupts results
([FM-4](./artifact.md)).

**Options weighed:**
- Engine owns windowing and temporal splitting — convenient for callers, but pushes domain logic into
  the agnostic core.
- `DataSource` yields ready-made windows/splits — keeps the core agnostic, but each app reimplements
  windowing.
- A thin, optional time-series helper layer beside the core that apps may use behind the `DataSource`
  interface — middle ground.

**What would decide it:** A research brief on how time-series windowing/splitting is conventionally
factored (e.g. sliding-window datasets, `TimeSeriesSplit`-style validation), and which split of
responsibility preserves both SC-1 ergonomics and the agnostic-core constraint.

**Resolution:** [Filled in when resolved — the answer, and the research/ADR that carries it.]
