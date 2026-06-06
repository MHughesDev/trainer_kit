# The Artifact

Fill this in before anything else. It does not need to be complete — it needs to be honest.
The goal is to get the most important things out of your head and into a form that can be designed against.

Each section feeds a downstream artifact: research briefs, plans, specs, or ADRs.

---

## What Are We Building?

`trainer_kit` — a PyTorch library that provides callable, end-to-end training and validation
pipelines for AI models. It owns no models and no training data; consuming applications inject
those through typed Python interfaces, and the library runs the train/validate loop on their
behalf. The first target domain is time-series models, built on a domain-agnostic engine
underneath.

---

## Who Uses It?

The primary users are other Python applications (and the developers who build them) that need to
train models without re-implementing training loops. `trainer_kit` is consumed as a `pip`
dependency — each application imports it, implements its interfaces, and drives the pipeline in
its own process. The secondary user is the author, reusing one trusted pipeline implementation
across multiple future, unrelated projects.

---

## What Problem Does It Actually Solve?

Re-writing train/validate loops, metric wiring, and run mechanics for every new project is
repetitive, error-prone, and drifts in quality from project to project. There is no single
trusted implementation to reuse. `trainer_kit` centralizes the training *mechanics* behind stable
interfaces so each new application supplies only what is genuinely unique to it — its model, its
data, its loss, its metrics — and inherits a correct, tested pipeline for everything else. Without
it, every application reinvents the loop and the same class of bugs and inconsistencies
multiplies across the portfolio.

---

## What Does Good Look Like?

- SC-1: A consuming application can train and validate a time-series model end-to-end by
  implementing the library's model and data-source interfaces and calling a single pipeline entry
  point — without the library importing, defining, or storing any dataset or model.
- SC-2: The same installed version of `trainer_kit` is used, unchanged, by at least two distinct
  applications — each with its own model and data — with no modification to the library.
- SC-3: The library holds no training data and no model weights in its own repository or package.
  The only data it persists is opt-in local cache/checkpoint files written to a path the caller
  controls.
- SC-4: Training emits structured metrics and progress through an injected logger interface, so a
  caller can route them to any destination (stdout, TensorBoard, MLflow, …) without the library
  depending on any specific tracker.
- SC-5: Replacing the single-device runtime with a richer one (e.g. multi-GPU / DDP) later
  requires no breaking change to the public pipeline interface that consuming apps depend on.

---

## What Would Make This Fail?

- FM-1: Abstraction leak — the interfaces are so generic that apps end up writing as much glue as
  a hand-rolled loop would have needed, eliminating the reuse value. — product
- FM-2: Hidden ownership — the library accidentally couples to a specific dataset or model shape,
  or persists data it shouldn't, breaking the "owns nothing" contract. — technical
- FM-3: API instability — frequent breaking changes to the injected interfaces force every
  consuming app to rewrite its integration, defeating the "one trusted dependency" goal. —
  operational
- FM-4: Correctness gap — a subtle bug in the shared loop (validation leakage, metric averaging,
  seeding/determinism) silently corrupts results for *every* consumer at once. — technical
- FM-5: Single-device assumptions bake into the core, turning the promised multi-GPU extension
  into a rewrite rather than an extension (directly threatens SC-5). — technical

---

## What Don't We Know Yet?

- The minimal set of injected interfaces and their exact method signatures — candidates:
  `ModelProvider`, `DataSource`, optimizer/scheduler/loss factories, `Metric`, `Logger`.
- How time-series specifics (windowing, forecast horizon, temporal train/val split) map onto a
  domain-agnostic core — what stays in the engine vs. what lives behind the injected data source.
- Versioning and compatibility strategy for the stable interfaces (semver policy, deprecation
  path) — load-bearing for FM-3.
- The checkpoint/cache contract: who owns paths, file format, and resume semantics. Checkpointing
  is out of v1 scope, but the seam must be reserved now (relates to SC-3 / FM-2).
- Packaging and distribution: public vs. private PyPI, and the supported Python / PyTorch version
  matrix.

---

## What Are We Deliberately Not Building?

- Not owning, bundling, downloading, or versioning any dataset or model weights — ever.
- Not a training service, daemon, or API. It is a library dependency; no job queue, scheduler, or
  multi-tenant runtime. (Apps may run many pipelines concurrently in *their own* processes.)
- Out of v1 scope (seams may be reserved, not implemented): distributed / multi-GPU, mixed
  precision (AMP), hyperparameter tuning, checkpointing / early stopping, and built-in
  experiment-tracker adapters.
- No domain logic beyond time series for now — vision, NLP, and tabular are deferred.
- No data preprocessing or feature engineering owned by the library — that lives in the
  application, behind the `DataSource` interface.

---

## What Are We Working Within?

- Built on **raw PyTorch** (torch primitives), with minimal additional dependencies.
- A pure Python library consumed via `pip` by multiple independent applications; each runs its own
  training process. No shared runtime between consumers.
- **Single-device** (CPU / single GPU) runtime for v1, deliberately designed for later multi-GPU
  extension without breaking the public API.
- **Interface-injection** architecture (typed ABCs / `typing.Protocol`) is the integration
  contract — this is how consumers hand their model and data to a pipeline.
- [unknown — revisit] Team size, timeline, budget, target Python / PyTorch version matrix, and
  public vs. private package distribution.
