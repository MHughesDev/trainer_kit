# specs/

Feature and component specification files. Specs define what something should do with enough precision for a developer (human or AI) to implement it, and for a tester to verify it.

> To add a new spec, follow the skill: `skills/create-spec.md` and procedure: `procedures/add-spec.md`.

---

## What Belongs Here

- Feature specs ("user authentication flow")
- Component specs ("rate limiter design")
- API contract specs (if not generated from code)
- Data model specs ("user profile schema")
- Integration specs ("payment provider webhook handling")

## What Does Not Belong Here

- Architecture decisions → `adr/`
- Research that informed the spec → `research/`
- Project plans and milestones → `plans/`

---

## Spec Types

### Feature Spec
Describes user-visible behavior: inputs, outputs, edge cases, error states.

### Component Spec
Describes an internal system component: interface, responsibilities, constraints, dependencies.

### Data Spec
Describes a data model or schema: fields, types, validation rules, relationships.

### System Overview Spec
A unified reference that links all component and feature specs for a complex system.
Includes a system diagram (ASCII or description) and a cross-reference index.
Use this instead of a separate documentation phase when the system warrants a top-level map.

---

## Naming Convention

Spec files are named with a type code + sequential number, not freeform:

```
<TYPE>-<NNN>-<kebab-name>.md
```

| Type | Code | Example |
|------|------|---------|
| Feature | `FEAT` | `FEAT-001-user-authentication.md` |
| Component | `COMP` | `COMP-002-rate-limiter.md` |
| Data | `DATA` | `DATA-001-user-profile-schema.md` |
| Integration | `INTG` | `INTG-001-stripe-webhooks.md` |
| System-overview | `SYS` | `SYS-001-system-overview.md` |

Numbering is per type (`FEAT-001` and `COMP-001` may both exist). The **Spec ID** is `<TYPE>-<NNN>`;
cite any line in a spec as `<SPEC-ID> §N.M.X` (e.g. `FEAT-001 §3.1.A`).

## Internal Structure

Every spec uses the same seven numbered sections, addressed `Section N → N.M → N.M.X`:

`1.` Overview · `2.` Scope · `3.` Requirements · `4.` Interface / Data · `5.` Behavior ·
`6.` Acceptance Criteria · `7.` Open Questions & Assumptions

See `procedures/add-spec.md` for the full template.

## Lifecycle (Status values)

`Draft` → `Ready for Review` → `Approved` → `Implemented`, plus two retirement states:
- `Deprecated` — no longer planned, nothing replaces it
- `Superseded by <SPEC-ID>` — replaced by a newer spec

Specs are never deleted — they retire so history stays readable (the same rule as ADRs).
Retiring or replacing a spec fires the plan re-derivation trigger; see `add-spec.md` Step 6.

## Conventions

- Every spec must reference the ADR(s) that inform its key design choices (§1.3)
- Specs include an explicit Non-Goals subsection (§2.2) to prevent scope creep
- Acceptance criteria (§6) cite the requirement (§3) they verify, and carry a `Verified by:` evidence field that is filled in when the criterion is actually met (closing requirement → criterion → evidence)
- Append new items at the end of a subsection (next letter) so references never shift

---

## Index

| File | Spec ID | Type | Status | Related ADR(s) |
|------|---------|------|--------|----------------|
| _none yet_ | — | — | — | — |
