---
description: "Restyle a module from its current implementation idiom to a fixed target idiom while keeping the same behavior tests green. The source and target idioms are defined by the bundled idiom-catalog skill. Runs as numbered gates: at each it discovers what the code does, proposes a choice, and stops for you to decide; each code-changing step commits on its own, so any step can be undone from its own commit."
argument-hint: "--target=<module-path>"
---

# /restyle: Idiom Swap Under a Green Safety Net

Input: $ARGUMENTS

Moves a unit of behavior from its **current implementation idiom** to a **target
idiom** — **without changing what it does**. The proof is that the same behavior
tests stay green. The mechanic is the four-layer test model: keep the Test Cases and
DSL fixed, swap the Protocol Driver and the implementation underneath.

The source idiom to recognize, the target idiom to build, and the recipe between
them are **defined by the bundled `idiom-catalog` and `idiom-transformation-recipes`
skills**. If the target module does not match the source idiom the catalog describes,
the tool says so and stops.

The HOW lives in four skills — load them as the steps say: **four-layer-test-architecture**
(the model + the raise/lower mechanic), **idiom-catalog** (the two idioms in play —
the source to recognize and the target to build — with the invariant the swap must
preserve), **idiom-transformation-recipes** (the step-by-step recipe for this
transition), **behavior-preservation-contract** (what stays invariant + the
green-tests rule).

The procedure runs through three agents, dispatched by the steps below:
`idiom-analyzer` (confirm the source idiom, extract the contract),
`test-harness-architect` (build the four-layer harness, raise the scenarios),
`idiom-transformer` (swap to the target idiom, lower-bind, prove green). The agents
do the work; **this command holds the gates and owns the commits**.

## Arguments

- `--target=<module-path>` (required) — the unit to restyle (a file, package, or
  glob resolving to one unit of behavior).

The target idiom comes from the bundled `idiom-catalog` skill. The one place a
genuine variant can appear — a target-idiom form that would change the published
contract — is confirmed (not defaulted) at gate 0c.

## Gate discipline (applies to every step)

Each step ends at a **gate**. At a gate, run the loop **discover → propose → stop**.

- **Discover** — read the target and its tests; do not assume. Surface what is
  actually there: that the unit really matches the source idiom the catalog
  describes, the test binding level, the contract to preserve, the test conventions
  the codebase already uses.
- **Propose** — state the decision, list the realistic options with a one-line
  trade-off each, and recommend the one *consistent with what you discovered* —
  never a hardcoded default, never a preferred style imposed on the code.
- **Stop** — present the finding and the proposal, and wait for the user's decision
  before acting. Carry the confirmed choice forward; do not re-ask it later.

Never silently pick a driver binding, whether to mock, a target-idiom variant, or a
contract change. The user instructs the choice at every gate.

**Record the gate.** Once the user confirms, append the step, its status, and the
choice they instructed to `state.json`, and set `current_step` to the step that
follows. That record is what lets an interrupted run resume without re-asking a
decided question.

## Commit cadence

Every step that changes code commits on its own, once its gate is confirmed —
**1b**, **2**, **3** (the raised harness, the idiom swap, the lower-bind +
retirement). After the user approves the diff at a gate, make one focused commit
for that step before starting the next, with a message naming what it did (the
four-layer harness; the idiom swap; the lower-bind and source-idiom retirement),
written as `restyle(<target-id>) <step>: <what changed>`. Stage the step's
production and test code only. The safety, analysis, and design steps — **0a, 0b,
0c, 1a** — change no code and are **not** committed. If the swap (step 2)
momentarily turns a frozen scenario red before the wiring is fixed, that red state
is a real, taggable point: commit it on its own if you want the catch on record,
then commit the green fix. That red lives inside
step 2: the step reaches its gate green, as `four-layer-test-architecture` requires.
Never squash steps together, and never `git push` unless asked.

## Step 0a — Safety gate (green baseline)

Resolve `--target` to **one** unit of behavior; if it expands to more than one, ask
the user to narrow it. Compute a `target-id` slug.

**Resume check.** Read `.restyle/<target-id>/state.json`. If it exists and its
`current_step` is not `done`, report where the previous run stopped — the step, the
gate log, and the commits it already made — and offer two moves: resume at
`current_step`, or start over. A resume carries the recorded gate choices forward;
do not re-ask a question the log shows as confirmed. Starting over backs the old
state file up first. With no state file, this is a fresh run.

Also confirm `.restyle/` is excluded by the target repo's tracked `.gitignore`, and
add the line there if it is missing.

Then **discover** the unit's behavior tests and run them.

- Green → record the baseline.
- Red → **halt**: "Baseline is red; fix or quarantine the failing tests before a
  restyle." Offer to list the failures.
- No tests → **end the run**: report that the unit has no behavior tests, so there
  is no safety net to swap under. Recommend the next move plainly — write behavior
  tests that pin the unit's current observable behavior, then run `/restyle` again —
  and stop there.

**Gate.** Show the baseline (which tests, how many, the result) and confirm the
target. No code changes here; nothing to commit.

## Step 0b — Confirm the source idiom, extract the contract (`idiom-analyzer`)

Load **idiom-catalog**. Dispatch `idiom-analyzer` (read-only) to confirm the unit
matches the **source-idiom** signals the catalog describes, map the **test binding
level** (domain or application), and extract the **contract to preserve** — the
observable outcomes, the consistency boundary, the public surface. It also surfaces
`coverage_gaps`: behaviors the baseline does not exercise.

If the unit does **not** match the source idiom, `idiom-analyzer` says so: this tool
does not apply. Report that and stop — do not force the move.

**Gate.** Read back the idiom card. Surface the coverage gaps as a caveat (lean
mode names them, does not block on them — they are simply not proven by the
restyle). Confirm the classification. Read-only step; nothing to commit.

## Step 0c — Confirm the target idiom

The target idiom is fixed by `idiom-catalog`. This gate is a confirmation, not a
menu. **Discover** from the 0b contract whether the move is clean, then **propose**:

- State the **invariant to preserve** (same observable outcomes for the same command
  and starting state; consistency boundary unchanged).
- Flag any **target-idiom variant** that would change the public contract. The
  default is the variant that keeps the published outcomes **unchanged** — the same
  event types with the same domain payload. A variant that would enrich, alter, or
  drop them is a **public-contract change** — surface it here; never slip into it
  silently. (`idiom-catalog` and
  `idiom-transformation-recipes` name the specific fork, if any, for this tool's
  target.)

Validation: if 0b reported the unit does not match the source idiom, stop — the tool
does not apply. If the recipe is likely to force a contract or boundary change (e.g.
choosing a contract-changing variant), it must be **explicitly approved and
recorded** at this gate.

**Gate.** Stop for the user to confirm the target idiom (and any variant choice) and
— if applicable — approval of any forced contract or boundary change. No code yet.

## Step 1a — Design the harness, then choose the testing strategy (`test-harness-architect`)

Load **four-layer-test-architecture**. Dispatch `test-harness-architect` in
**design mode**: it proposes the four-layer harness *without writing it yet* — the
**DSL** (domain verbs + fact-shaped observations, no implementation types), the
**application-level Protocol Driver** (the preservation proof surface), the fast
`domain` driver step 3 repoints onto the new core, and how the current
implementation-shaped assertions will be **re-expressed as domain facts**.

The re-expression comes back as an **assertion ledger**: original assertion → DSL
outcome → what the outcome no longer distinguishes → why that loss is acceptable.
The tool designs the DSL itself, so without the ledger the proof is measured against
a surface the tool chose; read it here and push back on any row whose loss you do
not accept. It is frozen with the DSL in 1b and lives in `contract.md`.

**Raise all — there is one binding for the proof.** The design binds **every**
behavior scenario through the application driver. That binding *is* the preservation
proof, so a scenario left below it proves nothing about the swap. If the architect
predicts one cannot pass at the application level — a facade short-circuit that
shadows a guard, an outcome with no observation surface — it reports the scenario
with its reason, and the answer is to **fix the harness**: add the query or read
projection the outcome needs, or bind to the surface that exposes it. Such
predictions are often wrong, so an empirical run through the real facade settles
them faster than a guess. The only test that stays outside the frozen set is a
focused unit test of an internal: it is not a behavior scenario, it keeps running as
the unit test it is, and the gate names it as outside the proof.

Two sub-decisions are genuine gates here, and the right answer is whatever the
codebase already supports — discover, do not assume:

- **Which application surface** does the proof driver bind to? Reuse the **real,
  existing** facade for this unit; do not invent a parallel one when a real one
  exists. Where the unit has no facade, the driver binds to the unit's own public
  API, and the gate records that this proof is weaker: the tool writes no
  production code to manufacture a contract surface.
- **Real collaborators or doubles?** Prefer the **real** collaborators and a
  **real** store (e.g. the project's integration-test infrastructure) over mocking
  at the proof surface — a mock there hides the very contract the restyle must
  preserve. Fall back to doubles only when no mock-free path exists, and say so.

**Gate.** Present the harness design, the assertion ledger, and these two choices
with trade-offs and a discovery-grounded recommendation. Stop for the user to
instruct each. The architect writes the proposal to `.restyle/<target-id>/contract.md`
(`status: proposed`) and writes nothing else — no test code, no production code,
nothing to commit.

## Step 1b — Build the harness, raise the scenarios, prove the raise (`test-harness-architect`)

With the design confirmed, `test-harness-architect` writes the harness: the DSL, the
**frozen** scenarios expressed purely in it, the application-level driver, and the
fast `domain` driver. It re-expresses the implementation-shaped assertions
as the domain facts chosen in 1a. From here the Test Cases and DSL are **frozen** —
never edited (a change to them would mean behavior changed).

**Every** frozen scenario is bound through the application driver. If one cannot be
made green there, the harness is what is wrong, not the scenario: fix the driver or
the observation surface, and report at the gate what you added.

**Prove the raise.** Because assertions were re-expressed, run the frozen scenarios
through **both** drivers (application + domain). Their agreement is the evidence the
re-expression did not change meaning. Both must be green.

The architect then updates `.restyle/<target-id>/contract.md` in place to
`status: frozen`, recording what was built and every deviation from the 1a
proposal. Steps 2 and 3 read that file.

**Gate.** Show the harness green through both drivers. Stop for confirmation, then
**commit 1b** (the four-layer harness) before step 2. Test code only — production is
still byte-identical.

## Step 2 — Swap to the target idiom behind the frozen contract (`idiom-transformer`)

Load **idiom-transformation-recipes** and **behavior-preservation-contract**.
Dispatch `idiom-transformer` to implement the target idiom behind the frozen
contract, following the recipe: introduce the target-idiom code (the decision and
guard logic moved **verbatim**, not reinterpreted) and repoint the application
facade onto it (facade signatures stay identical). The source idiom's now-replaced
methods stay in place until step 3 retires them. The published outcomes and the
consistency boundary stay unchanged.

Watch for the safety net catching a hidden behavior gap: a path that silently relied
on the old idiom's mechanics can turn a frozen scenario **red** during the swap. That
red is the net doing its job — fix it as **wiring**, never by touching a Test Case,
and it goes green again. (To put the catch on record, commit the red point on its own
before the green fix; see Commit cadence.) Where the fix adds persistence the old
path never performed, it repaired a **latent defect** and the observable effect
changed with it — say so at the gate.

**Gate.** Show the diff and the frozen scenarios green through the application
driver, and name any latent defect the swap repaired, so the new effect is on
record. If the recipe hit a forced contract/boundary change not approved at 0c
(e.g. a variant the recipe needs that 0c did not approve), return "needs approval"
and stop for the user. On confirmation, **commit 2** (the idiom swap) before step 3.

## Step 3 — Lower-bind, prove green, retire the source idiom, and verify (`idiom-transformer`)

`idiom-transformer` repoints the fast `domain` driver from the source idiom onto the
new target-idiom core (the "lower" move), giving the new idiom its own fast feedback
surface. Run the frozen scenarios through **both** drivers — all must be green: the
application driver proves preservation, the domain driver proves the new core works.
Then **retire the source idiom** — delete the now-dead methods and any old test fused
to it — and clean up scaffolding (e.g. a temporary test-only config bean replaced by
a proper one).

**Verify the whole transition.** Run the full suite once more and confirm it holds:
the invariants (outcomes, public contract, consistency boundary) intact, the frozen
scenarios green through the application driver, the full suite passing. If green
could not be reached through the application driver without editing a Test Case,
this is not a restyle: roll back and report what blocked the swap.

**Gate.** Show both drivers green, the source idiom's methods gone, and the full
suite passing. Stop for confirmation, then **commit 3** (the lower-bind, source-idiom
retirement, and cleanup). State the **coverage-honesty note** plainly — preservation
is proven only on the paths the frozen scenarios cover; uncovered paths are not
proven by this restyle. The per-step commits (1b, 2, 3) already cover every code
change; do not add a squashed wrap-up commit, and do not `git push`, unless asked.

## State and artifacts

Under `.restyle/<target-id>/`:
- `idiom-card.json` (0b) — the unit's current idiom, where its tests bind, and the
  contract to preserve. Read at 1a and 2.
- `contract.md` — written at 1a as `status: proposed`, updated in place at 1b to
  `status: frozen`. Carries the DSL verbs and outcomes, the assertion ledger, the
  facade contract, and which driver level is the proof surface. Read at 2 and 3.
- `state.json` — `current_step`, the confirmed gate choices, the baseline result,
  the per-step commit hashes, and the gate log. Read at 0a to resume an interrupted
  run.

These three files are the run's memory between steps. Each agent works in its own
context, so what one discovers reaches the next only through this folder. Keep the
folder in step with the list above: findings written under some other filename sit
where the next step does not look for them.

`.restyle/` is the only place this workflow writes, besides the target's own
production and test code; it never edits inputs outside the target. It is a run
journal, not a deliverable: it stays out of the step commits, so every gate diff is
production and test code alone. The 0a `.gitignore` line is what keeps it there, in
the repo's tracked ignore file, so the exclusion travels with the repo.

## Guiding rule (non-negotiable)

**There must be tests, and every step ends green.** No tests → no restyle. Red at
any gate — other than the transient, immediately-fixed red the swap may surface — →
stop or roll back. The same Test Cases and DSL are never edited; only the Protocol
Driver and the implementation change. Green tests prove behavior was preserved **on
the paths they cover** — the final summary says so plainly.

## Error handling

- Agent error → surface; options: retry / skip step / abort.
- Target does not match the source idiom → stop at 0b; the tool does not apply.
- Forced contract or boundary change without approval → halt at 0c/2 for the user.
- Cannot reach green through the application driver without editing a Test Case →
  roll back (step 3) and report what blocked the swap.
- Malformed state → back up, restart the current step.
