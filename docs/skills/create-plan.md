# Skill: Create Plan

## Purpose
Create a plan document — working or formal — that translates research findings and success criteria into sequenced, executable milestones.

## When to Use
- After research is complete and before specs are written
- When scoping a sprint, phase, or session of work
- When a large feature needs to be broken into sequenced steps
- When dependencies and risks need to be made explicit before work starts

---

## Procedures Used
- `procedures/add-plan.md`

---

## Workflow

### 1. Determine plan type
| Use `plan/` (working) when... | Use `plans/` (formal) when... |
|-------------------------------|-------------------------------|
| Notes are for this session only | The plan persists across sessions |
| You're thinking through scope | The plan has defined milestones |
| The plan may be discarded | The plan will be reviewed in retrospectives |
| Low formality is fine | Stakeholders will read it |

### 2. Check that prerequisites exist
Before writing a formal plan:
- [ ] Research briefs exist for major unknowns (`research/`)
- [ ] Success criteria are documented (from project intake)
- [ ] Failure modes are documented (from project intake)

If research is missing for a significant unknown, do research first.

### 3. Connect the plan to the intake artifacts
A good plan traces directly to the project's success criteria and failure modes:
- Every milestone should advance at least one success criterion
- Every significant risk in the plan should trace to a failure mode from intake
- If a milestone has no connection to a success criterion, question whether it belongs

### 4. Execute `procedures/add-plan.md`
Follow the procedure. For formal plans, pay special attention to:
- **Success signals** for each milestone — how do you know a milestone is done?
- **Open questions** — known unknowns that must be resolved during execution
- **Dependencies** — what must be true before execution can start?

### 5. Identify specs and ADRs triggered by the plan
After writing the plan, identify:
- What components or features need specs written? → run `skills/create-spec.md` for each
- What design decisions are implied by the plan's approach? → run `skills/create-adr.md` for each

---

## Tips

- A milestone without a success signal is not a milestone — it is a wish.
- List risks explicitly. A plan that pretends risks don't exist will be ambushed by them.
- Plans should be short enough to read in one sitting. If a plan exceeds ~4 pages, it is covering too much scope and should be split into phases.
- Working plans (`plan/`) can be rough. Formal plans (`plans/`) must be precise enough that someone else could execute them.
