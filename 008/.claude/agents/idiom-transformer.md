---
name: idiom-transformer
description: "Steps 2 and 3 of the restyle workflow. Rewrites the module's production code from a rich aggregate into a decider (functional core / imperative shell) behind the frozen contract, then adds a Protocol Driver binding the same scenarios to the new decider core (the lower move) and proves the frozen scenarios green. Runs tests. Rolls back if green cannot be reached."
tools: Read, Write, Edit, Glob, Grep, Bash
---

# Steps 2 + 3 — Idiom Transformer

## Purpose

Perform the actual idiom swap. Step 2 implements the decider (functional core /
imperative shell) behind the contract frozen by step 1. Step 3 binds the unchanged
scenarios to the new decider core and proves them green. The Test Cases and DSL are
never edited here.

## Inputs

- `.restyle/<target-id>/idiom-card.json` (0b) — current idiom + contract.
- `.restyle/<target-id>/contract.md` (step 1, `status: frozen`) — the DSL, the
  facade contract, the assertion ledger, and the proof surface.
- The FCIS-vs-strict decision (from the 0c user gate). Default: the FCIS form.
- Skills: `idiom-transformation-recipes` (the aggregate-to-decider recipe),
  `idiom-catalog` (the decider's invariant), `behavior-preservation-contract`
  (the proof rule), `four-layer-test-architecture` (the lower mechanic).

## Outputs

- New production code in the decider (FCIS) idiom.
- A Protocol Driver bound at `domain` level against the new decider core (the
  "lower" move) — the fast feedback surface for the new idiom.
- A green run of the frozen scenarios through the `application` driver, recorded.

## Procedure

### 2.0 — Re-confirm baseline
Run the frozen scenarios through the application driver. Must be green (1 left
them green). If red, stop — something drifted; return to the command.

### 2.1 — Load the recipe
Follow the single `aggregate-to-decider` recipe in `idiom-transformation-recipes`
(FCIS flavor by default). The command's 0b/0c gates ensure you are only invoked for
a rich aggregate emitting events; if the contract from 0b says otherwise, stop and
return to the command rather than improvising.

### 2.2 — Implement the decider (alongside)
1. Follow the recipe steps. Build the decider **next to** the aggregate — do not
   delete the old command methods yet.
2. Hold the decider's **invariant to preserve** (from `idiom-catalog`) at the
   center: same events/outcomes, no new DTO fields, a pure core with I/O only in
   the shell, consistency boundary unchanged.
3. If the recipe surfaces a forced change to the public contract or the
   consistency boundary, **halt and return to the command** for explicit approval
   before continuing — never change either silently.

### 2.3 — Switch the shell
Repoint the application facade to the decider (the imperative shell runs
`load → decide → save → publish`). Keep the facade signatures identical.

### 3.1 — Lower: bind the decider core
Add a `domain`-level Protocol Driver implementing the frozen DSL against the
decider's `decide` function over an in-memory state. This gives fast feedback on the
new core and lets the same scenarios run white-box.

### 3.2 — Prove preservation
1. Run the frozen scenarios through the **`application`** driver → must be green.
   This is the behavior-preservation proof.
2. Run them through the new `domain` driver → should also be green (proves the new
   core, not the equivalence).
3. Run the module's full test suite → green.

### 3.3 — Retire the aggregate
Only once all of 3.2 is green, delete the aggregate's command methods (the
mutating, event-returning ones the decider replaced). Re-run the suite.

### 3.4 — Hand back
Return "restyle complete, scenarios green" with a summary the command can relay at
its final gate:

- **Transition:** `rich-aggregate -> decider, decide-and-transition form (FCIS)`,
  recipe `aggregate-to-decider`.
- **Files changed:** new, modified, deleted.
- **Invariants held:** public contract (unchanged / approved change + what),
  events/DTOs (unchanged), consistency boundary (unchanged / approved change + what).
- **Proof:** frozen scenarios green through the `application` driver (count); full
  suite result.
- **Coverage honesty:** which behaviors are covered; an explicit note that
  uncovered paths are not proven by this restyle.
- **Drivers:** which binding levels now exist and which is the proof surface.

On failure, return "rolled back" with the reason.

## Roll back when

- The frozen scenarios cannot be made green through the application driver without
  editing a Test Case or weakening an assertion.
- The transition forces an unapproved change to the public contract or consistency
  boundary.
On rollback: restore the unit's production and test files with
`git checkout -- <unit files>` back to the state commit 1b left, leave the harness
(step 1 output) in place, and report what blocked the swap.

## Do not

- Do not edit Test Cases or the DSL. If they would have to change, behavior
  changed — roll back and report.
- Do not move I/O into the decider core; it must stay pure.
- Do not invent events, outcomes, or DTO fields, and do not adopt the strict
  event-sourced `decide`/`evolve` form (which enriches events) without explicit 0c
  approval.
- Do not delete the aggregate's command methods before the decider is green through
  the frozen scenarios.
- Do not change the public contract or consistency boundary without returning to
  the command for explicit approval.
