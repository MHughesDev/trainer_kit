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
- Runbooks or operational docs → `docs/`

---

## Spec Types

### Feature Spec
Describes user-visible behavior: inputs, outputs, edge cases, error states.

### Component Spec
Describes an internal system component: interface, responsibilities, constraints, dependencies.

### Data Spec
Describes a data model or schema: fields, types, validation rules, relationships.

---

## Conventions

- File names: `kebab-case.md` — include the type if helpful: `user-auth-feature.md`, `rate-limiter-component.md`
- Every spec must reference the ADR(s) that inform its key design choices
- Specs include explicit **out of scope** sections to prevent scope creep
- Use numbered acceptance criteria so they can be checked off

---

## Index

| File | Type | Status | Related ADR(s) |
|------|------|--------|----------------|
| *(empty — add your first spec)* | — | — | — |
