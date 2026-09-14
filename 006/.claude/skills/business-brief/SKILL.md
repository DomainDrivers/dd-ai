---
name: business-brief
description: How to distill any project material — a context map, an architecture
  diagram, code, docs, interviews, tickets — into a solution-free business brief; a
  narrative of what the business does, for whom, and where its money and risk move,
  carrying zero module structure. The brief is the only material a firewalled
  requirement generator ever sees, so the strip list, the mirror test, and the
  invariance test here decide whether the extensibility stress-test is honest. Load
  before framing the test.
---

# The business brief

The brief is the firewall's load-bearing wall. Downstream, requirements are invented
by a generator that reads *only* this file — so whatever solution structure leaks into
the brief leaks straight into the requirement set, and the test measures the model
against asks shaped by itself. Distill with that in mind: the brief describes a
**business**, and the same brief must come out no matter how that business happens to
be built.

Two leaks are worth telling apart. The **topology** is held out by the shape of the run
itself — the generator never receives the map. The **vocabulary** has no such guard: it
travels in whatever words this brief chooses, so how far the prose moves from the
source's own operations and labels is settled here and nowhere else.

## What goes in

Read every source the project provides — a context map, an architecture diagram, the
code, requirements, interviews, tickets, READMEs, stakeholder notes; do not assume any
particular document exists. From them, write:

- **What the business does and offers** — its activities, products, and services, as
  a narrative organized by business activity or value stream, in the business's own
  words.
- **Who is involved** — the actors and roles (customers, operators, partners,
  regulators) and what each one wants.
- **The money and risk map** — where the business earns, where it spends and saves,
  where it carries risk, where it sees opportunity.
- **The rules of the game** — business policies, constraints, and obligations
  (deadlines, penalties, regulatory duties, capacity limits), stated as facts of the
  business, not of any system.
- **What the business can already do** — its current abilities, so future asks can be
  told apart from things it does today.
- **Vocabulary** — the domain terms a stakeholder would use, each with its business
  meaning, woven into the prose or listed briefly.
- **Scale facts** where known — volumes, frequencies, seasonality — as business
  observations.

Every brief fact rests on a source. Where the material is thin, the gap is raised
with the user at the gate; a brief is never padded with invented business facts.

## Reading solution artifacts for business content

A context map or architecture diagram is a legitimate *source about the business* —
its boxes are usually named after real domain activities and its responsibilities
describe real business behavior. Harvest that content and discard the form:

- A box named after a domain activity contributes the **activity**, in prose: a
  "Pricing" box becomes "the business prices each loan to a target margin" — never a
  named unit, never an entry in a list of parts.
- A responsibility line contributes **what the business does**; who-owns-what does
  not survive the trip.
- An edge or arrow contributes, at most, a **business fact** ("an offer is only made
  once the applicant's identity is confirmed") — the mechanism that implements it
  (call, event, query) never does.
- A per-module description or public-operation list (a facade, an API document)
  contributes **what the business can already do**. Read it for the abilities behind
  the operations: a handful of operations usually add up to one thing the business can
  do, and that activity — the one a stakeholder would name — is the level the brief
  reads best at. Woven into the narrative of the work it serves, never listed module by
  module, and never under its technical name.

## The strip list

None of the following appears in the brief, as words or as ideas:

- unit names used as structure — modules, contexts, services, components, systems,
  layers — or any enumeration of a solution's parts,
- boundaries and ownership ("X owns Y", "the source of truth for Z lives in…"),
- integration mechanisms — synchronous/asynchronous, events, queries, messages,
  APIs, shared kernels, published language, anticorruption layers,
- pattern and paradigm labels — CRUD, saga, aggregate, locking, orchestration,
- strategic classifications — core, supporting, generic; class-of-problem labels,
- diagram topology — anything whose only meaning is "how the boxes connect",
- source attribution and self-assessment — the names or formats of the files the brief
  was distilled from, and any line describing what the brief itself does or does not
  carry. Both tell the reader that a solution artifact exists and was filtered.
  Provenance belongs in what you show the user at the gate.

## Two tests before the brief leaves the step

- **The mirror test.** If the brief's sections or bullets correspond roughly
  one-to-one to the source's boxes, the map's partitioning has survived as skeleton —
  the generator will reinvent the module layout from the brief's table of contents.
  Run it as four steps, not as a judgement: (1) list the source's boxes by name; (2)
  for each brief section, note which of those boxes it draws from; (3) rewrite every
  section that draws from exactly one box; (4) rewrite the outline when the brief
  carries at least as many top-level sections as the source has boxes, in the same
  order. Rewrite as activity narrative until the correspondence dissolves.
- **The invariance test.** Would this brief read word-for-word the same if the
  business ran on a single monolith, on twenty services, or on paper and telephones?
  Every sentence that fails is describing the solution; strip or rephrase it.

Two further signals, each checkable by eye:

- a section heading repeats a label from the source's boxes word for word,
- a sentence turns on "sends", "receives", "syncs", "owns the data for", or another
  verb that only makes sense between the parts of a system.

A useful final check: *could someone who has never seen the architecture have written
this?* That is exactly the reader the brief is for — put the same question to the user
at the gate.
