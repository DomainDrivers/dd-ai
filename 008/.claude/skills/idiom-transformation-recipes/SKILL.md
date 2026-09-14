---
name: idiom-transformation-recipes
description: The step-by-step recipe for moving a module from an object-oriented rich aggregate to a decider in the functional core / imperative shell flavor. States the precondition, the mechanical steps, what stays invariant, and how to verify. Use during the swap step of a restyle.
---

# Recipe: Rich Aggregate → Decider (Functional Core / Imperative Shell)

The recipe assumes the behavior contract is already frozen (a green
`application`-level driver exists — see `four-layer-test-architecture`) and that the
invariant to preserve is known (see `idiom-catalog` and
`behavior-preservation-contract`). It changes the **System Under Test** and adds a
**Protocol Driver**; it never edits Test Cases or the DSL.

Shape:
1. **Precondition** — what must be true before starting.
2. **Steps** — mechanical, reversible edits.
3. **Invariant** — what must read identically before and after.
4. **Verify** — re-run the frozen scenarios; green or roll back.

---

## Precondition

The aggregate already **returns events** from its command methods (or its writes
can be expressed as the events it already defines), and its consistency boundary
is **one load / one save** of a single unit. If either is false, stop — the swap's
invariant cannot be stated.

## Steps (FCIS flavor — the default)

1. **Name the state.** Keep the existing entity as the state carrier but treat it
   as **immutable**: same fields, constructed whole, no in-place setters. If it is
   a persistence entity, it stays the persistence type; immutability is about how
   the domain logic treats it, with the shell re-saving a new instance. A control
   field the store owns (an optimistic-lock version, a sequence counter) is carried
   **unchanged** into that instance under the same identity, so the store updates the
   same row instead of writing a second one.
2. **Name the commands.** One immutable command type per command method, named
   after the operation and carrying that method's parameters, as a sealed/closed
   set where the language supports it.
3. **Reuse the events.** The aggregate's existing event types are the decider's
   output. **Do not invent new events or add fields** — that would be a behavior
   change, not a restyle.
4. **Write one decision function per command:**
   `decide(state, command) -> Decision(nextState, optional event)`.
   - Move each command method's guard **verbatim**, reading the `state` parameter
     instead of `this`.
   - Compute the **next immutable state directly** inside the function — this is
     the mutation that used to happen in place, now expressed as building a new
     value.
   - Return the event the operation produced, or **no event** where the old method
     returned "nothing happened" (the old `Optional.empty()`).
   - Where the old method **threw** on a rejection, keep throwing the same error
     type, and let the shell propagate it unchanged. A thrown rejection and an
     empty result are different observable outcomes; keep each one as it was.
   - **No separate `evolve`, no fold over events.** The next state is part of the
     `Decision`, not rebuilt from events.
5. **Keep the core pure.** No repository, no clock, no publisher inside `decide`.
   Any value the guard needs is read from `state` or the command.
6. **Add the imperative shell** in the application facade:
   `load state → decide(state, command) → save(nextState) → publish(event)`.
   Facade signatures stay identical. Watch for a command method whose old version
   relied on in-place mutation being persisted implicitly (e.g. ORM dirty-checking)
   and therefore lacked an explicit save — once state is immutable that path
   silently stops persisting, so the shell must add the `save`. The safety net
   catches this as a red scenario; the fix is **shell wiring**, never a Test Case
   edit. Where the old path never persisted at all — a store without
   dirty-checking, an in-memory repository — that `save` repairs a **latent
   defect**: the effect a caller sees afterwards is new. Name it as such and record
   it at the swap gate instead of filing it as plain wiring.
7. **Add a `domain`-level driver** binding the DSL to `decide` over an in-memory
   state, for fast feedback on the new core (the "lower" move).

## The fork you must not take silently

The strict event-sourced decider splits this into
`decide(state, command) -> events` and `evolve(state, event) -> state`, with the
events self-contained so `evolve` can rebuild state from them alone. If an existing
event is *thin* (carries an id but not the full data `evolve` would need), the
strict form forces **enriching that published event** — a public-contract change
(`behavior-preservation-contract`). This recipe keeps the events unchanged and
folds the next state inside the core's `Decision` (the FCIS flavor). Choosing the
strict form instead is an explicit, approved, recorded contract change — surface it
at the target-confirmation gate; do not slip into it.

## Invariant

For the same command and starting state, the decider returns the **same event types
with the same domain payload** the aggregate emitted (identity and timestamp fields
excluded; no new fields) and signals the **same errors** on the same rejections;
the next state is observationally equal to the post-mutation aggregate; the
**consistency boundary** (one unit loaded once, saved once) is unchanged.

## Verify

The frozen scenarios pass through the `application` driver (unchanged facade
contract) and, once added, the new `domain` driver. Green or roll back.

## Do not

- Do not invent new events, outcomes, or fields — that is a behavior change.
- Do not move I/O into the core; `decide` stays pure.
- Do not adopt the strict event-sourced `decide`/`evolve` form without an explicit,
  recorded contract-change approval.
- Do not delete the old aggregate command methods before the decider is green
  through the frozen scenarios.
- Do not widen or narrow the consistency boundary as a side effect.
