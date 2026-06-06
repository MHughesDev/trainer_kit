# Procedure: Add Specification File

## Purpose
Create a spec file that defines what a feature, component, API, or data model should do — precisely enough to implement and verify without ambiguity.

## Trigger
Use this procedure when:
- Defining the behavior of a new user-facing feature
- Specifying the interface and responsibilities of a system component
- Documenting an API contract before implementation
- Defining a data model with validation rules and constraints
- Specifying an integration with an external system

## Inputs
Before starting, gather:
- The **type** of spec (feature, component, data, integration)
- The **name** of the thing being specified
- Any relevant **research briefs** from `research/` that inform this spec
- Any relevant **ADRs** from `adr/` that constrain design choices
- The **success criteria** from the project intake (or a scoped subset for this feature)

---

## Steps

### 1. Determine the spec type

| Type | Use when | Naming example |
|------|----------|----------------|
| Feature spec | Specifying user-visible behavior end-to-end | `user-login-feature.md` |
| Component spec | Specifying an internal service or module | `rate-limiter-component.md` |
| Data spec | Specifying a schema, model, or data contract | `user-profile-schema.md` |
| Integration spec | Specifying behavior with an external system | `stripe-webhook-integration.md` |

### 2. Create the spec file
1. File path: `specs/descriptive-name-[type].md`
2. Use kebab-case, include the type suffix when the name alone is ambiguous.

### 3. Fill in the spec template

```markdown
# Spec: [Name]

**Type:** Feature | Component | Data | Integration
**Status:** Draft | Ready for Review | Approved | Implemented
**Date:** YYYY-MM-DD
**Author:** [name or "Agent"]

## Related
- ADR: [adr/NNNN-title.md](../adr/NNNN-title.md)
- Research: [research/file.md](../research/file.md)
- Plan: [plans/file.md](../plans/file.md)

---

## Overview

[One paragraph describing what this spec covers and why it exists.]

## Goals

- [What this spec is designed to achieve]

## Non-Goals (Out of Scope)

- [What this spec explicitly does not cover — prevents scope creep during implementation]

---

## [For Feature Specs] User Stories

### [Story 1 Name]
**As a** [user type]  
**I want to** [action]  
**So that** [outcome]

---

## [For Component Specs] Interface

### Inputs
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| [name] | [type] | Yes/No | [description] |

### Outputs
| Field | Type | Description |
|-------|------|-------------|
| [name] | [type] | [description] |

### Errors
| Code / Type | Condition | Handling |
|-------------|-----------|----------|
| [error] | [when this happens] | [expected behavior] |

---

## [For Data Specs] Schema

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| [name] | [type] | Yes/No | [rules] | [description] |

---

## Behavior

[Describe the core behavior. Use numbered lists for sequences, tables for state transitions, and sub-headings for major scenarios.]

### Happy Path
1. Step 1
2. Step 2

### Edge Cases
- [Edge case 1]: [expected behavior]
- [Edge case 2]: [expected behavior]

### Error States
- [Error condition]: [expected system behavior]

---

## Acceptance Criteria

All of the following must be true for this spec to be considered implemented:

- [ ] AC-1: [Observable, testable condition]
- [ ] AC-2: [Observable, testable condition]
- [ ] AC-3: [Observable, testable condition]

---

## Open Questions

- [ ] [Question that must be resolved before or during implementation]

## Assumptions

- [Things assumed to be true that, if wrong, would change this spec]
```

### 4. Update the specs index
1. Open `specs/README.md`.
2. Add a row to the Index table:
   ```
   | [file-name.md](./file-name.md) | [Type] | Draft | [ADR-NNNN] |
   ```

### 5. Link related files
- If an ADR was written because of this spec, add a link in the spec's **Related** section.
- If this spec references a research brief, link to it.
- If this spec is part of a plan, add the spec file to the relevant plan's task list.

---

## Outputs
- A new file: `specs/descriptive-name.md`
- Updated index in `specs/README.md`
- Cross-links in related ADRs, plans, or research files

## Checklist
- [ ] Spec type is declared
- [ ] Non-goals section is present and non-empty
- [ ] Acceptance criteria are observable and testable (not vague)
- [ ] Edge cases and error states are documented
- [ ] Related ADRs and research are linked
- [ ] Index row added to `specs/README.md`
- [ ] No placeholder text remaining in the template

## Related
- Skill: `skills/create-spec.md`
- Procedure: `procedures/add-adr.md` (spec design choices often warrant an ADR)
- Procedure: `procedures/add-research.md` (spec uncertainty often warrants research)
