# Architecture

The current-state map of the system. This file starts empty and is filled in **only as real
decisions are made** — never speculatively. It does not propose an architecture; it records the
one that has actually emerged from the specs and ADRs.

- It is **not** an ADR. ADRs capture point-in-time decisions and their rationale.
- It is **not** a spec. Specs define individual components and contracts.
- It **is** the single answer to "what does the system look like right now, and how do the pieces fit?"

Update this file whenever an ADR changes the shape of the system or a new component spec is approved.
If a section has no decided answer yet, leave it marked `[not yet decided]` — do not guess.

---

## 1. Overview

`trainer_kit` is a raw-PyTorch **library** (no service, no runtime of its own) that provides
callable end-to-end train/validation pipelines. It owns no models and no data: a consuming
application implements two small `typing.Protocol` contracts and injects everything else, then calls
a single typed entry point. A time-agnostic engine drives the loop; time-series concerns live behind
the injected data source and an optional helper layer. It is consumed as a versioned dependency by
many independent applications, each running its own process. (Derived from ADR-0001…0011.)

## 2. Components

| Component | Responsibility | Spec |
|-----------|----------------|------|
| `ModelProvider` (protocol) | Caller-implemented: `build() -> nn.Module` — supplies the model the library never owns | `[spec pending]` (ADR-0001) |
| `DataSource` (protocol) | Caller-implemented: `train_loader()` / `val_loader()` — supplies data + owns windowing and temporal split | `[spec pending]` (ADR-0001, ADR-0002) |
| Training engine | Time-agnostic train/validation loop over plain `(input, target)` batches; owns the device/placement seam and seeded, leakage-guarded execution | `[spec pending]` (ADR-0002, ADR-0007, ADR-0008) |
| `fit()` / `evaluate()` entry point | Single public API: `fit(model_provider, data_source, config)` → result; `config` is a typed `TrainConfig` | `[spec pending]` (ADR-0011) |
| `Metric` (protocol) | Stateful accumulator (`update`/`compute`/`reset`); minimal time-series defaults; torchmetrics-compatible | `[spec pending]` (ADR-0009) |
| `Logger` (protocol) | Coarse lifecycle hooks with structured payloads; routes metrics/progress to caller-chosen destinations | `[spec pending]` (ADR-0010) |
| `Checkpointer` (protocol, seam only in v1) | Caller-owned destination; default `torch.save`/`torch.load`; reserved, not exercised in v1 | `[spec pending]` (ADR-0004) |
| `trainer_kit.timeseries` (optional helper) | Opt-in sliding-window and temporal-split utilities used behind a `DataSource`; engine never depends on it | `[spec pending]` (ADR-0002) |

## 3. Data Flow

```
Consuming application
  │  implements ModelProvider, DataSource
  │  constructs optimizer/loss/metrics/logger + TrainConfig
  ▼
fit(model_provider, data_source, config)         [ADR-0011]
  │
  ├─ engine seeds RNGs (if seed in config)        [ADR-0008]
  ├─ model = model_provider.build()               [ADR-0001]
  ├─ for each epoch:
  │     ├─ train over data_source.train_loader()  (batches already windowed) [ADR-0002]
  │     │     └─ device seam places tensors        [ADR-0007]
  │     ├─ validate over data_source.val_loader() (leakage-guarded)          [ADR-0008]
  │     ├─ metrics: update()/compute()/reset()     [ADR-0009]
  │     └─ logger.on_epoch_end(record)             [ADR-0010]
  │
  ▼
result object (final metrics, history)  →  caller routes/persists as it wishes
```

Data and model weights never leave the caller's ownership; the only disk write is opt-in
checkpointing to a caller-supplied path (ADR-0004), not present in v1.

## 4. External Dependencies

| Dependency | Used for | Justified by |
|------------|----------|--------------|
| PyTorch (`torch`, tested range `>=2.1,<3`) | Tensor ops, autograd, the underlying training primitives | [ADR-0006](./adr/0006-version-matrix.md) |
| Python ≥ 3.10 | Modern typing for the `Protocol`-based contract | [ADR-0006](./adr/0006-version-matrix.md) |
| (optional) `torchmetrics` | Drop-in metrics via the `Metric` protocol; not required | [ADR-0009](./adr/0009-stateful-metrics.md) |

No datasets, model weights, or experiment-tracker SDKs are dependencies — those are injected by the
consuming application (ADR-0001, ADR-0010).

## 5. Key Decisions

Newest first — a map into `adr/`, not a copy of it:

- [ADR-0011](./adr/0011-entry-point.md) — Single typed `fit()` entry point with `TrainConfig`
- [ADR-0010](./adr/0010-logger-protocol.md) — Lifecycle-hook `Logger` protocol with structured payloads
- [ADR-0009](./adr/0009-stateful-metrics.md) — Stateful accumulator `Metric` protocol
- [ADR-0008](./adr/0008-correctness-strategy.md) — Library-managed seeding, golden-run tests, leakage guard
- [ADR-0007](./adr/0007-device-seam.md) — Internal device/placement seam for future multi-GPU
- [ADR-0006](./adr/0006-version-matrix.md) — Python ≥ 3.10 and a tested PyTorch range
- [ADR-0005](./adr/0005-distribution.md) — VCS pre-1.0, PyPI at 1.0
- [ADR-0004](./adr/0004-checkpoint-seam.md) — Checkpointing behind a caller-owned `Checkpointer` seam
- [ADR-0003](./adr/0003-versioning-policy.md) — 0.x now, SemVer + deprecation cycle at 1.0
- [ADR-0002](./adr/0002-time-agnostic-engine.md) — Time-agnostic engine; `DataSource` yields windows
- [ADR-0001](./adr/0001-inject-dependencies-via-protocols.md) — Inject dependencies via typing Protocols

## 6. Known Constraints & Boundaries

- **Owns nothing.** No datasets or model weights in the repo or package; the only disk write is
  opt-in checkpointing to a caller-supplied path (artifact SC-3, ADR-0004).
- **Library, not a service.** No daemon, queue, or multi-tenant runtime; consumers run pipelines in
  their own processes (artifact non-goals).
- **v1 scope:** train/val loop + metrics only. Out of v1: distributed/multi-GPU, AMP, HPO,
  checkpointing behavior (seam only), built-in tracker adapters, domains beyond time series
  (artifact non-goals).
- **Stable public API.** Device handling and checkpoint destination are internal seams so multi-GPU
  and storage backends can land without breaking `fit()` (ADR-0007, ADR-0004, SC-5).
