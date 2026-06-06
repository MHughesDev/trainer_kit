# AGENT.md — Agent Operating Instructions

> This file governs how an AI agent should behave inside this repository. Read it before touching anything else. Also read [README.md](./README.md) for project context.

---

## 1. Skill Discovery Protocol (MANDATORY)

**Before processing any user query or executing any task**, search the `skills/` folder for a skill that matches the request.

Steps:
1. List all files in `docs/skills/`.
2. Read the title and `## Purpose` block of any skill that might apply.
3. If a matching skill exists, follow it — do not improvise a different approach.
4. If no skill matches, check `docs/procedures/` for relevant atomic steps to compose.
5. Only freeform if neither skills nor procedures apply. In that case, consider whether to create a new skill or procedure afterward.

This ensures repeatable, consistent outputs and keeps the repo self-improving.

---

## 2. Idea Intake Protocol

When a user describes a new idea or application, do not immediately start producing design artifacts. First complete the intake sequence:

### Step 1 — Understand the Idea
Ask for (or derive from context):
- What is being built?
- Who are the primary users?
- What problem does it solve?
- What constraints exist (timeline, budget, team size, existing stack)?

### Step 2 — Derive Success Criteria
Generate a numbered list of **success criteria** — observable, testable conditions that confirm the system is working as intended. Example format:

```
Success Criteria:
1. A user can complete [core action] in under [N] seconds.
2. The system handles [N] concurrent users without degradation.
3. Data is persisted durably across restarts.
```

Present these to the user and confirm or refine before continuing.

### Step 3 — Derive Failure Modes
Generate a numbered list of **failure modes** — specific ways the system could fail to meet its success criteria, broken into categories:

```
Failure Modes:
Technical:
  - [component] fails under [condition]
  - [dependency] becomes unavailable
Product:
  - Users misunderstand [interaction]
  - Core action is too slow or too complex
Operational:
  - No runbook for [failure scenario]
  - Monitoring does not cover [edge case]
```

### Step 4 — Confirm Before Designing
Present the success criteria and failure modes to the user. Get explicit confirmation (or corrections) before producing research, plans, specs, or ADRs.

---

## 3. Folder Responsibilities

| Folder | What goes there | Agent action |
|--------|----------------|--------------|
| `docs/research/` | Findings, comparisons, prior art, technology evaluations | Follow `docs/skills/create-research-brief.md` |
| `docs/plan/` | Lightweight working plans, session scratchpads | Follow `docs/skills/create-plan.md` |
| `docs/plans/` | Formal versioned project plans | Follow `docs/skills/create-plan.md` |
| `docs/specs/` | Component and feature specifications | Follow `docs/skills/create-spec.md` |
| `docs/adr/` | Architecture Decision Records | Follow `docs/skills/create-adr.md` |
| `docs/procedures/` | Atomic task instructions | Only modify when adding/improving a procedure |
| `docs/skills/` | Agent skill definitions | Only modify when adding/improving a skill |

---

## 4. Output Standards

- File names: `kebab-case.md`
- Headings: Title Case for H1, Sentence case for H2+
- Every new artifact must be linked from the relevant folder's `README.md`
- ADRs are numbered: `adr/0001-title.md`, `adr/0002-title.md`, etc.
- Specs reference the ADR(s) that informed them
- Plans reference the research that preceded them

---

## 5. Constraints

- Do not create `.cursor`, `.claude`, `.vscode`, or any editor-specific config files.
- Do not modify `procedures/` without explicitly telling the user what changed and why.
- Do not skip the Skill Discovery Protocol (section 1) for any task.
- Do not produce design artifacts before completing the Idea Intake Protocol (section 2) for new projects.
- Keep procedures editor- and agent-agnostic. They must work regardless of the AI tool being used.

---

## 6. Self-Improvement

If you complete a task and realize no skill or procedure covered it:
1. Draft a new procedure in `docs/procedures/` for the atomic steps you followed.
2. Draft a new skill in `docs/skills/` if it warrants a higher-level wrapper.
3. Update the relevant `README.md` index.
4. Mention what you added to the user at the end of the session.

---

## References

- [README.md](./README.md) — project overview and folder structure
- [docs/skills/README.md](./docs/skills/README.md) — full skill index
- [docs/procedures/README.md](./docs/procedures/README.md) — full procedure index
