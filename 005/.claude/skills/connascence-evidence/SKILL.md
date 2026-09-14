---
name: connascence-evidence
description: 'The source-reference contract for a connascence analysis — every claim the tool makes (a finding exists, the kind assigned to it, a degree count, a cost) must carry at least one source-ref: a `kind:locator` pointer to a concrete, checkable artifact. The kind set is OPEN — code, map, contract, doc are common examples, and a trace, schema, ADR, test, or ticket count just as well. The source-ref itself shows how solid a claim is: a reader weighs it by what it cites. A claim with no source is an explicit assumption, surfaced at a gate for confirmation. Stack-agnostic. Referenced by the command, the scanner agent, connascence-taxonomy, and connascence-scoring.'
---

# Connascence evidence (source-reference contract)

Every claim carries **at least one source-ref** — a pointer to the artifact that
makes the claim checkable by hand. This keeps the score transparent: a reader
can follow any row, any number, and any decision back to the source it came from.

## What counts as a claim (all of these must be sourced)

- a seam element exists and crosses the boundary,
- the **kind** assigned to it (Name / Type / … / Identity),
- the **degree** — each counted site is itself a source-ref,
- the **cost** — derived, so it cites the refs behind its strength and degree.

## Source-ref format and common kinds (an open set)

A source-ref is a short **`kind:locator`** pointer plus, where useful, the
decisive fragment. The **kind is a free label** — it only has to name what sort
of artifact the locator points into, so a reader can find it. The kinds below are
**common examples**; use whatever names the real artifact.

| kind (example) | points into | example ref |
|---|---|---|
| `code:` | a source line | `code:src/orders/OrderService.x:62` |
| `map:` | a mermaid context-map node/edge | `map:context-map.md:42 (Orders-->\|charges\|Billing)` |
| `contract:` | an interface / schema / event-spec line | `contract:billing.api.md:10` |
| `doc:` | a quoted requirement / interview / note | `doc:requirements.md:5 "returns false when already paid"` |
| `trace:` / `log:` | a captured runtime observation | `trace:checkout-span.json:#88` |
| `search:` | the search that produced a list of sites, so the list can be replayed | `search:word="charge(" scope=src/orders minus=import → 3 production, 1 import, 2 test` |
| `derived:` | the other refs it was computed from | `derived:[code:…:62, code:…:79]` |
| …anything else | whatever artifact attests the claim | `adr:0007.md:12`, `test:OrderIT.x:140`, `ticket:PROJ-321` |

Quote the smallest decisive fragment next to the ref.

## The rule

1. Every claim lists **≥ 1** source-ref.
2. A **degree** of N lists N distinct source-refs (or a sourced range when the
   source describes intent rather than enumerable realized sites), and matches the
   refs behind it. A list read off code also cites the `search:` that produced
   it, so a second run replays the search and lands on the same list.
3. A **derived** claim — a **cost**, or any number computed from findings — cites
   the refs it was computed from.
4. A claim with **no** source is written as an explicit `assumption: …`. It is
   listed with the findings, carries cost `—`, stays out of the counts and the
   ranking, and is surfaced at the next gate for the user to confirm or drop.
5. When two sources describe the **same** element differently — the map shows an
   event, the code shows a synchronous call — the row lists **both** refs. The
   code ref sets the realized kind, and the divergence is noted beside it.

## How solid a claim is — read it from the source-ref

Each claim's trustworthiness is read directly off **what it cites** — that is the
whole point of the mandatory ref. A claim backed by source code or a captured
trace is as solid as that artifact; a claim backed by a context-map edge or a
requirement sentence is as solid as that design intent. The reader judges from
the ref, which is always present.

Express uncertainty **concretely, next to the ref**:

- when the cited artifact cannot enumerate real sites (a diagram, a prose
  description), give **degree as a range** and let the ref show why;
- when a claim leans on a sketch or an unstable contract, add a **one-line
  caveat** beside its ref;
- a **`derived:`** claim is exactly as solid as the weakest ref it cites, so it
  simply carries those refs.

A claim with no source is an explicit `assumption:` — listed with the findings at
cost `—`, but outside the counts and the ranking until the user confirms it at a
gate. Keep every claim's refs visible and present findings per-claim, so a reader
can weigh solid and speculative claims by them.
