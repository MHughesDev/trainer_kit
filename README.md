# System Design Template

A copy-paste starter repository for designing new ideas and applications using a structured, agentic workflow. Drop this into any new project, describe your idea, and the agent will guide you from concept to complete system design.

> **For AI agents working in this repo, see [AGENT.md](./AGENT.md) first.**

---

## What This Is

This repo is a procedurized system for turning rough ideas into well-documented architectures. It is built around three principles:

1. **Procedures are the source of truth.** Every repeatable task lives in `procedures/` as a named, atomic instruction set.
2. **Skills compose procedures.** Files in `skills/` combine procedures into higher-level agent capabilities.
3. **Folders have a single job.** Each directory has a README that defines what belongs there and why.

---

## Quick Start

1. Copy this repo into your new project.
2. Delete the example files (keep the READMEs and procedures).
3. Open a session with your AI agent and describe your idea:
   - What is the thing you are building?
   - Who uses it?
   - What problem does it solve?
4. The agent will derive **success criteria** and **failure modes** before producing any design artifacts.
5. Work through the folders in order: `research/` → `plan/` → `specs/` → `adr/`.

---

## Folder Structure

```
/
├── README.md            ← you are here
├── AGENT.md             ← agent operating instructions
│
└── docs/
    ├── adr/             ← Architecture Decision Records
    ├── plan/            ← lightweight working plans (session-scoped)
    ├── plans/           ← formal, versioned project plans
    ├── procedures/      ← atomic step-by-step task instructions
    ├── research/        ← research briefs, findings, comparisons
    ├── skills/          ← agent skill definitions (reference procedures)
    └── specs/           ← feature and component specification files
```

---

## Workflow Overview

```
Idea
 │
 ├─► Research (research/)      Gather context, prior art, constraints
 │
 ├─► Plan (plan/ or plans/)    Define scope, milestones, open questions
 │
 ├─► Specify (specs/)          Write component and feature specs
 │
 ├─► Decide (adr/)             Record architecture decisions with context
 │
 └─► Document (docs/)          Capture long-lived reference material
```

Each step has a corresponding procedure in `docs/procedures/` and a skill in `docs/skills/`.

---

## Conventions

- File names use `kebab-case`.
- Every folder has a `README.md` explaining what belongs there.
- ADRs are numbered sequentially: `adr/0001-title.md`.
- Specs reference the ADRs that informed them.
- Plans reference the research that preceded them.
- Procedures never reference specific project names — they are reusable.

---

## See Also

- [AGENT.md](./AGENT.md) — agent operating model and skill discovery protocol
- [docs/procedures/README.md](./docs/procedures/README.md) — procedure index and authoring guide
- [docs/skills/README.md](./docs/skills/README.md) — skill index and authoring guide
- [docs/adr/README.md](./docs/adr/README.md) — ADR index and template
