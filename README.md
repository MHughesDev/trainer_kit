# template

A copy-paste starter repository for designing new ideas and applications using a structured, agentic workflow. Drop this into any new project, describe your idea, and the agent will guide you from concept to complete system design.

> **For AI agents working in this repo, see [AGENT.md](./AGENT.md) first.**

---

## What This Is

This repo is a procedurized system for turning rough ideas into well-documented architectures. It is application- and stack-agnostic — it defines documentation *practice*, not any particular system. It is built around four principles:

1. **Procedures are the source of truth.** Every repeatable task lives in `procedures/` as a named, atomic instruction set.
2. **Skills compose procedures.** Files in `skills/` combine procedures into higher-level agent capabilities.
3. **Folders have a single job.** Each directory has a README that defines what belongs there and why.
4. **Research before design.** The agent must research prevailing practice, existing building blocks, and trade-offs on the internet before writing any spec, architecture, or decision — never assume. See [AGENT.md](./AGENT.md) §3.

---

## Quick Start

1. Copy this repo into your new project.
2. Delete the example files (see the manifest below).
3. Open a session with your AI agent and describe your idea.
4. The agent will fill in `docs/artifact.md` with you before producing any other artifact.
5. Work through the folders in order: `docs/artifact.md` → `research/` → `plans/` → `specs/` → `adr/`.

### Example files to delete

A worked example (a B2B task-management app) ships across these files to demonstrate the format.
Delete all of them before starting your own project — they are illustrative, not scaffolding.
Keep everything else (READMEs, procedures, skills, and the blank `docs/artifact.md` template).

| File | What it demonstrates |
|------|----------------------|
| `docs/research/example-auth-provider-comparison.md` | A complete research brief |
| `docs/adr/0001-example-use-clerk-for-auth.md` | A complete ADR |
| `docs/specs/FEAT-001-user-authentication.md` | A complete spec (note: no `example-` prefix — it follows the spec naming convention) |
| `docs/plans/example-task-management-app.md` | A complete formal plan |
| The `Q-1`–`Q-5` rows and the Q-5 detail block in `docs/open-questions.md` | Example open-question entries |

> One-liner to remove the four standalone files:
> ```
> rm docs/research/example-*.md docs/adr/0001-example-*.md docs/specs/FEAT-001-*.md docs/plans/example-*.md
> ```
> Then clear the example rows from `docs/open-questions.md` by hand.

---

## Folder Structure

```
/
├── README.md            ← you are here
├── AGENT.md             ← agent operating instructions
│
└── docs/
    ├── artifact.md       ← foundational project definition (fill in first)
    ├── open-questions.md ← living register of trade-offs and open forks
    ├── architecture.md   ← current-state system map (emergent, never assumed)
    ├── adr/              ← Architecture Decision Records
    ├── plans/            ← all plans: working scratchpads and formal roadmaps
    ├── procedures/       ← atomic step-by-step task instructions
    ├── research/         ← research briefs, findings, comparisons
    ├── skills/           ← agent skill definitions (reference procedures)
    └── specs/            ← feature and component specification files
```

---

## Workflow Overview

```
Idea
 │
 ├─► Define (artifact.md)      Fill in the foundational project definition
 │
 ├─► Research (research/)      Gather context, prior art, constraints
 │
 ├─► Plan (plans/)             Define scope, milestones, open questions
 │
 ├─► Specify (specs/)          Write component and feature specs
 │
 └─► Decide (adr/)             Record architecture decisions with context
```

Each step has a corresponding procedure in `docs/procedures/` and a skill in `docs/skills/`.

Two files live *across* all steps: `docs/open-questions.md` (every trade-off the agent weighs) and `docs/architecture.md` (the current-state map, updated as decisions land).

---

## Conventions

- File names use `kebab-case` (specs are the exception: `<TYPE>-NNN-name.md`).
- Every folder has a `README.md` explaining what belongs there.
- ADRs are numbered sequentially: `adr/0001-title.md`.
- Specs reference the ADRs that informed them.
- Plans reference the research that preceded them, and are *derived* from artifacts, never assumed.
- Procedures never reference specific project names — they are reusable.

### Stable IDs

Each artifact assigns IDs so anything can be cited precisely from anywhere downstream:

| Prefix | Lives in | Example reference |
|--------|----------|-------------------|
| `SC-N` | artifact success conditions | "milestone advances SC-2" |
| `FM-N` | artifact failure modes | "risk traces to FM-1" |
| `Q-N` | open-questions register | "gated by Q-5" |
| `ADR-NNNN` | `adr/` | "constrained by ADR-0001" |
| `<TYPE>-NNN` + `§N.M.X` | specs | "task → FEAT-001 §3.1.C" |

### Status vocabularies

Each artifact type has its own status set — they are intentionally different:

| Artifact | Status values |
|----------|---------------|
| Research brief | `In Progress` · `Complete` |
| Open question | `Open` · `Researching` · `Resolved` |
| Plan | `Draft` · `Active` · `Complete` · `Superseded` (plus `Derivation Status:` `Current` / `STALE`) |
| Plan task | `NOT STARTED` · `IN PROGRESS` · `COMPLETE` · `BLOCKED:UPSTREAM` · `BLOCKED:HUMAN` · `BLOCKED:TESTING` |
| Spec | `Draft` · `Ready for Review` · `Approved` · `Implemented` · `Deprecated` · `Superseded by <SPEC-ID>` |
| ADR | `Proposed` · `Accepted` · `Deprecated` · `Superseded by ADR-NNNN` |

---

## See Also

- [AGENT.md](./AGENT.md) — agent operating model and skill discovery protocol
- [docs/procedures/README.md](./docs/procedures/README.md) — procedure index and authoring guide
- [docs/skills/README.md](./docs/skills/README.md) — skill index and authoring guide
- [docs/adr/README.md](./docs/adr/README.md) — ADR index and template
