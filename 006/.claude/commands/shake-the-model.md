---
description: Stress-test a model's extensibility along one named business metric. The run distills whatever material the project provides into a solution-free business brief and, where competitor material or a permitted web pass allows, a short grounded digest of what rivals already offer. Brief, digest, and metric go to a firewalled generator agent that invents plausible future requirements — the architecture never reaches it, so the set reproduces the same neighborhood of asks whatever the module layout, and replays against a rival decomposition on equal terms. Only then does the architecture enter, to measure how many modules each requirement forces to change (localized / distributed / new-module) with per-module reasons and source-refs. Neutral per-metric summary, no verdicts. Three steps; the first two end at a manual gate. Project-agnostic and stack-agnostic.
argument-hint: "[domain / architecture material (context map file, per-module descriptions with public APIs, diagram, codebase path, docs) and a business metric to improve; optionally named competitors or competitor material, and a requirement count, e.g. path/to/map.md \"time-to-offer\" 8]"
---

# /shake-the-model

Input: $ARGUMENTS

A model earns its keep when the changes the business actually asks for land in few
places. This command tests that along one axis: a **business metric**. It invents the
requirements the business might raise to improve that metric, then measures, for each,
how many modules of the current model must change and how.

The asks come from two wells. Most are drawn from the business itself — what it does,
earns, and risks. A few can be drawn from the **competition**: where the run has
competitor material, rivals the user names, or the user's go-ahead for a web pass, it
scans what rival businesses already offer their customers and lets that surface asks
this business has not raised yet but plausibly would once a competitor sets the bar.
Both wells are business-level; neither owes anything to the architecture.

The test is only honest if the requirements owe nothing to the model. A requirement
invented while looking at the module layout fits that layout by construction — every
change lands locally for free and the measurement confirms nothing. So this command
**builds the run around that blindness** instead of asking anyone to hold it:

1. The main session distills a **business brief** from everything the project provides,
   and grounds a short **competitor digest** beside it — both business-level, both
   stripped of solution structure.
2. A separate **`blind-requirements-generator` agent** receives those two files and the
   metric, and nothing else. It runs in a fresh context, so nothing it invents can be
   shaped by the architecture.
3. Only after the requirement set is fixed does the architecture enter — as the module
   inventory the blast radius is counted against.

A context map, an architecture diagram, or **per-module descriptions with their public
APIs** are welcome, and they live a double life: at Step 1 they are evidence about the
business, stripped by the brief rules; at Step 3 they are the inventory the impact is
measured against — module names at worst, the operations each module offers today at
best. Provide the finer material whenever you can. None of it ever reaches the
generator.

Because the asks derive only from the business, the competition, and the metric — none
of them the architecture — the requirement set is **solution-invariant**: rerun against
a different candidate decomposition of the same business, Steps 1–2 reproduce the same
neighborhood of asks — so the same set measures two rival models on equal terms.

This command **carries no project facts**. It discovers the business and the model at
runtime and confirms each step at a gate. Accept any input shape: a context map, a
diagram, a codebase, a pile of notes — or nothing but a pointer to the project.

Load the `business-brief` skill (what goes into the brief, the strip list, the mirror
and invariance tests), the `competitor-scan` skill (how to find, ground, and strip
competitor capabilities into the digest), and the `change-impact` skill (how to estimate
and classify each requirement's blast radius and summarize the result) before Step 1.

**Gates are hard stops.** Steps 1 and 2 each end at a gate: present the step's output,
put the gate's question to the user, and **end the turn** — the next step begins only
from the user's reply. This holds for every run, including one whose invocation
describes the whole journey ("generate seven requirements and then evaluate them"):
that phrasing sets the run's scope, and each gate still stops for the user's answer.
Confirmation comes from the user's message, never from the shape of the original
request. Step 3 carries no gate: once the requirement set is confirmed at the Step 2
gate, it measures the change impact, displays the assessment, and writes the report to
`shake-the-model-report.md` in the same turn.

**The metric is an input, never an invention.** If the run names a metric, take it
verbatim as the axis. A document the run points at names one just as a sentence does:
read the metric from that file verbatim. If run or file names several, run the
generation once per metric and keep the results grouped per metric. If nothing names
one, **ask the user** — what number, moved which way, would make the business say
"better"? Picking a metric out of prose nobody pointed at, or inventing one yourself,
puts the axis of the whole run in your hands rather than the business's.

A metric phrased in solution terms — "fewer services touched per change" — carries the
module layout into the brief and into the asks, so hold the metric to the brief's
invariance test: if it only makes sense against one architecture, ask the user to
restate it as a business number, and take that restatement verbatim as the axis.

**Evidence rule.** Every change-impact claim — "this requirement forces module X to
change" — carries a **source-ref** (`code:`, `map:`, `doc:`, or an explicit
`assumption:`) naming *which* responsibility or rule in that module changes, so a
reader can audit each impact rather than trust a number. This is a projection, not a
measurement; where a source cannot pin the blast radius, give a range and say so.

**Scope.** This measures conceptual / business models, where extensibility is the
value. A component whose whole purpose is a technical guarantee (throughput, latency,
a protocol limit) is read on that guarantee, not by this blast-radius measurement —
note such components and exclude them.

---

## Step 1 — Fix the metric, distill the business brief, and scan the competition

1. **Fix the metric.** State it verbatim, with its direction (improve = up or down)
   and, if ambiguous, its unit and horizon — confirm these with the user rather than
   guess. Several metrics → several axes, handled per metric from here on.
2. **Distill the business brief** from every source the project provides, per the
   `business-brief` skill: a narrative of what the business does, for whom, where it
   earns, saves, and risks, its rules of the game, and its vocabulary — organized by
   business activity, carrying no module names as structure, no boundaries, no
   integration mechanisms, no diagram topology, no strategic labels. Run the skill's
   **mirror test** (brief sections must not correspond one-to-one to the source's
   boxes) and **invariance test** (the brief must read the same whether the business
   ran on one monolith or twenty services) before showing it. Write it to
   `shake-the-model-brief.md`.
3. **Scan the competition** per the `competitor-scan` skill, from the material at hand:
   competitor material the run provides, or rivals the user names. Distill a short digest
   of customer-facing capabilities they offer that this business appears to lack, each
   entry grounded with a source-ref (`doc:`, `user:`, or `web:` when a web pass ran) and
   filtered to the ones that plausibly bear on the metric under test — with several
   metrics in play, write one digest for the run and tag each entry with the metrics it
   bears on. Strip each entry of the competitor's own solution and run the same
   invariance test the brief passes. Never
   invent a competitor or a capability. **Reach the web only after the user says yes**:
   the search terms come from the business's own facts, so name the rivals you would
   search for and get that answer at the gate first. Write the digest to
   `shake-the-model-competitors.md` only when it holds at least one grounded entry;
   otherwise say the digest is empty and the run proceeds on the brief alone.
4. Note any purely technical components to exclude from the summary (per Scope).

**Gate.** Present the metric (verbatim, with direction), the brief, and the competitor
digest, and say which sources each fact rests on — for the digest, which rival and which
`web:`/`doc:`/`user:` ref backs each capability. Ask the user to audit one thing above
all: *could someone who has never seen the architecture have written this brief and this
digest?* Invite the user to name rivals or add competitor material you could not reach,
and to strike any competitor capability they judge irrelevant to this business or this
metric. If no competitor material came in and no rival was named, offer the scan here:
name the rivals a web pass would look for, and ask whether to run it — a "no" keeps the
run brief-only. If material was too thin to ground the brief, ask for the missing
business facts here — never pad the brief or the digest with invented ones. With several
metrics under test, settle the per-metric requirement count here too: 8 asks per metric
is the default, so four axes come to 32 — confirm that total, or agree a smaller count per
metric. Then end the turn and wait for the user's confirmation or corrections — everything
downstream stands on these files, and Step 2 begins only from their reply.

## Step 2 — Generate requirements (firewalled)

Spawn the **`blind-requirements-generator`** agent. Pass it exactly these things:

- `metric` — verbatim, with its direction and any unit/horizon fixed at Step 1,
- `brief-path` — the path to `shake-the-model-brief.md`,
- `competitors-path` — the path to `shake-the-model-competitors.md` when the digest is
  non-empty (omit it when the scan grounded nothing — the run is then brief-only),
- `count` — the requirement count **per metric** (default 8 per metric, whatever the
  number of metrics; honor an explicit count in the run). With several metrics under
  test, take the per-metric count the user settled at the Step 1 gate,
- `competitor-count` — the cap on competitor-derived asks (default: at most 3, and never
  more than a third of `count`, rounded down — when that yields 0, the set is brief-only;
  honor an explicit "max a few from competitors" instruction).

Pass nothing else — no map path, no codebase pointer, no summary of the architecture, no
hint of the model's parts. Where architecture material sits beside the brief, put the
brief and the digest in a folder of their own: the agent is told to open nothing else,
and a separate folder keeps a stray read away from the model. Both files it receives are
business-level, never the solution. The agent returns the requirement set: each ask a
pure business desire that could plausibly improve the metric, with its causal
**mechanism** on the metric and its **grounding** — in the brief, or, for the few
competitor-derived asks, in a digest entry (tagged `competitor:<rival>`) plus the brief
fact that makes it credible here. One agent invocation per metric when several are under
test — each gets the same digest and the one metric it is testing.

**Gate.** Present the requirement set with the metric stated at the top, marking each
ask's source — `brief` or `competitor:<rival>` — so the user can audit the
competitor-derived few against the rival and ref they came from. This is where the user
adds the asks they actually fear and drops any that are unrealistic — the set drives
every later number. A user-added ask must hold the same discipline: business phrasing, a
mechanism on the metric, no note of which part of the model it touches. Ask the user
whether each ask makes sense for this business and this metric, then end the turn and
wait for their verdict. After this gate the set is **frozen text** — Step 3 interprets
it, never rewrites it.

## Step 3 — Measure change impact and write the report

Now the architecture enters. Establish the **inventory** from whatever the project
provides, at the finest granularity the sources allow, per the `change-impact` skill:

- **module level** — the context map, an architecture diagram, or a survey of the
  codebase if no map exists;
- **capability level** — for each module, the operations it publicly offers today,
  taken from per-module descriptions, public API documents, or the module's facade
  read straight from the code. Work at this level whenever the sources support it:
  it turns the estimate from "which responsibilities sound involved" into a **build
  check** against concrete blocks.

For each requirement, run the **build check** and classify the result by measured
blast radius, both exactly as the `change-impact` skill defines them: try to assemble
the ask from the operations that exist, sort every module it involves into uses /
extends / gains / no home, count the modules that extend or gain, and give each a
one-line reason with a source-ref naming the operation added or altered (`code:` the
facade line, `map:`, `doc:`, or an explicit `assumption:` where capability information
is missing). Classify only after that inspection — the requirement arrives with no
expected reach, so its classification is whatever the modules reveal.

**Display the change-impact assessment.** Present the impact as **one table, one row
per requirement**: the ask (short form), its classification, the module count, the
modules touched, and a **why** column — a short justification of why the change lands
in exactly these modules and not in fewer or more: what pulls each module in, and what
would have to be true for the change to stay narrower or spread further. Where the
inventory carries capabilities, the why speaks in blocks: which existing operations the
ask would simply use, and which are missing or must change. The per-module detail — how
each touched module must change, with refs — goes into the report; put it on screen too
when the set is short or the user asks for it.

**Write the report — automatically, no gate.** Right after showing the assessment, build
the **Result summary** for the metric under test — a neutral projection of where
metric-improving change lands and how far it spreads — and write it to
`shake-the-model-report.md`. Nothing stands between the assessment and the file:
display, then dump. Render it in the order the `change-impact` skill's Result summary
sets out — metric, requirement set, per-requirement table, which modules and how,
aggregate, distribution. The requirement set goes in as frozen at the Step 2 gate, each
ask carrying its source tag (`brief` or `competitor:<rival>`) and pointers to
`shake-the-model-brief.md` and, where the run used one,
`shake-the-model-competitors.md`.

Close the report with a **replay note**: the requirement set derives only from the
brief, the competitor digest, and the metric — never the architecture — so to compare an
alternative decomposition of the same business, keep the set (and the digest it drew on)
frozen and rerun this step against the other inventory — same asks, different model,
numbers directly comparable.

Report only facts — where each change lands and how far it spreads. If a wide-spread
requirement is worth flagging, flag it by facts: module count and which boundary it
crosses. No line of the output states whether the result is good or bad. After writing
the file, tell the user where the report landed; the run ends there.
