# Skill: Generate System Design

## Purpose
Transform a user's idea description into a complete system design — spanning research, planning, specifications, and architecture decisions — using a structured, evidence-based workflow.

## When to Use
- A user describes a new idea, product, feature, or application and wants help designing it
- A project needs to go from "rough concept" to "ready to build"
- An existing system needs a redesign or significant extension

---

## Procedures Used
- `procedures/add-research.md`
- `procedures/add-plan.md`
- `procedures/add-spec.md`
- `procedures/add-adr.md`

---

## Workflow

### Phase 0: Idea Intake (MANDATORY — do not skip)
Follow the full Idea Intake Protocol from `AGENT.md` section 2:

1. **Understand the idea** — gather: what, who, why, constraints
2. **Derive success criteria** — numbered, observable, testable conditions
3. **Derive failure modes** — categorized by Technical / Product / Operational
4. **Confirm with user** — do not proceed to Phase 1 until the user approves the criteria and failure modes

### Phase 1: Research
For each significant unknown or technology choice surfaced in Phase 0:
1. Apply `procedures/add-research.md` for each question.
2. Prioritize research that addresses failure modes directly.
3. After all research is complete, summarize key findings and recommendations for the user before moving on.

### Phase 2: Plan
1. Apply `procedures/add-plan.md` to create a formal plan in `plans/`.
2. The plan should:
   - Reference each research brief produced in Phase 1
   - List milestones that map to the success criteria from Phase 0
   - List risks that map to the failure modes from Phase 0
3. Present the plan to the user for review before moving to specs.

### Phase 3: Specify
For each component or feature identified in the plan:
1. Apply `procedures/add-spec.md`.
2. Order: start with the most foundational components (data models, core services) before dependent features.
3. Each spec's acceptance criteria should map to one or more success criteria from Phase 0.

### Phase 4: Decide
For each significant design choice made during spec writing:
1. Apply `procedures/add-adr.md`.
2. An ADR is warranted when:
   - Two or more reasonable alternatives existed
   - The choice has long-term consequences
   - A future maintainer would reasonably wonder "why was this done this way?"
3. Link each ADR back to the spec(s) that prompted it.

### Phase 5: Document (optional)
If the system design is complex enough to warrant a unified reference:
1. Create a summary document in `docs/` that links to all produced artifacts.
2. Include a system diagram (as ASCII or a description) if helpful.

---

## Output Checklist

At the end of a full system design session, the following should exist:

- [ ] Success criteria captured (in the plan or a dedicated intake doc)
- [ ] Failure modes captured (in the plan or a dedicated intake doc)
- [ ] At least one research brief per major technology choice
- [ ] A formal plan in `plans/` with milestones linked to success criteria
- [ ] Specs for each major component and user-facing feature
- [ ] ADRs for each significant architectural decision
- [ ] All folder READMEs updated with new index entries

---

## Tips

- Failure modes are as important as success criteria. A design that only optimizes for the happy path will fail in production.
- Do not write specs before research is done — specs written in ignorance require rewrites.
- ADRs written after the fact ("we already decided this, just document it") are better than no ADRs, but the best ADRs are written while the decision is live.
- When in doubt about whether something warrants an ADR: if you would want to know *why* it was done that way in 6 months, write the ADR.
