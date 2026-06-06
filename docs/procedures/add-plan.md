# Procedure: Add Plan

## Purpose
Create a plan document — either a lightweight working plan in `plan/` or a formal versioned project plan in `plans/`.

## Trigger
Use this procedure when:
- Scoping work for a session or sprint (→ `plan/`)
- Defining a multi-phase technical approach (→ `plans/`)
- Breaking down a large feature or migration into sequenced steps (→ `plans/`)
- Capturing open questions before diving into specs (→ `plan/`)

## Inputs
Before starting, determine:
- **Type**: working plan (transient, `plan/`) or formal plan (persistent, `plans/`)
- **Goal**: what does success look like at the end of this plan?
- **Scope**: what is in and out of scope?
- **Known dependencies**: what must be true or complete before this plan can execute?

---

## Steps

### 1. Choose the correct folder
| If the plan is... | Use folder |
|-------------------|------------|
| A session scratchpad, quick scope outline, or spike note | `plan/` |
| A formal roadmap, release plan, or multi-phase migration | `plans/` |

### 2. Create the plan file

**For working plans (`plan/`):**
- Name: `kebab-case-description.md` or `YYYY-MM-DD-description.md`
- Example: `2024-03-15-auth-service-spike.md`

**For formal plans (`plans/`):**
- Name: `kebab-case-plan-title.md`
- Example: `api-rate-limiting-rollout.md`

### 3. Fill in the plan template

**Working plan template (`plan/`):**
```markdown
# [Plan Title]

**Date:** YYYY-MM-DD
**Status:** Active | Complete | Abandoned

## Goal
[One sentence: what are we trying to achieve or figure out?]

## Scope
- In: [what is included]
- Out: [what is excluded]

## Open Questions
- [ ] [Question 1]
- [ ] [Question 2]

## Steps / Notes
[Freeform working notes, task list, or outline]

## Outcome
[Fill this in when the plan is complete or abandoned]
```

**Formal plan template (`plans/`):**
```markdown
# [Plan Title]

**Date:** YYYY-MM-DD
**Author:** [name or "Agent"]
**Status:** Draft | Active | Complete | Superseded

## Goal

[What does success look like at the end of this plan? Make it measurable.]

## Scope

**In scope:**
- [Item]

**Out of scope:**
- [Item]

## Dependencies

- [What must be true or done before this plan begins?]
- Reference research: [research/file.md](../research/file.md)
- Reference ADR: [adr/NNNN-title.md](../adr/NNNN-title.md)

## Risks

[Reference failure modes from the project intake, or list plan-specific risks]
- Risk: [description] → Mitigation: [approach]

## Milestones

| # | Milestone | Description | Success Signal |
|---|-----------|-------------|----------------|
| 1 | [Name] | [What happens] | [How you know it's done] |
| 2 | [Name] | [What happens] | [How you know it's done] |

## Tasks by Milestone

### Milestone 1: [Name]
- [ ] Task 1
- [ ] Task 2

### Milestone 2: [Name]
- [ ] Task 1
- [ ] Task 2

## Open Questions

- [ ] [Question that must be resolved before or during execution]

## Change Log

| Date | Change | Author |
|------|--------|--------|
| YYYY-MM-DD | Initial draft | [name] |
```

### 4. Update the folder index
1. Open the README.md in the folder you created the file in (`plan/README.md` or `plans/README.md`).
2. Add a row to the Index table with the file name, description, and status.

---

## Outputs
- A new file in `plan/` or `plans/`
- Updated index in the corresponding `README.md`

## Checklist
- [ ] Correct folder chosen (working vs. formal)
- [ ] Goal is clear and measurable
- [ ] Scope explicitly states what is out of scope
- [ ] Dependencies are listed and linked
- [ ] Index row added to the folder README

## Related
- Skill: `skills/create-plan.md`
- Procedure: `procedures/add-research.md` (plans should reference prior research)
- Procedure: `procedures/add-spec.md` (plans often spawn specs)
- Procedure: `procedures/add-adr.md` (plans often surface decisions that need ADRs)
