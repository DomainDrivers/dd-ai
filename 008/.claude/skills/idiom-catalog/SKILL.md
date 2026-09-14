---
name: idiom-catalog
description: The two implementation idioms this tool moves between — the source (an object-oriented rich aggregate that mutates state and returns events) and the target (a decider written as a functional core with an imperative shell). Gives the signals that confirm the source, the shape of the target, and the one invariant the swap must preserve. Use to confirm a module really is a rich aggregate before a restyle and to keep the decider's invariant in view during the swap.
---

# The Two Idioms: Rich Aggregate → Decider (Functional Core / Imperative Shell)

This tool moves a unit of behavior from an **object-oriented rich aggregate** to a
**decider** hosted as a **functional core / imperative shell**, without changing
what the unit does. This file describes those two idioms — the source to recognize,
the target to build — plus the single invariant the swap must hold.

Use it two ways:
1. **Confirm the source.** Read the module's methods and check they match the
   rich-aggregate signals below. If they do not, this tool does not apply — stop
   and say so.
2. **Hold the target's invariant.** Keep the decider's "invariant to preserve" at
   the center of the swap.

---

## Why move from the source to the target

The swap is worth doing when you want the decision logic **testable in isolation**
(a pure function over values, with no framework to stand up), when an
**event-sourced future** is plausible (the decision core already speaks in events),
or simply to **isolate the decision from its side effects** so each can change
independently. It is only ever undertaken as a **behavior-preserving** change: the
goal is a better shape for the *same* behavior, proven by the same tests staying
green (see `behavior-preservation-contract`). If none of those motivations applies,
the rich aggregate is fine as it is — do not restyle for its own sake.

---

## Source idiom: `rich-aggregate` (object-oriented)

Behavior and private mutable state live in one entity. Command methods enforce
invariants and mutate `this`, returning domain events as the result of the
operation.

- **Signals (look for these together):** an entity/class with private mutable
  fields; public command methods that assign those fields in place; invariants
  checked inside the methods; the methods return events
  (`Optional<Event>` / `List<Event>`); persistence happens by the surrounding
  framework observing the mutated entity (e.g. ORM dirty-checking) rather than by
  an explicit save.
- **Why this is the source here:** it already emits events, so a decider's output
  contract already exists; and its state is a single load/save consistency unit,
  which the swap must keep intact.
- **Not a match — stop:** a data holder with the logic in a separate service (no
  behavior on the entity); a stateless mapping with no events; pure CRUD with no
  real decisions. None of these is a rich aggregate, and none is what this tool
  converts. Say the tool does not apply rather than forcing the move.

## Target idiom: `decider` as functional core / imperative shell

The decision logic becomes a **pure function** over an **immutable state**; all
I/O is pushed out to a thin **shell** around it.

This tool builds the **functional core / imperative shell (FCIS) flavor** of a
decider — not the strict event-sourced form. The distinction is fixed by default
and matters:

- **FCIS flavor (the target) — the `decide-and-transition` form:** one decision
  function per command,
  `decide(state, command) -> Decision(nextState, optional event)`. It computes the
  **next immutable state directly** and returns it alongside the event the
  operation produced. The transition lives inside `decide`; there is no fold over
  events to rebuild state. Name it that way when you describe the result: the
  canonical decider of the literature is the pair `decide` + `evolve` — the strict
  form below. The shell runs `load → decide → save(nextState) → publish(event)`.
- **Strict event-sourced form (deliberately not used):** two functions —
  `decide(state, command) -> events` and `evolve(state, event) -> state` — where
  the events alone must carry enough data for `evolve` to rebuild state. Adopting
  it typically forces **enriching the published events** with fields they do not
  carry today, which is a public-contract change. That is out of scope by default;
  choosing it is an explicit, recorded contract change (see
  `behavior-preservation-contract` and the recipe's "fork" note).

- **Signals of a correct target build:** an immutable state type; one pure
  decision function per command, returning `(nextState, optional event)`; no
  repository, clock, or publisher access inside the core; the only place I/O
  happens is the shell.
- **Invariant to preserve across the swap:** for the same command and starting
  state, the decider returns the **same event types with the same domain payload**
  the aggregate emitted (identity and timestamp fields excluded; no new fields),
  signals the **same errors** on the same rejections, and keeps the **consistency
  boundary unchanged** (the same one unit is loaded once and saved once).
- **Fits:** an event-emitting consistency unit with real decision logic —
  maximally testable, side effects isolated at the edges.
- **Hurts:** trivial CRUD with no genuine decisions; there is nothing to lift into
  a core.

---

## Classification rule

Read the **methods**, never the file name: what each one mutates, what it returns,
where I/O happens. The only classification this tool makes is binary — *is the
module a rich aggregate that emits events, yes or no.* If something else is mixed
in (e.g. a pure transformation hidden inside the aggregate), note it as a possible
follow-up, but the decision to proceed depends only on whether the rich-aggregate
signals dominate.

## Do not

- Do not classify by file name. Read what is mutated, returned, and where I/O
  lives.
- Do not silently adopt the strict event-sourced `decide`/`evolve` form — it forces
  an event-contract change and must be an explicit, recorded decision.
- Do not proceed if the source is not a rich aggregate emitting events; the swap's
  invariant ("same events") cannot be defined otherwise.
