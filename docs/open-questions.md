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
| Q-1 | What is the minimal set of injected interfaces, and the exact method signature of each (`ModelProvider`, `DataSource`, optimizer/scheduler/loss factories, `Metric`, `Logger`)? | Resolved | Few coarse protocols vs. many fine-grained ones; ABCs vs. `typing.Protocol`; required vs. optional methods | Two required `typing.Protocol` contracts (`ModelProvider`, `DataSource`); everything else injected as objects/factories | [ADR-0001](./adr/0001-inject-dependencies-via-protocols.md) |
| Q-2 | How do time-series specifics (windowing, forecast horizon, temporal train/val split) divide between the domain-agnostic engine and the injected `DataSource`? | Resolved | Engine owns windowing/splitting vs. `DataSource` yields ready windows vs. a thin time-series helper layer | Engine stays time-agnostic; `DataSource` yields pre-windowed batches; optional `timeseries` helper layer | [ADR-0002](./adr/0002-time-agnostic-engine.md) |
| Q-3 | What versioning and compatibility policy governs the injected interfaces (semver line, deprecation path, support window)? | Resolved | Strict SemVer + deprecation cycle vs. 0.x rapid iteration vs. pinned major with long support | 0.x while interfaces settle; strict SemVer + one-minor deprecation cycle from 1.0 | [ADR-0003](./adr/0003-versioning-policy.md) |
| Q-4 | What is the checkpoint / cache contract — who owns the path, the on-disk format, and resume semantics? | Resolved | Caller passes a path/handle vs. injected storage interface; native `torch.save` vs. format-agnostic blob | Caller-owned path behind a `Checkpointer` protocol seam (default `torch.save`); seam reserved in v1 | [ADR-0004](./adr/0004-checkpoint-seam.md) |
| Q-5 | Is the package distributed via public PyPI or a private index? | Resolved | Public PyPI vs. private/internal index vs. git/VCS install | VCS/git (pinned ref) pre-1.0; PyPI at 1.0 (public-vs-private deferred to 1.0) | [ADR-0005](./adr/0005-distribution.md) |
| Q-6 | What Python and PyTorch version matrix must be supported? | Resolved | Single pinned torch vs. supported range; min Python (3.10/3.11/3.12) | Python ≥ 3.10; PyTorch as a CI-tested range (floor `>=2.1,<3`) | [ADR-0006](./adr/0006-version-matrix.md) |
| Q-7 | How is the engine structured so single-device → multi-GPU/DDP is an extension, not a rewrite (the device/strategy seam)? | Resolved | Device-strategy abstraction now vs. internal seam only vs. defer entirely | Internal device/placement seam in v1; no public strategy API; `fit()` stays device-agnostic | [ADR-0007](./adr/0007-device-seam.md) |
| Q-8 | What is the correctness strategy for the shared loop — determinism/seeding contract, validation-isolation guarantees, and the test approach that proves them? | Resolved | Caller-owned seeding vs. library-managed; golden-run tests vs. property tests vs. reference-impl parity | Library-managed (opt-in) seeding + golden-run regression tests + train/val leakage guard | [ADR-0008](./adr/0008-correctness-strategy.md) |
| Q-9 | What is the `Metric` interface and the metric-aggregation semantics (per-batch vs. per-epoch, reduction, multi-metric reporting)? | Resolved | Stateful accumulator objects vs. pure functions; library-provided defaults vs. fully injected | Stateful accumulator protocol (`update`/`compute`/`reset`); minimal time-series defaults; torchmetrics-compatible | [ADR-0009](./adr/0009-stateful-metrics.md) |
| Q-10 | What is the `Logger` protocol shape and event vocabulary (what events fire, with what payloads, at what granularity)? | Resolved | Fine-grained event stream vs. coarse start/step/end hooks; structured records vs. free-form | Coarse lifecycle hooks (`on_run_start`…`on_run_end`) with structured payloads; stdout default | [ADR-0010](./adr/0010-logger-protocol.md) |
| Q-11 | How does a caller configure and invoke a run (epochs, lr, device, etc.) — the public entry-point shape? | Resolved | Plain kwargs vs. a typed config/dataclass vs. builder; single `fit()` entry vs. staged API | Single `fit(model_provider, data_source, config)` returning a result; `config` a typed `TrainConfig`; matching `evaluate()` | [ADR-0011](./adr/0011-entry-point.md) |

> Number questions sequentially: `Q-1`, `Q-2`, … Append new ones at the end; never renumber.

---

## Entry Detail (optional)

For questions that need more than a table row, add a section below. Keep the table as the index.

### Q-1: What is the minimal injected-interface set and each method signature?

**Status:** Resolved ([ADR-0001](./adr/0001-inject-dependencies-via-protocols.md))
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

**Resolution:** Two required `typing.Protocol` contracts — `ModelProvider` (`build() -> nn.Module`)
and `DataSource` (`train_loader()`, `val_loader()`). Optimizer, scheduler, loss, metrics, and logger
are injected as plain objects/factories, not mandatory protocols. Structural typing (`Protocol`) over
nominal (`ABC`) to minimize coupling. See [ADR-0001](./adr/0001-inject-dependencies-via-protocols.md).

### Q-2: How do time-series specifics divide between the engine and the injected DataSource?

**Status:** Resolved ([ADR-0002](./adr/0002-time-agnostic-engine.md))
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

**Resolution:** The engine consumes plain `(input, target)` batches and knows nothing about time.
The injected `DataSource` owns windowing and the temporal train/val split. An optional
`trainer_kit.timeseries` helper layer provides sliding-window and temporal-split utilities apps may
use behind their `DataSource`, but the engine never depends on it. See
[ADR-0002](./adr/0002-time-agnostic-engine.md).
