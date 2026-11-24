# Harness design decisions

Running log of decisions for our PLE competition entry. Newest at the bottom of each section.

## Decided

1. **Deliverables are frozen designs, not programs.** The delivered file (e.g. `solution.py`)
   takes no inputs and recomputes nothing from data at run time. It is emitted by the harness
   from a design state, with every value written as a literal. A small fixed helper runtime
   (e.g. `place_box(...)`) is allowed; loops over data are not.
2. **The harness is data-driven internally:** facts → design → emitter → deliverable.
   - *Facts*: structured data read from the inputs, each item tagged with its source (sheet/note).
   - *Design*: the decisions, as structured state.
   - *Emitter*: renders the design into the deliverable deterministically (stable order and rounding).
3. **Evaluator first.** Before designing, the agent writes its own evaluator for the task:
   objective + constraints as code, computed deterministically. It then designs, emits, and
   evaluates the emitted deliverable in a loop.
4. **Requirement coverage.** The agent first extracts the task's requirements; every
   requirement maps to a check and every check traces back to a requirement.
5. **Evaluators are tested before they are trusted.** Each check must fail on a design broken
   specifically to trip it, and the evaluator must pass a trivially compliant design.
6. **Shared check library.** Generic checks (deliverable exists at the exact path, runs headless
   within the time limit, gives the same output twice, IFC4 + `express_rules` validation, unit
   sanity, clashes, connectivity, keep-out volumes) are tested library code. The agent writes only
   the task-specific parts. The library is uploaded into the task container (pure Python + numpy).
7. **Evaluators report a breakdown, not one number:** gates, per-rule violations with element
   IDs, and the objective with its inputs.
8. **Checks run on the emitted deliverable against the full set of facts**, not just the
   elements the agent chose to model.

## To test

- **Separate evaluator-author role** (evaluator written by a different role from the designer,
  frozen before design, revisions logged with a reason). Run with and without; keep it only
  if it measurably matters.

## Leaning towards (not yet decided)

- Hybrid design: a deterministic solver written by the agent for optimisation-shaped problems
  (routing, sizing, cost); LLM judgement for interpretive work (reading drawings, applying code
  clauses, memo arguments).
- Facts kept as a JSON file in the task container, with predefined consistency checks
  (counts match schedules, row ends land on their stated tiles, sheets agree on dimensions).
- Image input for the model (`read_file` or a `view_image` tool). None of the samples support it.
- Use the dev set to score the evaluator separately from the design: compare the agent's
  evaluator with the real grader on the same designs.

## Findings behind these decisions (task 1: `data-hall-mep-routing`)

- Score = (1 − violation_rate) × min(1, L_opt / L), behind three pass/fail gates: IFC4 schema,
  zero clashes over 2 mm, connectivity. This variant does not grade how accurately the room was
  rebuilt.
- The grader checks clashes and compliance only against the agent's own model, so leaving
  elements out hides violations. That is why our checks must use the full set of facts (decision 8).
- Reading facts accurately looks like the main source of lost points; the routing itself is
  fairly forgiving.
