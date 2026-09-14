---
name: change-impact
description: How to estimate a requirement's blast radius across modules — at
  capability granularity where module descriptions, public APIs, or facades exist, via
  a build check (uses / extends / gains / no home) against the operations each module
  offers today — classify it (localized / distributed / new-module), list how each
  affected module must change, and roll the results into a neutral per-metric summary,
  including how to replay a frozen requirement set against an alternative
  decomposition. Load when measuring a model's extensibility stress-test.
---

# Change impact

For each requirement, estimate how far the change spreads and what that spread means.
The aim is a neutral projection of where each change lands: which modules must change,
and how — reported per metric, without a good/bad verdict.

## The requirement set is frozen text

The requirements arrive from a generator that never saw the architecture, and were
confirmed at a gate. Treat each ask as fixed: interpret it against the model, never
rewrite it to fit the model. Where an ask is ambiguous once held against real modules,
record the interpretation you chose alongside the impact — a reader must be able to
see which reading produced the number.

## Establishing the module inventory

A context map — or any architecture diagram that shows the system's modules and how
they depend on each other — is the key input to this step. It is the inventory of
modules you count the blast radius against, and it is what lets you say *how many*
modules a change touches at all. Establish that inventory before estimating — from a
context map, an architecture diagram, or a survey of the codebase — and count against
it. The more the map shows about each module's responsibility and its dependencies,
the sharper the count; a thin or absent map leaves the count resting on `assumption:`
ranges until it is filled in, so treat obtaining or reconstructing the inventory as
the first move of the measurement.

Whenever the sources allow, sharpen the inventory one level further, to
**capabilities**: for each module, the operations it publicly offers today — taken
from a per-module description, a public API document, or the module's facade / public
interface read straight from the code. A capability entry is an operation in business
terms with its essential inputs and outcome. This granularity changes what the
estimate can do: with module names and responsibilities you judge *which modules
sound involved*; with capabilities you check *whether the ask can be built from the
blocks that exist* — and a module that merely gets invoked stops masquerading as a
module that must change. Record where each module's capability list comes from; a
module with no capability information is estimated at responsibility level, with
`assumption:` refs and ranges.

## The build check

With a capability-level inventory, estimate each requirement by trying to **assemble
the ask from the operations that exist**. Sort every module the ask involves:

- **uses** — an existing operation covers its part as-is. The module is invoked, not
  changed: it does not count toward the blast radius, but it belongs in the why — it
  is the evidence that keeps the count down.
- **extends** — an existing operation must change: a new rule inside it, a different
  input or output, a changed behavior. The module changes.
- **gains** — the ask needs an operation the module does not offer today, and this
  module is its natural host. The module changes.
- **no home** — no existing module can reasonably host the missing capability; the
  requirement points at a brand-new module.

An ask assembled entirely from existing operations changes nothing — a real and
reportable outcome (classified localized, flagged "composition-only") that only the
capability inventory can reveal. The reverse is equally real: a module whose
responsibility reads as "already covers this" may turn out to offer no operation
that actually does. The build check converts both kinds of guess into a checkable
claim.

## Estimating blast radius

List every domain module a requirement forces to change — where a build check ran,
these are its *extends* and *gains* modules. Do not pre-filter by any importance
label a document attaches to a module — only purely-technical components (per the
tool's scope) are set aside. For each module, give a one-line reason naming the
operation, responsibility, or rule that changes, and a source-ref (`code:`, `map:`, `doc:`, or
an explicit `assumption:`). This is a projection, not a measurement — where the
evidence cannot pin the radius, give a range and say what would settle it. Every
impact must be auditable; a count with no per-module reason is not a finding.

## Classifying each requirement

Classify by the *measured* blast radius. This is a neutral count of reach, not a
verdict.

- **localized** — zero or one existing module changes. (A config-only outcome, or an
  ask assembled entirely from existing operations — flagged "composition-only" — is
  localized.)
- **distributed** — two or more existing modules change. List each and why; do not
  label the spread good or bad.
- **new-module** — the requirement forces a brand-new module to appear. New-module wins
  over the other two labels: an ask that needs a new module *and* changes existing ones
  is new-module. Report it as `modules: N (+1 new)`, where N counts only the existing
  modules that extend or gain, and list them beside the new one.

Where the evidence gives the module count as a range, classify on the range's upper
bound, flag the row `range: a-b`, and name the one fact that would settle it.

Assign the classification only after inspecting the modules. The requirement arrives
with no expected reach attached, so its classification is whatever that inspection
reveals — including an ask that turns out to have no home and forces a brand-new
module.

Every classification carries a **why**: one or two sentences justifying why the change
lands in exactly these modules and not in fewer or more — what pulls each module in,
and what would have to be true for the change to stay narrower or spread further.
Where the inventory carries capabilities, the why speaks in blocks: the operations
the ask would use as-is, and the operations that are missing or must change. A
bare label with a module count is not a finding.

## Result summary (neutral, per metric)

State plainly, for the metric under test:

1. **Metric evaluated** — verbatim, with its direction, on one explicit line.
2. **Requirement set** — each ask with its source tag, mechanism, and grounding, as
   frozen at the confirmation gate, with a pointer to the business brief and, where the
   run used one, the competitor digest.
3. **Per-requirement table** — one row per requirement: the ask (short form),
   classification, module count, the modules, and the **why** behind the
   classification.
4. **Which modules — and how each must change.** For every module touched, a listing
   of the concrete change: which operation is added or altered — or, at
   responsibility level, what rule/field/API/event changes — with a source-ref, plus
   the existing operations the ask uses as-is. Naming a module is not enough — say
   what goes into it and where.
5. **Aggregate over the metric** — for each module, how many requirements make it
   change (extends or gains), in a separate column how many merely use it, and what
   accumulates there across the whole set: which modules absorb the business's push on
   this metric, and which never feel it. Only the change column feeds the counts.
6. **Distribution** — counts of localized / distributed / new-module.

Report the numbers and the module-level changes; do not grade the model or call any
outcome good or bad. If a wide-spread requirement is worth flagging, flag it by facts
(module count, which boundary it crosses), not by verdict.