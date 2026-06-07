# trainer_kit v1 Implementation Plan

**Date:** 2026-06-07
**Type:** Formal
**Author:** Agent
**Status:** Draft
**Derivation Status:** Current

## Goal

A pip-installable trainer_kit library that passes all acceptance criteria in specs COMP-001 through COMP-005, FEAT-001, and DATA-001 — specifically: a consumer can train and validate a model end-to-end by implementing ModelProvider and DataSource and calling fit(), without the library owning any model or data (artifact SC-1 through SC-5).

## Derived From

- Artifact: [docs/artifact.md](../artifact.md) — SC-1 through SC-5, FM-1 through FM-5
- Architecture: [docs/architecture.md](../architecture.md) — components and build order (§2, §3)
- Specs: [SYS-001](../specs/SYS-001-system-overview.md), [COMP-001](../specs/COMP-001-package-structure.md), [COMP-002](../specs/COMP-002-protocol-definitions.md), [DATA-001](../specs/DATA-001-config-and-result-types.md), [COMP-003](../specs/COMP-003-training-engine.md), [FEAT-001](../specs/FEAT-001-fit-and-evaluate.md), [COMP-004](../specs/COMP-004-timeseries-helpers.md), [COMP-005](../specs/COMP-005-test-suite.md)
- ADRs: [ADR-0001](../adr/0001-inject-dependencies-via-protocols.md) through [ADR-0011](../adr/0011-entry-point.md)

## Scope

**In scope:**
- Package scaffold (pyproject.toml, module hierarchy) — COMP-001
- All five protocol definitions and StdoutLogger — COMP-002
- TrainConfig, EvalConfig, EpochRecord, Result dataclasses — DATA-001
- Training engine: loop, device seam, seeding, leakage guard — COMP-003
- fit() and evaluate() public entry points — FEAT-001
- trainer_kit.timeseries: SlidingWindowDataset, temporal_split, MAE/MSE/RMSE — COMP-004
- Full test suite: golden-run, determinism, leakage guard, unit tests — COMP-005

**Out of scope (v1):**
- Distributed / multi-GPU training (device seam is reserved, not implemented beyond single-device)
- Checkpointing behavior (Checkpointer seam is defined but the engine does not call it)
- Mixed-precision (AMP), HPO, early stopping, learning-rate scheduling
- Built-in experiment-tracker adapters (TensorBoard, MLflow, W&B)
- PyPI publication (pre-1.0 distribution is VCS-pinned per ADR-0005)

## Dependencies

- Python ≥ 3.10 and PyTorch ≥ 2.1 must be available in the development environment.
- All specs (COMP-001 through COMP-005, DATA-001, FEAT-001) must remain at `Draft` or above — do not execute against a spec marked `Superseded` without re-deriving this plan.
- No gating open questions remain (Q-1 through Q-11 all resolved).

## Risks

- Risk: golden-run expected values (COMP-005 §4.3) cannot be populated until the engine is working → Mitigation: populate them during Milestone 5 as the final step before marking it complete.
- Risk: metric name collision when two metrics of the same class are used (DATA-001 §7.2.B) → Mitigation: document in TrainConfig docstring; enforce uniqueness with a validation check in __post_init__ if two metrics share __class__.__name__.
- Risk: FM-4 (silent loop-math bug) → Mitigation: golden-run and determinism tests in Milestone 6 are the detection mechanism; run them before marking any engine change complete.
- Risk: device seam (COMP-003 §4.2) not isolated properly → Mitigation: CI static analysis (grep for `.cuda()` outside _device.py) as part of Milestone 4 acceptance.

## Milestones

| # | Milestone | Source | Success Signal |
|---|-----------|--------|----------------|
| 1 | Package scaffold | COMP-001, artifact SC-1 | `pip install -e .` succeeds; `from trainer_kit import fit` works |
| 2 | Protocol definitions and StdoutLogger | COMP-002, artifact SC-1, SC-4 | All isinstance() checks pass; StdoutLogger prints correct format |
| 3 | Config and result types | DATA-001, ADR-0011 | TrainConfig validation raises on bad input; Result fields are correct |
| 4 | Training engine | COMP-003, artifact SC-5, FM-4, FM-5 | Engine runs a synthetic train/val loop; device seam isolated; leakage guard fires |
| 5 | Public entry points | FEAT-001, artifact SC-1, SC-2 | fit() and evaluate() pass all FEAT-001 ACs; TypeError on bad inputs |
| 6 | Test suite + golden-run values | COMP-005, ADR-0008 | `pytest tests/` exits 0; golden-run constants populated; all ACs green |
| 7 | timeseries helpers | COMP-004, artifact SC-1, ADR-0002 | SlidingWindowDataset, temporal_split, MAE/MSE/RMSE pass all COMP-004 ACs |

## Tasks by Milestone

### Milestone 1: Package scaffold

- `NOT STARTED` Create pyproject.toml with name, version, requires-python, dependencies (→ COMP-001 §4.2)
- `NOT STARTED` Create trainer_kit/ directory with empty __init__.py (→ COMP-001 §4.1)
- `NOT STARTED` Create trainer_kit/timeseries/ directory with empty __init__.py (→ COMP-001 §4.1)
- `NOT STARTED` Create stub files: _protocols.py, _config.py, _engine.py, _device.py, _seed.py, _logger.py each with `# stub` (→ COMP-001 §4.1)
- `NOT STARTED` Populate trainer_kit/__init__.py with __version__ and placeholder imports (→ COMP-001 §4.3)
- `NOT STARTED` Verify: `pip install -e .` in a clean venv with torch pre-installed succeeds (→ COMP-001 §6.1.D)

### Milestone 2: Protocol definitions and StdoutLogger

- `NOT STARTED` Implement ModelProvider protocol in _protocols.py with @runtime_checkable (→ COMP-002 §4.1)
- `NOT STARTED` Implement DataSource protocol in _protocols.py with @runtime_checkable (→ COMP-002 §4.2)
- `NOT STARTED` Implement Metric protocol in _protocols.py with @runtime_checkable (→ COMP-002 §4.3)
- `NOT STARTED` Implement Logger protocol in _protocols.py with @runtime_checkable (→ COMP-002 §4.4)
- `NOT STARTED` Implement Checkpointer protocol in _protocols.py with @runtime_checkable (→ COMP-002 §4.5)
- `NOT STARTED` Implement StdoutLogger in _logger.py satisfying the Logger protocol (→ COMP-002 §4.6)
- `NOT STARTED` Verify: isinstance checks for all five protocols pass (→ COMP-002 §6.1.A–E)
- `NOT STARTED` Verify: importing _protocols.py does not import _engine.py (→ COMP-002 §6.1.F)

### Milestone 3: Config and result types

- `NOT STARTED` Implement EpochRecord frozen dataclass in _config.py (→ DATA-001 §4.3)
- `NOT STARTED` Implement Result frozen dataclass in _config.py (→ DATA-001 §4.4)
- `NOT STARTED` Implement EvalConfig frozen dataclass in _config.py with StdoutLogger default (→ DATA-001 §4.2)
- `NOT STARTED` Implement TrainConfig frozen dataclass in _config.py with all fields and __post_init__ validation (→ DATA-001 §4.1)
- `NOT STARTED` Write tests/test_config.py covering ValueError on epochs<1, val_every_n_epochs<1, and FrozenInstanceError on mutation (→ DATA-001 §6.1.A–C)
- `NOT STARTED` Verify: importing _config.py does not import _engine.py (→ DATA-001 §6.1.F)

### Milestone 4: Training engine

- `NOT STARTED` Implement _device.py: _resolve_device() and _to_device() (→ COMP-003 §4.2)
- `NOT STARTED` Implement _seed.py: _seed_all() seeding Python/NumPy/torch (→ COMP-003 §4.3)
- `NOT STARTED` Implement the leakage guard _assert_no_leakage() in _engine.py (→ COMP-003 §4.5)
- `NOT STARTED` Implement _run_evaluate() in _engine.py: single val pass under no_grad(), metric accumulation, logger hooks (→ COMP-003 §4.4, §3.1.E)
- `NOT STARTED` Implement _run_fit() in _engine.py: full train/val loop per pseudocode in COMP-003 §4.4 (→ COMP-003 §3.1.A–H)
- `NOT STARTED` Static check: grep _engine.py and _seed.py for any `.cuda()` or `.to("cuda")` calls — must be zero (→ COMP-003 §6.1.F)
- `NOT STARTED` Manual smoke test: _run_fit() completes 3 epochs on synthetic data without error (→ COMP-003 §5.1.A)
- `NOT STARTED` Verify: _engine.py has no import of trainer_kit.timeseries (→ COMP-003 §6.1.H)

### Milestone 5: Public entry points

- `NOT STARTED` Implement fit() in trainer_kit/__init__.py (or a dedicated _entry.py) with isinstance validation and delegation to _run_fit() (→ FEAT-001 §4.1)
- `NOT STARTED` Implement evaluate() with isinstance validation and delegation to _run_evaluate() (→ FEAT-001 §4.2)
- `NOT STARTED` Update trainer_kit/__init__.py exports to include fit and evaluate (→ COMP-001 §4.3)
- `NOT STARTED` Write tests/test_entry_points.py: TypeError on None inputs, evaluate() returns empty train_history, duck-typed objects pass isinstance checks (→ FEAT-001 §6.1.A–E)

### Milestone 6: Test suite and golden-run values

- `NOT STARTED` Create tests/ directory and conftest.py with shared fixtures (→ COMP-005 §4.2)
- `NOT STARTED` Write tests/test_determinism.py: two fit() calls with seed=42 produce identical results (→ COMP-005 §6.1.C)
- `NOT STARTED` Write tests/test_leakage_guard.py: fit() with leaking DataSource raises AssertionError (→ COMP-005 §6.1.D)
- `NOT STARTED` Write tests/test_protocols.py: isinstance() checks for all protocols (→ COMP-002 §6.1.A–E)
- `NOT STARTED` Run fit() once with seed=42, capture train/val loss values, populate EXPECTED_* constants in test_golden_run.py (→ COMP-005 §4.3, §7.2.A)
- `NOT STARTED` Write tests/test_golden_run.py asserting loss values match constants within TOLERANCE (→ COMP-005 §6.1.B)
- `NOT STARTED` Run `pytest tests/` and confirm exit 0 and runtime under 60 seconds (→ COMP-005 §6.1.A, §6.1.E)
- `NOT STARTED` Corruption check: temporarily break optimizer.zero_grad() and confirm golden-run test fails (→ COMP-005 §6.1.F); revert

### Milestone 7: timeseries helpers

- `NOT STARTED` Implement SlidingWindowDataset in timeseries/_dataset.py with __len__, __getitem__, and ValueError guards (→ COMP-004 §4.1, §5.3.A–B)
- `NOT STARTED` Implement temporal_split() in timeseries/_split.py (→ COMP-004 §4.2)
- `NOT STARTED` Implement MAE in timeseries/_metrics.py (→ COMP-004 §4.3)
- `NOT STARTED` Implement MSE in timeseries/_metrics.py (→ COMP-004 §4.3)
- `NOT STARTED` Implement RMSE delegating to MSE in timeseries/_metrics.py (→ COMP-004 §4.3)
- `NOT STARTED` Populate timeseries/__init__.py with exports (→ COMP-004 §4.4)
- `NOT STARTED` Write tests/timeseries/test_dataset.py: len() correctness, ValueError on too-short series, stride behavior (→ COMP-004 §6.1.A, §6.1.G)
- `NOT STARTED` Write tests/timeseries/test_split.py: non-overlapping indices, chronological order, val_fraction edge cases (→ COMP-004 §6.1.B–C)
- `NOT STARTED` Write tests/timeseries/test_metrics.py: multi-call accumulation, isinstance checks, reset() idempotence (→ COMP-004 §6.1.D–E)
- `NOT STARTED` Verify: `from trainer_kit.timeseries import ...` without first importing trainer_kit core succeeds (→ COMP-004 §6.1.F)
- `NOT STARTED` Run `pytest tests/` after adding timeseries tests; confirm still exits 0 (→ COMP-005 §6.1.A)

## Open Questions

- [ ] Metric name collision: should TrainConfig.__post_init__ raise ValueError if two metrics share __class__.__name__? (DATA-001 §7.2.B — currently an assumption, should be confirmed before Milestone 3)
- [ ] Should pytest be in the mandatory dev dependency group or a separate test group in pyproject.toml?

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-06-07 | Initial draft derived from all 8 specs and ADR-0001 through ADR-0011 | Agent |
