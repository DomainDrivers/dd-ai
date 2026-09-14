---
name: connascence-taxonomy
description: 'The vocabulary for grading coupling between two modules — the nine kinds of connascence (Name, Type, Meaning, Position, Algorithm, Execution, Timing, Value, Identity) ordered weakest-to-strongest, the two properties every row carries (strength, degree), the one-agreement-one-kind-one-row splitting rules with a deterministic test per kind, and the boundary rule that strong connascence must stay inside a module while only weak low-degree connascence may cross. Stack-agnostic and domain-agnostic. Use to cut a seam into rows, whether the evidence is code or a design artifact (context map, contract, or prose).'
---

# Connascence taxonomy

Two elements are **connascent** when a change to one forces a matching change to
the other to keep the system correct. This skill is the shared vocabulary for
naming and ranking that relation across a module boundary, and the rules that
cut a seam into rows the same way on every run. All examples here use generic
placeholders (`module-a`, `Orders`, `Billing`) standing in for the pair under
analysis.

## The nine kinds, weakest → strongest

The order is the **strength rank** (1 = weakest, 9 = strongest). Use the rank as
the strength score in `connascence-scoring`. The letter in brackets is the row-id
prefix.

### Static — visible by reading the code or the contract

1. **Name (CoN)** `[N]` — both sides must agree on a name.
   *Example:* `Orders` calls `billing.charge(...)`; rename `charge` and `Orders`
   breaks. The weakest kind; almost unavoidable and usually fine.

2. **Type (CoT)** `[T]` — both sides must agree on a type.
   *Example:* `Orders` passes a `Money` where `Billing` expects `Money`. Change
   the type on one side and the other must follow.

3. **Meaning / Convention (CoM)** `[M]` — both sides must agree on what a value
   *means*, beyond its type.
   *Example:* `charge(...)` returns a `boolean` where `false` secretly means
   "already charged". Or: an `ownerId` field is "really" a customer id by
   convention. The type does not enforce the meaning — a silent break.

4. **Position (CoP)** `[P]` — both sides must agree on an order.
   *Example:* `reserve(itemId, quantity, customerId)` — swap two arguments of the
   same type, or reorder a record's fields / CSV columns, and correctness breaks
   with no type error.

5. **Algorithm (CoA)** `[A]` — both sides must run the *same* algorithm.
   *Example:* both modules compute the same checksum, hash, sort order, or
   id-encoding round-trip independently; change it on one side only and they
   disagree.

### Dynamic — only visible at runtime (generally stronger: no static check catches a break)

6. **Execution (CoE)** `[E]` — the *order* in which two operations run matters.
   *Example:* `module-a` must call `prepare()` before `commit()`; reorder and the
   result is wrong even though both calls are individually valid.

7. **Timing (CoTm)** `[TM]` — the *time* at which things run matters.
   *Example:* a token is valid only within a window; a reaction must land before
   a deadline; a retry must wait a backoff.

8. **Value (CoV)** `[V]` — several values, possibly in different stores, must
   change together to stay consistent.
   *Example:* `module-a` records a slot as "taken" while `module-b` records the
   matching booking; if the two stores drift, the system is wrong. A cross-module
   invariant.

9. **Identity (CoI)** `[I]` — both sides must refer to the *same* live instance,
   not merely equal values.
   *Example:* two modules must share one and the same counter / lock / aggregate
   instance, not two copies that happen to be equal.

## The two properties of every row

- **Strength** — the rank above (1–9). Higher = harder and riskier to change.
- **Degree** — how many distinct places are bound by this agreement. A name
  shared in 30 call sites is far worse than the same name in 2.

Every row in this analysis **crosses** the A↔B boundary: one end of the
agreement sits in A, the other in B. That is what the boundary rule judges.
An agreement with both ends inside one module is cohesion — good design, and
the subject of a different analysis.

A row matters most when it is **strong and high-degree**: the cost formula
weighs strength and degree together, so a weak kind bound in many places can
outrank a strong one bound in a single place.

On top of these two properties, every row carries a **source-ref** per
`connascence-evidence` — the artifact the classification is read off (a code
line, a context-map node/edge, a contract line, a quoted note — or any other
concrete artifact; the kind set is open). Assign a kind by pointing at the source
that shows it; a row with no source is recorded as an explicit `assumption:`.

## The boundary rule

1. **Maximize** connascence **within** a module (keep things that change
   together together).
2. **Minimize** the connascence that **crosses** the boundary.
3. Of what must cross, keep it **weak** (low rank) and **low-degree**.

Restated as the test this tool applies: *the further apart two connascent
elements live, the weaker their connascence must be.* A module boundary is an
"apart", so a strong, high-degree, crossing connascence is the signal that the
boundary is leaky — or fictional.

## One agreement, one kind, one row

A **row is one agreement of exactly one kind**. A fact in the code often carries
several agreements; each becomes its own row with its own degree and its own
source-refs. Splitting is what makes two runs land on the same rows: each shape
of evidence has one fixed decomposition, so the cut is the same on every run.
The rules below say, for each shape of evidence, which rows it yields.

### R1 — an operation call

An operation one side calls on the other (a method, an endpoint, an event the
other side consumes) yields one **N** row. Degree = its production call sites on
the calling side. Its parameter and return types yield **T** rows under R2. A
`void`, primitive, or platform return (`boolean`, `int`, a collection, an
option/maybe wrapper) yields no T row.

### R2 — a type whose values cross

Each type whose values cross as an argument, a return, a payload field, or an
injected instance yields **one T row for the whole seam**, listed once no matter
how many operations carry it. Shared-kernel types (fixed at Step 0) yield no
row. An injected instance (a facade, a client, a port) is a T row whose degree is
the number of receiving components. In a dynamically typed stack, or over the
wire, the "type" is the value shape the receiver assumes — the fields it reads,
the operations it calls — and the T row binds the sites that assume it.

### R3 — Meaning

An **M** row exists when the source shows a value whose correct use depends on
an agreement its type does not state, **and** the other side acts on that
agreement. The test is two citations: the line where the meaning is produced or
defined, and the line where the other side relies on it. A **rely site** is a
production line on the other side that consumes the value under the agreement
— branches on it, compares it, looks it up, or stores it — whether or not that
line names the convention: a caller that compares a raw value against a
normalized return relies on the normalization even though it never mentions
it. Concretely:

- a return value where the callee maps **≥2 distinct causes** to one value
  (`false` for "not found" and for "rejected"; `empty` for "nothing matched"
  and for "nothing left"; a default returned on a miss) and the caller
  branches on it;
- a field or id that carries a value **of another type's meaning** (a bare
  id that is really a customer id; a `String` that is really a currency);
- a magic value, a unit, an "empty means all" rule.

Without both citations there is no M row.

### R4 — Position

A **P** row exists only where **adjacent same-typed** parameters, fields, or
columns could swap silently. Where distinct types guard the order, the type
check catches a swap: no P row (the T rows already cover it).

### R5 — Algorithm

An **A** row exists when a computation is **performed on both sides**, or
performed on one side **to match** a form the other side stores or returns (an
id encoding, a checksum, a rounding rule, a normalization). The row is the
computation; the agreement about *which* value equals *which* is a separate M
row under R3.

### R6 — Execution

One **E** row per **unit of work** that spans the seam: a transaction-bearing
method, a `BEGIN…COMMIT` block, a saga step, whose body both calls the other
side and mutates its own state, or whose calls to the other side must run in a
fixed order. The row is the order; the consistency it protects is a V row under
R7.

### R7 — Value

One **V** row per **pair of stores** (tables, aggregates, files) that must agree
after an exchange completes. The row exists whether or not an E row maintains
it; its enforcement (which E rows, a foreign key, a reconciliation job, or
`none`) is written on the row.

### R8 — Timing

One **TM** row per window, deadline, retry policy, TTL, or redelivery window
that both sides must honor.

### R9 — Identity

One **I** row per **live instance** both sides hold — one object, connection,
lock, transaction manager, or aggregate instance. Two equal copies are an M row
under R3, never an I row.

## Reading the kind off the evidence

Each reading below must cite the **source-ref** it is read from (per
`connascence-evidence`) — the call site, the map edge, the contract line, or the
quoted sentence that shows the connascence.

- A shared **name/symbol** crossing the boundary → Name (R1).
- A **type** that must match → Type (R2).
- A **value whose meaning is not in its type** → Meaning (R3).
- **Same-typed neighbors** that could swap → Position (R4).
- A **computation repeated or reproduced** → Algorithm (R5).
- A **required call order** or a **unit of work** → Execution (R6).
- A **consistency invariant spanning two stores** → Value (R7).
- A **window/deadline/backoff** → Timing (R8).
- A **shared single live instance** → Identity (R9).

### When the seam runs over the wire

An HTTP, RPC, or message seam shares no symbol to search for, so read the kinds off the
payload and the protocol instead:

- an endpoint path, an operation name, a topic or queue name → **Name**,
- a field's declared type and the required/optional shape of the schema → **Type**,
- a status string, an enum carried as text, an assumed unit or currency, an
  "empty means all" rule → **Meaning**,
- element order in a positional payload, tuple-shaped or CSV rows → **Position**,
- an id encoding, a signature, a checksum, or a total both sides recompute →
  **Algorithm**,
- a required order between endpoints or between event types → **Execution**,
- a timeout, a retry policy, a TTL, an at-least-once redelivery window → **Timing**,
- two stores that must agree once the exchange completes → **Value**.

Source-refs here point at the schema, the contract, or a captured exchange
(`contract:openapi.yaml:120`, `trace:checkout-span.json:#88`), not at a call
site. Degree is the number of distinct producing or consuming sites, counted
the same way.
