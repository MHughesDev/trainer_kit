# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repository Is

`trainer_kit` is a **design-phase** PyTorch library project. There is no implementation code yet — the repository contains exclusively documentation, design decisions, and agentic workflow scaffolding.

**The library's job:** provide callable, end-to-end train/validation pipelines for AI models. Consuming applications implement two `typing.Protocol` contracts (`ModelProvider`, `DataSource`) and call a single entry point (`fit()`). The library owns no models and no data.

---

## Mandatory Operating Protocol

**Before any task**, read `AGENT.md`. It defines three mandatory protocols:

1. **Skill Discovery** — search `docs/skills/` for a matching skill before doing anything else. If one exists, follow it. Do not improvise.
2. **Idea Intake** — fill in `docs/artifact.md` first for any new project idea before producing any other artifact.
3. **Research-First** — web-research prevailing practice, existing building blocks, and trade-offs before writing any spec, architecture entry, or design decision. If internet access is unavailable, stop and say so.

---

## Build / Test / Lint

No implementation code, build system, or test infrastructure exists yet. When implementation begins:

- **Python ≥ 3.10**, **PyTorch `>=2.1,<3`** (ADR-0006)
- Package metadata will live in `pyproject.toml`
- Planned correctness tests: golden-run regression tests, determinism checks, validation-leakage guard (ADR-0008)

---

## Architecture

### Public API (planned)

```python
fit(model_provider: ModelProvider, data_source: DataSource, config: TrainConfig) -> Result
evaluate(model_provider: ModelProvider, data_source: DataSource, config: EvalConfig) -> Result
```

### Protocol Contracts

| Protocol | Implementer | Methods |
|----------|-------------|---------|
| `ModelProvider` | Consuming app | `build() -> nn.Module` |
| `DataSource` | Consuming app | `train_loader()`, `val_loader()` |
| `Metric` | Library or app | `update()`, `compute()`, `reset()` — torchmetrics-compatible |
| `Logger` | App-injected | `on_run_start`, `on_epoch_end`, `on_run_end` (lifecycle hooks) |
| `Checkpointer` | App-injected (v1 seam only) | Caller-owned path; default `torch.save` |

### Module Layout (planned)

- `trainer_kit.core` — time-agnostic train/val engine
- `trainer_kit.timeseries` — optional sliding-window and temporal-split helpers (engine never imports this)
- `trainer_kit.config` — `TrainConfig` dataclass
- `trainer_kit.protocols` — all `typing.Protocol` definitions

### Key Boundaries

- **Owns nothing** — no datasets, model weights, or experiment-tracker SDKs in the package
- **v1 out of scope** — distributed/multi-GPU, AMP, HPO, early stopping, checkpointing behavior (seam is reserved), built-in tracker adapters
- **Internal seams** — device placement and checkpoint destination are internal so multi-GPU/storage backends can land without breaking `fit()` (ADR-0007, ADR-0004)

---

## Documentation Workflow

The design pipeline flows: `artifact.md` → `research/` → `plans/` → `specs/` → `adr/` → `architecture.md`

| File / Folder | Purpose |
|---------------|---------|
| `docs/artifact.md` | Foundational project definition — fill in first |
| `docs/open-questions.md` | Living register of every trade-off; log when raised, resolve when decided |
| `docs/architecture.md` | Emergent current-state map; update as ADRs and specs land |
| `docs/research/` | Research briefs, comparisons, prior art |
| `docs/plans/` | Working scratchpads and formal versioned roadmaps |
| `docs/specs/` | Component and feature specifications |
| `docs/adr/` | Architecture Decision Records (11 accepted, ADR-0001 through ADR-0011) |
| `docs/procedures/` | Atomic step-by-step task instructions (source of truth — do not bypass) |
| `docs/skills/` | Higher-level agent capabilities that compose procedures |

### Conventions

- **File names:** `kebab-case.md`; specs are `<TYPE>-NNN-name.md` (e.g. `FEAT-001-…`)
- **ADRs:** numbered sequentially `adr/NNNN-title.md`; status: `Proposed` → `Accepted` → `Deprecated` / `Superseded by ADR-NNNN`
- **Specs:** status: `Draft` → `Ready for Review` → `Approved` → `Implemented` → `Deprecated` / `Superseded by <SPEC-ID>`
- **Plans:** status: `Draft` → `Active` → `Complete` → `Superseded`; task status: `NOT STARTED` · `IN PROGRESS` · `COMPLETE` · `BLOCKED:UPSTREAM` · `BLOCKED:HUMAN` · `BLOCKED:TESTING`
- **Every new artifact** must be linked from its folder's `README.md`
- Procedures are editor- and agent-agnostic (never reference a specific tool)
- Do not execute any plan milestone without first running `docs/skills/execute-plan.md`
- Do not execute a plan whose `Derivation Status` is `STALE` — re-derive it first

### Stable ID System

Cross-references use these IDs so anything can be cited precisely from anywhere:

| Prefix | Lives in | Example |
|--------|----------|---------|
| `SC-N` | `artifact.md` success conditions | "milestone advances SC-2" |
| `FM-N` | `artifact.md` failure modes | "risk traces to FM-1" |
| `Q-N` | `open-questions.md` | "gated by Q-5" |
| `ADR-NNNN` | `adr/` | "constrained by ADR-0001" |
| `<TYPE>-NNN §N.M.X` | `specs/` | "task → FEAT-001 §3.1.C" |

Run `docs/skills/verify-traceability.md` at the end of every design session and before handing a plan to execution. Do not start implementing against a plan with broken references.

---

## Do Not

- Create `.cursor`, `.claude`, `.vscode`, or any other editor-specific config files
- Modify `docs/procedures/` without telling the user what changed and why
- Skip the Skill Discovery Protocol for any task
- Produce design artifacts before completing the Idea Intake Protocol for new projects
- Write specs, architecture entries, or ADRs before completing the Research-First Protocol
