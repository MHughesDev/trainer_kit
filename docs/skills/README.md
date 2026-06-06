# skills/

Agent skill definitions. Skills are higher-level capabilities that combine one or more procedures from `procedures/` into a coherent workflow for a specific goal.

> **Agents:** Before any task, search this folder for a matching skill. See [AGENT.md](../AGENT.md) section 1.

---

## Skill vs. Procedure

| | Procedure | Skill |
|--|-----------|-------|
| **Scope** | One atomic task | A goal achieved by composing tasks |
| **Detail** | Step-by-step instructions | When to use, how to sequence, what to watch for |
| **Source of truth** | Yes — authoritative | No — references procedures |
| **Modified by** | Agent (with user notice) | Agent (with user notice) |

Skills never replace procedures. If a skill and its underlying procedure conflict, the procedure wins.

---

## Skill Index

| File | Purpose |
|------|--------|
| [generate-system-design.md](./generate-system-design.md) | Full workflow: idea intake → research → plan → spec → ADR |
| [create-adr.md](./create-adr.md) | Create a new Architecture Decision Record |
| [create-spec.md](./create-spec.md) | Create a new specification file |
| [create-research-brief.md](./create-research-brief.md) | Create a new research brief |
| [create-plan.md](./create-plan.md) | Create a working or formal plan |

---

## Authoring a New Skill

```markdown
# Skill: [Name]

## Purpose
One sentence: what goal does this skill help accomplish?

## When to Use
[Conditions or triggers that indicate this skill is appropriate]

## Procedures Used
- `procedures/[file].md`
- `procedures/[file].md`

## Workflow
[Step-by-step, referencing procedure names rather than duplicating their content]

## Tips
[Common pitfalls, useful context, or things to confirm with the user]
```
