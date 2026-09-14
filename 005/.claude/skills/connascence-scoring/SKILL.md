---
name: connascence-scoring
description: 'Score a connascence seam — the site rules (what one site is; production only; import lines out; unreferenced declarations out), the degree rule per kind (call sites for Name, naming sites for Type, producer + consumer sites for Meaning/Position/Algorithm, sequenced calls for Execution, write sites on both sides for Value, receiving components for Identity), the cost formula (cost = strength_rank × log2(1 + degree)), the Boundary Coupling Profile, the ranked offenders, the unitization ledger, and what a reader may compare across runs. Stack-agnostic. Use after cutting the seam into rows with connascence-taxonomy.'
---

# Connascence scoring

Input: the seam table from `connascence-taxonomy` (one row per agreement, each
with one kind, strength rank, and **source-ref(s)**). Output: that
table with a **degree** and a **cost** per row, plus the views derived from it —
a Boundary Coupling Profile, the ranked offenders, and the unitization ledger.
Every number traces back to named rows and their source-refs (per
`connascence-evidence`), and every degree traces back to a replayable search.

## Site rules (S1–S6)

A degree is a count of **sites**. These rules fix what a site is, so two runs
count the same list.

- **S1 — one site = one production line.** A site is one distinct `path:line`
  in production sources. Two references on the same line are one site; the same
  call made in three methods is three sites. A whole file, class, or module is
  never one site. One line is a site on **every row whose agreement it binds**:
  a line that calls `B.of(x)` is a site for the Name row of `of` and for the
  Type row of `B`; charging it on one row never removes it from another.
- **S2 — production only.** Sources under the test path pattern fixed at Step 0
  are excluded from the costed degree and listed separately as `+N test`.
- **S3 — import lines out.** Lines matching the import/using/require pattern
  fixed at Step 0 are never sites: removing a use removes its import with it, so
  counting both charges one binding twice.
- **S4 — unreferenced declarations out.** A declaration with zero production
  references (a method nobody calls, a constructor nobody invokes) is never a
  site. List it under **unused surface** or as a by-product.
- **S5 — a helper localizes.** When one side reaches an agreement through a
  single method of its own (a helper that performs an encoding, a wrapper that
  makes the call), that method's definition is **one site** and its callers are
  zero sites for that row. The callers still count for the rows they carry
  themselves. Naming the helper is a ledger line.
- **S6 — enumerate, don't estimate.** A degree read off code is the **list** of
  sites, produced by the fixed search the row cites as `search:<command>` per
  `connascence-evidence` — whole-word, over the other side's membership, import
  lines dropped by the Step-0 pattern, S4 exclusions named. The same symbol
  gives the same search on every run. Write the list; the degree is its
  length. If the set
  cannot be pinned (a diagram, a prose description), give a **range** tied to
  the sites you did enumerate — never a bare number. A list too long for its
  row moves to an appendix keyed by the row's id, with the first three refs
  left on the row.

## Degree rule per kind (D1–D9)

The rules speak in code terms. When the evidence is a map, a contract, or
prose, the same rule counts the artifact's elements instead — edges, contract
lines, quoted sentences — or gives a range per S6 when they cannot be
enumerated.

| kind | degree = the number of… | counted on |
|---|---|---|
| **D1 Name** | production call/reference sites of the name | the side that does not own the name |
| **D2 Type** | production sites naming the type (S3 applies); for an injected instance, the components that receive it, read off wiring | the side that does not own the type |
| **D3 Meaning** | sites that produce or define the meaning **plus** sites that rely on it | both sides |
| **D4 Position** | sites that produce the order plus sites that consume it | both sides |
| **D5 Algorithm** | sites that perform the computation (S5 applies: a helper is one site) | both sides |
| **D6 Execution** | crossing calls the unit of work sequences (distinct call sites inside its body) | the side that owns the unit of work |
| **D7 Timing** | sites that set or check the window/deadline/retry | both sides |
| **D8 Value** | production lines matching the **write marker** fixed at Step 0 (the operations that persist state in this stack) against either store — every write to the store counts, whichever fields it touches | both sides |
| **D9 Identity** | components that receive the instance, read off wiring or construction; when the instance is framework-provided and has no wiring line, the components that declare one of the **handle types** fixed at Step 0 (the types through which code reaches that instance), found by searching those types over both memberships — every such component counts, whatever it does with the handle | both sides |

A **shared-kernel** type (fixed at Step 0) is sanctioned coupling: it yields no
row and no degree.

## Cost per row

```
cost = strength_rank × log2(1 + degree)
```

Round to two decimals, half up.

- `strength_rank` is the row's kind rank 1–9. The rank **orders** the kinds; it
  does not measure them, so rank 4 is not twice rank 2. Read a cost as a
  heuristic ordering inside one seam: it ranks the rows and shows where the
  weight sits.
- `degree` is the list length from D1–D9 and equals the number of distinct
  `source-ref`s behind it. A range degree gives a range cost: keep both bounds
  on the row, and use the **upper** bound to total a band and to place the row
  in the ranking, so a range never sorts below a point cost it may exceed.
- `log2(1 + degree)` so degree matters while strength stays the lead term.

Show the inputs **and the source-refs** next to the cost so the reader can both
recompute it and check each counted site by hand. The cost is a `derived:` claim
that cites the refs it came from. An `assumption:` row has no realized degree —
it stays in the table with cost `—`, and is left out of the profile counts and
the ranking until the user confirms it at the gate.

## Boundary Coupling Profile

Bucket the rows by strength band and report count, total cost, and
share of the total:

| band | kinds | count | total cost | share |
|---|---|---|---|---|
| static-weak | Name, Type (1–2) | … | … | … |
| static-semantic | Meaning, Position, Algorithm (3–5) | … | … | … |
| dynamic | Execution, Timing, Value, Identity (6–9) | … | … | … |

Position sits in the static-semantic band by its rank; it is an agreement about
order rather than about meaning.

The profile shows where the cost sits — how much is cheap static naming
vs. semantic conventions vs. dynamic coupling. It is a roll-up of the table; every
count and total cites the rows behind it.

## Ranked offenders

List the rows by cost, highest first — the top offenders carry the
boundary's coupling. Give each by id with its **cost** and **source-ref(s)** so
the reader can check it. This is the same table, ordered.

## Ordering

Sort by cost, highest first. Break a tie by the rows' source-refs compared one
by one in path-then-line order — first ref, then second ref, and so on; when
one row's refs run out first it sorts first. Break a tie that survives all refs
by row id, ascending.

## Unitization ledger

The rules above leave a short list of judgments to the analyst. Each one is a
ledger line — `rule | decision | rows affected | source-ref` — so a reader who
disagrees can recompute exactly the rows it touches:

- S5: which method was credited as a localizing helper;
- S4: which declaration was excluded as unreferenced;
- R7 (taxonomy): which stores were paired into one Value row;
- R6 (taxonomy): which methods were read as units of work, by which marker;
- R3 (taxonomy): which produce/rely citation pair backs each Meaning row;
- Step 0: which ambiguous member was placed on which side, what the shared
  kernel holds, and which patterns were fixed as site rules (test path, import
  line, unit-of-work marker, write marker, handle types).

A ledger with no lines is a claim that the rules decided everything — state it.

## What a reader may compare across runs

- The **set** of top offenders and the **band shares** are the stable reading
  of the seam: they survive a site miscount, because `log2` compresses degree.
- The **total** is an ordering inside one report. Compare two seams, or two runs,
  band by band and offender by offender — never by one total.
- Two runs that differ in a row's existence differ in a ledger line or a rule
  breach; find that line first.

## Honesty rules

- Report the table, the profile, the ranked offenders, and the ledger together;
  show every cost's inputs so the reader can recompute it.
- Every claim — row, kind, degree, cost, a profile count or total — carries ≥1
  `source-ref` per `connascence-evidence`. An unsourced claim is an explicit
  `assumption:`, surfaced for confirmation.
- Keep every row in the table; if you cap the offenders list for length, say
  how many were omitted and why.
