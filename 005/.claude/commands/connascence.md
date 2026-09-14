---
description: Grade the coupling between two modules (or two contexts) at the resolution of connascence. Works on whatever evidence exists — source code, a context map, a proposed contract, or a plain description — so you can grade a boundary even while still designing the contexts; every claim cites the concrete source it rests on, and that source-ref shows how solid each claim is. Runs as three numbered steps, each ending in a manual gate. Project-agnostic and stack-agnostic; drop it into any project.
argument-hint: "[two modules/contexts to compare — names, paths, a context-map edge, or a sentence describing how they integrate; e.g. module-a and module-b]"
---

# /connascence

Input: $ARGUMENTS

Grade the boundary between **two modules** (call them A and B). For every
agreement that crosses the boundary, name its kind of connascence, its strength,
its degree, and its cost, each backed by the source-ref it rests on. The output
is a per-row table — one row per agreement — plus a Boundary Coupling Profile,
ranked offenders, and a unitization ledger derived from it, so a reader can
recompute every number and see every judgment.

This command **carries no project facts**. It discovers everything about A and B
at runtime and confirms each step with you at a gate before moving on.

Load the `connascence-evidence` skill (the source-reference contract), the
`connascence-taxonomy` skill (the nine kinds and the one-agreement-one-row
rules), and the `connascence-scoring` skill (site rules, degree rules per kind,
the cost formula, the profile, the ledger) now — Step 1 already applies all
three.

**Evidence rule (applies to every step).** Per `connascence-evidence`, every
claim this command makes — a row, its kind, its degree, its cost — carries at
least one **source-ref**: a `kind:locator` pointer to a concrete, checkable
artifact (`code:`, `map:`, `contract:`, `doc:`, `search:`, or any other kind that
names the artifact — the set is open). A claim with no source is written as an
explicit `assumption:` and surfaced at the gate for confirmation.

**Determinism rule (applies to every step).** Two runs on the same evidence must
produce the same rows and the same degrees. Every cut into rows follows a
numbered rule from `connascence-taxonomy`; every degree follows a numbered rule
from `connascence-scoring` and is backed by a replayable search. Wherever a rule
still needed a judgment (which method is the localizing helper, which store pair
forms an invariant), the judgment is written into the **unitization ledger**.

---

## Step 0 — Frame & inventory the evidence

If a `connascence-<a>-<b>.frame.md` already exists for this pair, read it and
present it at the gate as the proposed frame. Otherwise build it:

1. Identify A and B from the argument, then **bind each to a concrete
   membership** — the path/package set (for code) or the node set (for a map)
   that defines what sits *inside* each side. List the membership explicitly;
   what counts as *crossing* in Step 1 is defined entirely by this line.
2. Inventory what evidence exists for each side: source code? a context map? a
   written contract or interface? an event list? only prose? Give each kind of
   artifact one **evidence label** (`code:`, `contract:`, `schema:`, `wiring:`,
   `map:`, …) and keep it for the whole run.
3. Note what each source attests (the realized system vs intended design) so each
   claim can be weighed by its ref.
4. Propose the **shared kernel**: the types both sides carry by design (a common
   `Money`, a shared id type). Step 1 keeps these out of the Type rows, so the
   list belongs to the user, not to a later judgement call.
5. Fix the **site rules** for this codebase, each as a pattern read off the
   stack in use: the path pattern that marks test sources; the line pattern
   that marks import/using/require lines; the marker that identifies a unit of
   work (an annotation, a `BEGIN…COMMIT` block, a saga step); the **write
   marker** (the operations that persist state — an ORM save, a SQL statement,
   a document put, a file write); and the **handle types** through which code
   reaches a framework-provided shared instance (a connection, a session, a
   client, a repository base type). Step 1 applies these mechanically, and a
   different stack changes only this list. When the evidence is a map, a
   contract, or prose, record each pattern as `n/a` — degrees then come from
   the artifact's elements or as ranges per `connascence-scoring`.

When the only evidence is a map, a contract, or prose, say so here: degrees will
come as ranges and several kinds will close as sourced absences. A low total
then reports thin evidence, and it takes code or a trace to read it as loose
coupling.

**Gate.** Report: the pair, each side's frozen membership, the evidence and its
labels, the shared kernel, and the site rules — and resolve any ambiguous member
(a sub-package that could belong to either side) before continuing. Stop for
confirmation or correction. After approval, offer to write the frame to
`connascence-<a>-<b>.frame.md` (project root by default) so a re-run starts from
the same frame. Write it only to a path the user approves.

## Step 1 — Inventory the seam, one agreement per row

List every agreement that crosses the A↔B boundary. A **row is one agreement of
one kind** per `connascence-taxonomy` — Name, Type, Meaning, Position,
Algorithm, Execution, Timing, Value, Identity — with a stable id built from the
kind and an ordinal in source-ref order: `N1, N2, …, T1, …, M1, …, P1, …,
A1, …, E1, …, TM1, …, V1, …, I1, …`. Carry the same id into Step 2.

**Where to look.** The eight shapes below are the search checklist; each shape
yields rows of the kinds listed. Search every shape deliberately — dynamic
coupling has no symbol to search for, so it is only found when looked for:

| shape | rows it yields |
|---|---|
| an **operation** one side calls on the other (including an event one side publishes and the other consumes) | one **N** row; one **T** row per crossing parameter/return type (shared kernel excluded, counted once per type across all operations); one **M** row when the return or a field is overloaded |
| a **type or id** whose values cross | one **T** row; plus an **M** row when its value carries a meaning its type does not state; plus an **A** row when an encoding is reproduced to match it |
| a **convention** with no carrier (a magic value, a rounding rule, a column order) | one **M**, **A**, or **P** row |
| a **unit of work** spanning the seam | one **E** row per unit of work (a transactional operation, a saga step, a handler that calls across and then commits) |
| a **cross-store invariant** | one **V** row per store pair that must agree |
| a **shared store / table** both sides read or write | one **V** row (the rows both sides write) and, when both hold one live handle, an **I** row |
| a **shared instance** both sides hold | one **I** row |
| a **wiring / injection edge** | one **T** row whose degree is the number of receiving components |
| a **window, deadline, retry, or TTL** | one **TM** row |

**Splitting rules** (from `connascence-taxonomy`, applied every time):

- A fact that carries several agreements is several rows. A transaction that
  orders calls and protects consistency is an **E** row and a **V** row. A
  `boolean` whose `false` means two things is an **N** row for the call and an
  **M** row for the meaning. A carrier id with an encoding round-trip is a
  **T** row, an **M** row (which value equals which), and an **A** row (the
  encoding) — each with its own degree.
- Each type is one **T** row for the whole seam. An operation adds a **T** row
  only for a type not yet listed.

**Degree is counted, not estimated.** A row's degree is the number of distinct
production sites bound by it, one site being one `path:line`, following the
per-kind degree rules and the site rules in `connascence-scoring`: production
only, import lines out, unreferenced declarations out, one line one site, and a
line counts on every row it binds. The
list of sites *is* the degree; write the list. Every counted list cites the
`search:` ref that produced it, so a reader replays the search and lands on the
same list. Give a sourced range only when the source is a diagram or a
description. Test sites are listed as `+N test` outside the costed degree.

**Standard columns**, one table per kind, in kind order:

`id | agreement | how it crosses | degree | source-ref(s) | search`

If code sources exist and the codebase is non-trivial, dispatch the
`connascence-scanner` agent (read-only) to run the search recipe and return raw
per-site rows with `code:` source-refs and the searches it ran — this keeps the
heavy searching and reading out of the main context; then fold its hits into rows by
kind, keeping every site ref so degree stays countable. Without code (only a
map, contract, or description), or for a small, obvious seam, inventory inline.

**Completeness self-check (before the gate).** Resolve every one of the nine
kinds explicitly: either list ≥1 row, or write a one-line **sourced
absence** — `none — checked <locator>, <why>` (e.g. `Execution: none — no unit
of work spans the seam; checked code:…`). Do the same for the **unused
surface**: operations one side exposes that the other never calls in production.

**Gate.** Present all nine kinds in kind order — rows or sourced absences — with
the site rules stated once at the top, then the unused surface. Stop so the user
can add what was missed or remove false hits before the expensive step.

## Step 2 — Cost, profile, ledger

For each confirmed row, cost it per `connascence-scoring`. Build one table, one
row per agreement, with these columns:

`id | agreement | kind | strength(1–9) | degree | cost | source-ref(s)`

- **cost** = `strength_rank × log2(1 + degree)`, rounded to two decimals,
  shown so the reader can recompute it (a `derived:` claim citing the refs behind it). An `assumption:`
  row has no realized degree to cost — it stays in the table with cost `—`, and
  is left out of the profile counts and the ranking until the user confirms it
  at the gate.
- From **8 refs up**, keep the row readable: show the first three, then
  `+K more → appendix`, and list the full set under the row's id in a **Degree
  appendix**. The degree stays the length of the full list.

Sort the table by cost, highest first; break ties per the ordering rule in
`connascence-scoring` (source-refs compared one by one in path-then-line order,
then row id), so every row lands in the same place on every run.

Then add four views of that same table, derived entirely from its rows:

1. The **Boundary Coupling Profile** — the rows bucketed by strength band
   (static-weak 1–2 / static-semantic 3–5 / dynamic 6–9), with count, total
   cost, and share per band, each citing the rows behind it.
2. The **ranked offenders** — the rows by cost, highest first, each
   with its cost and **source-ref(s)**.
3. The **unitization ledger** — one line per judgment the rules left to the
   analyst: which method was credited as a localizing helper, which declaration
   was excluded as unreferenced, which stores form a Value pair, which
   sub-package was placed on which side. Each line names the rule applied and
   the row it affects. A reader who disagrees with a line can recompute the
   affected rows from it.
4. **What to read across runs** — the set of top offenders and the band shares
   are the stable reading of the seam; the total is an ordering inside this one
   report. State both in one sentence each.

**Gate.** Present the table, the profile, the ranked offenders, and the ledger.
Stop for review and correction.

Offer to write a single report (`connascence-<a>-<b>.md`, in the project root by
default) containing the frame, the per-row table, the profile, the ranked
offenders, the ledger, the degree appendix, and the unused surface. Write it
only to a path the user approves.

**Done.**

---

## Conduct

- Work one gate at a time; advance only after approval.
- Every claim carries ≥1 `source-ref` per `connascence-evidence`; the ref shows
  how solid the claim is — a reader weighs it by what it cites. A claim with no
  source is an explicit `assumption:`.
- One agreement, one kind, one row. Every cut follows a numbered rule; every
  residual judgment is a ledger line.
- Every degree is a written list of sites backed by a replayable search.
- Every row crosses the A↔B boundary; the boundary rule weighs strong,
  high-degree rows. Connascence with both ends inside one module is cohesion —
  a different analysis.
