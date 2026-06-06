# plans/

Formal, versioned project plans. Use this folder for plans that need to persist across sessions, be shared with stakeholders, or serve as a milestone reference throughout the project lifecycle.

> For lightweight session-scoped working notes, use `plan/` instead.

---

## What Belongs Here

- Project roadmaps
- Release plans with defined milestones
- Sprint or iteration plans that will be reviewed in retrospectives
- Multi-phase technical migration plans
- Dependency and sequencing diagrams

## What Does Not Belong Here

- Quick scratchpad notes → `plan/`
- Architecture decisions → `adr/`
- Feature specifications → `specs/`
- Research findings → `research/`

---

## Conventions

- File names: `kebab-case.md`
- Every plan should include:
  - **Goal** — what does success look like at the end of this plan?
  - **Scope** — what is explicitly in and out of scope?
  - **Milestones** — numbered phases or checkpoints
  - **Dependencies** — what must be true before this plan can proceed?
  - **Risks** — what could cause this plan to fail? (link to failure modes from intake)
- Plans reference research: `[research/topic.md](../research/topic.md)`
- Plans reference ADRs for any major decisions baked into the approach

---

## Index

| File | Description | Status |
|------|-------------|--------|
| *(empty — add your first project plan)* | — | — |
