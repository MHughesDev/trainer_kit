# Spec: COMP-001 — Package Structure and Module Hierarchy

**Spec ID:** COMP-001
**Type:** Component
**Status:** Draft
**Date:** 2026-06-07
**Author:** Agent

## 1. Overview
1.1 Purpose — Defines the Python package layout, module hierarchy, public exports, and pyproject.toml requirements so all other specs have a stable target to build against.
1.2 Context — trainer_kit is a pip-installable library (ADR-0005). Its public API must be importable from the top-level namespace. The internal module split (core, timeseries, protocols, config) isolates the optional timeseries helpers from the agnostic engine (ADR-0002).
1.3 Related artifacts
   1.3.A ADR-0005: [adr/0005-distribution.md](../adr/0005-distribution.md)
   1.3.B ADR-0006: [adr/0006-version-matrix.md](../adr/0006-version-matrix.md)
   1.3.C ADR-0002: [adr/0002-time-agnostic-engine.md](../adr/0002-time-agnostic-engine.md)

## 2. Scope
2.1 Goals
   2.1.A Define the on-disk package structure that all other specs depend on.
   2.1.B Specify the pyproject.toml fields needed for pip installation and version enforcement.
   2.1.C Specify what is re-exported from the top-level trainer_kit namespace.
2.2 Non-Goals
   2.2.A Does not define the runtime behavior of any module — that is in COMP-002 through COMP-005 and FEAT-001.
   2.2.B Does not cover CI/CD pipeline configuration.
   2.2.C Does not bundle or distribute any dataset or model weight (artifact SC-3).

## 3. Requirements
3.1 Functional requirements
   3.1.A The top-level trainer_kit namespace must export: fit, evaluate, ModelProvider, DataSource, Metric, Logger, Checkpointer, TrainConfig, EvalConfig, Result, EpochRecord.
   3.1.B trainer_kit.timeseries must be a separately importable sub-package; importing it must not be required to use the core pipeline.
   3.1.C Internal modules (_engine, _device, _seed) must be prefixed with underscore and not exported from the top-level namespace.
   3.1.D The package must declare a version string accessible as trainer_kit.__version__.
3.2 Non-functional requirements
   3.2.A Mandatory runtime dependencies: torch>=2.1,<3. typing_extensions is allowed for Python 3.10 compatibility. No other mandatory runtime dependencies.
   3.2.B Python requires: >=3.10.
   3.2.C The package must be installable in a clean virtualenv with pip install trainer_kit (or pip install -e . for development).

## 4. Interface / Data

### 4.1 Directory layout

```
trainer_kit/
├── __init__.py           ← re-exports all public names (§3.1.A)
├── _protocols.py         ← Protocol definitions (consumed by COMP-002)
├── _config.py            ← TrainConfig, EvalConfig, Result, EpochRecord (DATA-001)
├── _engine.py            ← Training engine core (COMP-003)
├── _device.py            ← Internal device/placement seam (COMP-003 §4.2)
├── _seed.py              ← Seeding utilities (COMP-003 §4.3)
├── _logger.py            ← StdoutLogger default implementation (COMP-002 §4.4)
└── timeseries/
    ├── __init__.py       ← re-exports timeseries public names
    ├── _dataset.py       ← SlidingWindowDataset (COMP-004)
    ├── _split.py         ← temporal_split (COMP-004)
    └── _metrics.py       ← MAE, MSE, RMSE default metrics (COMP-004)
```

### 4.2 pyproject.toml required fields

```toml
[project]
name = "trainer_kit"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "torch>=2.1,<3",
]

[project.optional-dependencies]
dev = [
    "pytest",
    "torch>=2.1,<3",
]
```

### 4.3 trainer_kit/__init__.py exports

```python
from trainer_kit._protocols import (
    ModelProvider,
    DataSource,
    Metric,
    Logger,
    Checkpointer,
)
from trainer_kit._config import TrainConfig, EvalConfig, Result, EpochRecord
from trainer_kit._engine import fit, evaluate

__version__ = "0.1.0"

__all__ = [
    "fit",
    "evaluate",
    "ModelProvider",
    "DataSource",
    "Metric",
    "Logger",
    "Checkpointer",
    "TrainConfig",
    "EvalConfig",
    "Result",
    "EpochRecord",
    "__version__",
]
```

## 5. Behavior
5.1 Happy path
   5.1.A `import trainer_kit` succeeds and exposes all names in §3.1.A.
   5.1.B `from trainer_kit.timeseries import SlidingWindowDataset, temporal_split, MAE, MSE, RMSE` succeeds without importing the engine.
   5.1.C `trainer_kit.__version__` returns a PEP 440 version string.
5.2 Edge cases
   5.2.A Importing trainer_kit.timeseries must not trigger any import of torch beyond what torch itself imports — no model construction or CUDA initialization.
5.3 Error states
   5.3.A If torch is not installed, importing trainer_kit raises ImportError with a clear message naming torch as the missing dependency.

## 6. Acceptance Criteria
6.1 Criteria
   6.1.A [ ] `from trainer_kit import fit, evaluate, ModelProvider, DataSource, Metric, Logger, Checkpointer, TrainConfig, EvalConfig, Result, EpochRecord` succeeds in a clean install (verifies §3.1.A) — Verified by: [—]
   6.1.B [ ] `from trainer_kit.timeseries import SlidingWindowDataset, temporal_split, MAE, MSE, RMSE` succeeds without first importing trainer_kit core (verifies §3.1.B) — Verified by: [—]
   6.1.C [ ] `trainer_kit._engine` is not present in `dir(trainer_kit)` (verifies §3.1.C) — Verified by: [—]
   6.1.D [ ] `pip install -e .` in a clean venv with only torch pre-installed succeeds (verifies §3.2.A, §3.2.C) — Verified by: [—]
   6.1.E [ ] `python -c "import trainer_kit; print(trainer_kit.__version__)"` prints a version string matching the pyproject.toml version (verifies §3.1.D) — Verified by: [—]

## 7. Open Questions & Assumptions
7.1 Open questions — see [open-questions.md](../open-questions.md). Distribution (PyPI vs. VCS) resolved by ADR-0005.
7.2 Assumptions
   7.2.A The version string is hardcoded in __init__.py for v1; importlib.metadata can be adopted later without an API break — Validated: [—]
   7.2.B typing_extensions is not listed as a mandatory dependency because Python 3.10 ships typing.Protocol with runtime_checkable natively — Validated: [—]
