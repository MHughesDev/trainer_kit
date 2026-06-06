# procedures/

Atomic, step-by-step instructions for every repeatable task in this repo. Procedures are the source of truth. Skills in `skills/` reference and compose procedures — they never replace them.

---

## What Is a Procedure?

A procedure is:
- **Atomic** — one task, one file, one clear outcome
- **Reusable** — written without project-specific names or paths
- **Complete** — someone following it should not need to guess or improvise
- **Agent-agnostic** — works whether executed by a human or an AI

A procedure is not:
- A high-level overview (that belongs in a skill or README)
- A project plan (that belongs in `plans/`)
- A design decision (that belongs in `adr/`)

---

## Procedure Index

| File | Task |
|------|------|
| [add-adr.md](./add-adr.md) | Add a new Architecture Decision Record |
| [add-research.md](./add-research.md) | Add a new research brief |
| [add-plan.md](./add-plan.md) | Add a new plan (working or formal) |
| [add-spec.md](./add-spec.md) | Add a new specification file |

---

## Authoring a New Procedure

When creating a new procedure file, use this structure:

```markdown
# Procedure: [Task Name]

## Purpose
One sentence: what does following this procedure produce?

## Trigger
When should this procedure be used?

## Inputs
What information must be known or gathered before starting?

## Steps
1. Step one
2. Step two
   - Sub-step if needed
3. Step three

## Outputs
What artifact(s) does this procedure produce?

## Checklist
- [ ] Item 1
- [ ] Item 2

## Related
- Skill: `skills/relevant-skill.md`
- Procedure: `procedures/related-procedure.md`
```

---

## Rules for Modifying Procedures

- Never modify a procedure without explaining the change to the user.
- If a procedure is no longer accurate, update it — do not delete it.
- If a task has split into two distinct procedures, create a second file; do not bloat the original.
