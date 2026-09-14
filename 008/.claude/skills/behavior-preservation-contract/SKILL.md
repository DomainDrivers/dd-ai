---
name: behavior-preservation-contract
description: Defines what must stay invariant when a module is restyled from one idiom to another, and the green-tests rule that proves it. Use to set up the safety gate before a restyle and to verify behavior was preserved after it.
---

# Behavior-Preservation Contract

A restyle is only legitimate if the module behaves the same afterward. The method
makes the swap behavior-preserving **by construction**: freeze the behavior as
tests, change only what is allowed to change, and let the unchanged tests be the
proof — rather than swapping first and hoping. This skill defines (1) the
non-negotiable precondition, (2) the three things held invariant, and (3) the rule
that proves preservation.

## Precondition: there must be tests, and they must be green

**No tests, no restyle.** The entire method is "swap the implementation under a
green safety net" — without the net there is nothing to swap under.

Before any transform step:
1. Locate the module's tests.
2. Run them. Record a **green baseline**.
3. If there are no tests for the module's behavior, **stop** and first write
   characterization tests that pin the current observable behavior. Only then
   proceed.

A red baseline is a hard stop: fix or quarantine failing tests first. You cannot
prove you preserved behavior against a suite that was already broken.

## The three invariants

Across any idiom swap, these must read identically before and after:

1. **The scenarios and their assertions.** The set of behavior tests (Test Cases)
   and the outcomes they assert, expressed in domain terms. If a Test Case must
   change to make the new idiom pass, behavior changed — that is not a restyle.
2. **The public contract.** The application facade / use-case method signatures,
   the results, events, or DTOs they return, and the **errors they signal** with
   the types those errors carry. A rejection that threw before still throws the
   same type; a rejection that came back as an empty result still comes back
   empty. Callers outside the module must not be able to tell which idiom is
   underneath.
3. **The consistency boundary.** What is loaded and saved as one atomic unit. A
   swap may change object shapes, file layout, or function signatures *inside* the
   module, but it must not silently widen or narrow the atomic unit. If the target
   idiom *requires* changing this (for example, moving to event-sourced append
   semantics), that is an explicit decision surfaced and approved before
   proceeding — never a side effect.

## The proof rule

Behavior preservation is proven by exactly one thing:

> The **same** frozen scenarios, run through a Protocol Driver bound at the
> **`application` (contract) level**, are green against the new idiom.

Notes:
- The `application`-level driver is the one that matters for the proof, because it
  is idiom-agnostic (see `four-layer-test-architecture`). A `domain`-level driver
  is idiom-specific and is rewritten per idiom, so green there proves the new core
  works — not that two idioms agree.
- "Green" means the unchanged assertions pass. If you had to relax, delete, or
  reinterpret an assertion to get green, you changed behavior — surface it.

## What this contract does **not** claim

- It does not claim the new idiom is correct on paths the scenarios do not
  exercise. Preservation is bounded by what the tests cover. State this honestly
  in the final summary; do not present green tests as exhaustive proof.
- It does not claim the swap was a good idea — only that it kept observable
  behavior the same. Idiom fitness is a separate judgment (see `idiom-catalog`).

## Lean mode (deliberate scope)

This is a green-tests safety net, not a formal coverage or mutation gate — by
design. The consequences, stated honestly:

- **Coverage gaps are caveats, not blockers.** Behaviors the baseline does not
  exercise are surfaced and named, and the restyle proceeds; they are simply not
  proven by it. The final summary lists them rather than implying full coverage.
- **Green tests prove preservation only on the paths they cover.** Never present
  them as a total correctness guarantee.
- **A stricter mode is possible but out of scope by default:** add a coverage or
  mutation threshold at the safety gate, plus a characterization snapshot taken
  before the swap and re-checked after. Reach for that only when the stakes
  warrant it; the default stays lean.

## Verification checklist (run at the end of a restyle)

- [ ] Green baseline was recorded before the transform.
- [ ] No Test Case or DSL definition was edited during the transform.
- [ ] The application facade signatures, returned events/DTOs, and signalled
      errors are unchanged (or changes were explicitly approved and recorded).
- [ ] The consistency boundary is unchanged (or the change was explicitly approved
      and recorded).
- [ ] The frozen scenarios are green through the `application`-level driver
      against the new idiom.
- [ ] The final summary names which behaviors are covered and notes that uncovered
      paths are not proven.

## Do not

- Do not start a restyle against a red or missing test baseline.
- Do not edit a Test Case to make the new idiom pass.
- Do not change the public contract or the consistency boundary as an unrecorded
  side effect.
- Do not describe green tests as a total correctness guarantee.
