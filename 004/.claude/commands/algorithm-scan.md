---
description: Scan a domain for canonical algorithms hidden behind business vocabulary (bridge-word + structural recognition) and report candidates with mappings.
argument-hint: "[domain requirements, feature description, or path to a candidate-areas file]"
---

# /algorithm-scan

Recognise which canonical algorithms a domain reduces to — so a known,
already-solved problem can be reused instead of reinvented. Recognising the
algorithm is a fail-fast: if a domain problem is really a Topological Sort or a
Bipartite Matching, the modeler inherits known complexity, known pitfalls, and
existing libraries instead of reinventing — and if it is *not* a recognised
problem, that surfaces immediately as genuinely novel territory.

Run this procedure **inline in the current conversation**. It is read-only on its
inputs and writes only its own report.

The central question: **does the domain contain a canonical algorithm hidden
behind everyday business vocabulary?** Two equal mechanisms answer it — a lexical
bridge-word lookup and your own structural recognition (see below).

This scan produces a per-area report (or a single domain report) listing
canonical algorithm candidates with:
- the bridge-word(s) that triggered the match (verbatim, source-language preserved)
- the algorithm name and id (from the algorithm catalog)
- the fit question the candidate passed (the structural test)
- a full domain → algorithm element mapping — every element the catalog entry
  names (node, edge, weight, item, constraint, …), not a single line
- near-miss redirects (the variant the evidence brushed against, e.g.
  "precedence only → topological-sort")
- confidence (high / medium / low)
- a short rationale tied to the evidence

This scan does **not** decide which algorithm wins, pick a tactical pattern, or
draw boundaries. It surfaces grounded candidates so the modeler starts from an
evidenced set instead of a blank page. `top_pick` follows the same rule: it
names the strongest-evidenced candidate, and every candidate that ties with it
— where the evidence is thickest, never a choice.

## Two matching mechanisms

The scan surfaces candidates through **two independent mechanisms**, and tags
every candidate with which one(s) found it (`match_basis`). Run both.

1. **Bridge-word lookup (lexical).** Walk the catalog's bridge-word index.
   Whenever the requirements use a catalog trigger word — or a clear near-synonym,
   in any language — the entry it points at becomes a candidate. Fast,
   high-recall, and it gives a verbatim audit trail. `match_basis: "bridge-word"`.

2. **Structural recognition (your understanding).** Independently of any word
   list, read the domain and ask: "what is the *shape* of the work here?" — a
   precedence order, a pairing, a packing-under-budget, a reachability
   question. Step 2 makes this concrete: nine shape questions plus every fit
   question in the catalog. When you recognise a canonical problem by its
   structure, it becomes a candidate **even if no trigger word ever appears**.
   Evidence it with the quote that shows the structure. `match_basis:
   "structural"`.

Neither mechanism is subordinate, and the two are **additive — a union, not an
intersection**. A candidate is matched if **either** mechanism fires; requiring
both to agree would silently drop real fits, so it is forbidden:

- bridge-word fires, structure unseen by your reading → **still matched**,
  `match_basis: "bridge-word"`;
- structure you recognise, no trigger word present → **still matched**,
  `match_basis: "structural"`;
- both fire on the same algorithm → matched, `match_basis: "both"` — the
  strongest signal, and the only combination eligible for `high` confidence.

`honest_misses[]` does **not** absorb these. It keeps its own narrow meaning: a
word in the requirements that clearly signals a canonical problem, yet points at
**no catalog entry at all** — the "unknown algorithm" flag. A mechanism that
fires always yields a candidate, never an honest miss.

## Inputs

Read-only. Accept the domain in whichever of these forms is available:

- **Requirements text** — taken from `$ARGUMENTS` if given (treat a file path as a
  candidate-areas / sources file to read), else recovered from the conversation
  (confirm a 2–3 sentence summary), else, if neither is present, ask:
  > "Describe the domain — what entities exist, what operations change state,
  > what constraints hold, and what questions must the system answer?"

- **A candidate-areas file, if present** — a Markdown or JSON file listing
  business areas, each with an `id`, a `name`, a short `description`, and
  `evidence` quotes (with source + location). When such a file is given, scan
  **per area** and key the report by `area_id`. When only free-form requirements
  are given, treat the whole input as one domain and emit a single report.

- **Supporting source files, if present** — interview transcripts, glossaries,
  event-storming exports, feature descriptions. Treat them as the quote bank to
  scan and to cite from.

Store the requirements text verbatim and preserve every quote in its source
language — never translate.

## Outputs

Write to an `algorithms/` directory in the working directory (or to a
caller-named output directory, if one was given). Name the resolved path before
the first write:

- `algorithms/scan.md` — one report, shaped as step 5 sets out. Write it
  incrementally — one area per write/append — so a long run cannot run out of
  output and truncate the file.

This scan only writes its own report. It never edits its inputs.

## Skill used

- `algorithm-catalog` — the curated catalog of canonical algorithms;
  authoritative for `algorithm_id` (its kebab-case `id:` slugs). Each entry
  carries the inputs that make a *deep* candidate: a **fit question** (the
  structural test), bridge words, a **domain → algorithm element mapping** shape
  (node / edge / weight / item / constraint …), **anti-signals with near-miss
  redirects** (`Not this if … → <other id>`), pitfall, and a
  **Skiena cross-reference** (`Skiena:`, the source of `skiena_pointer`). It
  also carries a bridge-word → algorithm
  shortlist. Use the fit question and the mapping
  shape for structural recognition; use the anti-signals to calibrate confidence
  and to emit near-miss redirects.

Defer to that skill file for canonical definitions.

**Evidence rule (applies throughout):** every candidate must cite ≥1 verbatim
quote with source + location — the trigger-word quote for a lexical hit, or the
structure-bearing quote for a structural match. No quote → no candidate. Quotes
are preserved in their source language and never translated.

## Procedure

### 1 — Load inputs

1. Read the requirements text (and any areas file / supporting sources).
2. Build an in-memory list of `(area_id, name, description, quotes[])` — one
   entry per area when an areas file is given, otherwise a single entry for the
   whole domain.

### 2 — Candidate extraction (both mechanisms)

For each area, step 1 collects the quotes that feed both passes; steps 2–3 are
the lexical pass and step 4 is the structural pass:

1. Collect the quotes that name this area's events / commands / pivotal events /
   actors / nouns / verbs. Reuse any pre-attributed evidence the areas file
   already carries.
2. Extract bridge-word candidates from the quotes. Bridge-word categories
   (semantic intent, matched in whatever language the source uses — no
   language-specific keyword list):
   - adjectives / participles: excluding, satisfying, matched, constrained,
     dependent, ordered, selected, shifted, nearest, next, following, related, …
   - nouns: queue, path, cycle, graph, tree, schedule, flow, matching,
     partition, cover, …
   - verbs: queue, shift, match, exclude, arrange, assign, select, propagate,
     estimate, …
3. Multi-language tolerant: sources are often mixed-language (e.g. an interview
   transcript plus event-storming extractions). Capture verbatim — do not
   translate.
4. **Structural pass (mechanism 2 — run it on the area, never on step 2's
   hits).** Read the area for its *shape*, not its words. Answer all nine shape
   questions, and answer them from the area alone:
   1. **Order?** — must things be flattened into one sequence that honours every
      "must come before"?
   2. **Pairing?** — must members of two groups be paired up, by cost or by
      preference?
   3. **Subset under a limit?** — must a best subset fit inside a budget, a
      capacity or a container?
   4. **Coverage?** — must the fewest items be chosen so that everything is
      covered or touched?
   5. **Route?** — must something traverse a network at least cost, connect all
      of it, or push volume through it?
   6. **Grouping?** — must items fall into connected, balanced or
      non-conflicting groups?
   7. **Overlap in time?** — do intervals collide, or must a non-conflicting set
      be chosen from them?
   8. **Similarity?** — must near-matches be found in text, in space or between
      vectors?
   9. **Live dispatch?** — must the next item be picked under urgency that
      changes?

   Then walk the catalog end to end and test **every** entry's fit question
   against the area. The nine questions are a first sweep over the families, not
   the edge of this pass: they never narrow the walk, and the walk covers what
   they do not ask (reachability, constraint satisfaction, range aggregates, …).

   Every yes — from a shape question or from a fit question — is a candidate
   carrying `match_basis: "structural"`, evidenced by the quote that shows the
   structure, even when step 2 found no trigger word for it.

### 3 — Catalog lookup

For every candidate from step 2 — **both** the lexical hits (steps 1–3) and the
structural matches (step 4):

1. Resolve the catalog entry. For a lexical hit, look the bridge-word up in the
   `algorithm-catalog` skill (its bridge-word → algorithm shortlist). For a
   structural match, name the catalog entry your reading points at directly — no
   index row is required. Use the entry's `id:` slug (kebab-case) as
   `algorithm_id`.
2. **Apply the entry's fit question and anti-signals.** Read the entry's **Fit
   question** against the area: a clear yes is a real candidate. A tripped
   **Not this if** anti-signal means the evidence points at the redirected
   problem as well — record the redirect as a `near_miss` on the original
   candidate, and raise the redirected entry as a candidate of its own. The fit
   question is also how a structural match is justified when no bridge-word
   fired.
3. For each candidate, record:
   - `algorithm_id`, `algorithm_name`, `match_basis`
     (`bridge-word | structural | both`), the evidence quote (the trigger-word
     quote for a lexical hit, or the structure-bearing quote for a structural
     match), and the `area_id`.
   - If the **same** algorithm was raised by both mechanisms in this area, merge
     the two into one candidate with `match_basis: "both"`.
   - `fit_question` — echo the entry's fit question, so a reader sees the test
     the candidate passed.
   - The candidate `domain_to_algorithm_mapping` — build it from the entry's
     **Domain → algorithm** line and map **every element it names** (node, edge,
     weight, item, constraint, …) onto concrete domain terms, not a single line.
     A one-element mapping is usually an under-read; reach for the full shape.
   - `near_misses[]` — the entry's **Not this if** redirects the evidence brushed
     against, each as `{to, because}`: `to` is the redirected entry's
     `algorithm_id`, `because` is the quote that made the evidence brush against
     it. A near-miss is a finding (the domain is a *variant* of a known
     problem), not a rejection.
   - Confidence (agreement raises it; it never gates candidacy):
     - **high**: both mechanisms fired on the same algorithm
       (`match_basis: "both"`), the structural evidence is a **different quote**
       than the bridge-word evidence, and the mapping names ≥3 of the entry's
       elements. Two readings of one quote are one mechanism, not two.
     - **medium**: a single mechanism fires with a plausible mapping that fits
       ≥1 quote.
     - **low**: a single mechanism fires but the mapping is speculative.

### 4 — Cross-area pattern detection

Only when an areas file split the input into multiple areas. Detect patterns
that span areas (the same algorithm often unifies seemingly separate areas):

- If the same algorithm scores ≥medium in ≥2 areas → flag it `consolidation`
  in the cross-area observations table (step 5) — a merge signal — or
  `shared_kernel`, when the areas keep their own boundaries and share one
  implementation of that algorithm.
- If two areas share a bridge-word cluster but resolve to different algorithms
  → flag it `seam` — a boundary diagnostic.

### 5 — Output assembly

Organise `scan.md` with one section per area (or a single section for an
undivided domain). Each section carries **two representations of the same
data**, in this order: the YAML block first (the machine-readable record —
structure English, quotes verbatim), then a Markdown rendering of that same
data immediately below it (tables — for a human reading the report; narrative
prose follows the language of its evidence). Never author the two separately;
the Markdown is a direct rendering of the YAML that precedes it, so they cannot
drift apart.

```yaml
area_id: "<id>"                 # omit when scanning a single undivided domain
name: "<verbatim>"
algorithm_candidates:
  - algorithm_id: "topological-sort"   # `id:` slug from algorithm-catalog
    algorithm_name: "Topological Sort"
    match_basis: "bridge-word | structural | both"
    fit_question: "<the entry's fit question, verbatim from the catalog>"
    bridge_words:
      - quote: "<verbatim>"
        source: "<file>"
        loc: "<line or section ref>"
        word: "<extracted word, verbatim — null for a structural match>"
        unverified: true            # only when the quote could not be searched
    domain_to_algorithm_mapping:    # every element the entry's Domain → algorithm names
      Stage: "Node"
      Dependency: "Directed edge (predecessor → successor)"
      "Execution order": "Topological order"
      "Impossible plan": "Cycle (contradictory constraints)"
    near_misses:                    # the entry's 'Not this if' redirects the evidence brushed against
      - to: "job-scheduling"        # `id:` slug of the redirected entry
        because: "<verbatim quote that brushed against it>"
    confidence: "high | medium | low"
    rationale: "<one paragraph, in the language of the evidence it cites>"
    skiena_pointer: "<the entry's Skiena cross-reference slug>"
  - ...
top_pick: "<strongest-evidenced algorithm_id — a list when several tie>"
honest_misses:
  - "<problem-signalling word noticed but no catalog hit — the 'unknown
    algorithm' flag for deeper modeling>"
```

**Rendered**

```markdown
### Algorithm candidates

| Algorithm | Match basis | Confidence | Fit question |
|---|---|---|---|
| Topological Sort (`topological-sort`) | both | high | Must items be flattened into one order that honours every "must-come-before"? |
| ... | ... | ... | ... |

#### Topological Sort (`topological-sort`)

**Domain → Algorithm Mapping**

| Domain Concept | Algorithm Element | Notes |
|---|---|---|
| Stage | Node | <verbatim domain term, if useful> |
| "depends on" | Directed edge (predecessor → successor) | |
| Execution order | Topological order | |
| Impossible plan | Cycle (contradictory constraints) | |

*(every element the catalog entry's Domain → algorithm line names gets a row
— node, edge, weight, item, constraint, … — a one-row mapping is usually an
under-read)*

**Evidence**
- "<verbatim quote>" — <source>, <loc> — word: "<extracted bridge word, or
  omit for a structural match>"

**Near misses**
- → `job-scheduling` — "<verbatim quote that brushed against it>"

**Rationale**: <one paragraph, in the language of the evidence it cites>

**Skiena**: `<the entry's Skiena cross-reference slug>`

#### <next candidate>
...

**Top pick**: `<strongest-evidenced algorithm_id — every tied id on a tie>`

**Honest misses**
- <problem-signalling word noticed but no catalog hit — the "unknown algorithm"
  flag for deeper modeling>
```

Cross-area observations at the end (only when multiple areas were scanned),
same pattern — YAML then its rendering:

```yaml
cross_area_observations:
  - id: "obs_1"
    note: "<plain prose, in the language of the evidence it cites>"
    algorithm_id: "topological-sort"
    areas: ["...", "..."]
    seam_type: "consolidation | seam | shared_kernel"
```

**Rendered**

```markdown
| Observation | Algorithm(s) | Areas | Seam type | Note |
|---|---|---|---|---|
| obs_1 | `topological-sort` | area-a, area-b | consolidation | <plain prose, in the language of the evidence it cites> |
```

`seam_type` is `consolidation` or `shared_kernel` (the same algorithm, ≥medium,
in ≥2 areas — a merge signal, or one implementation shared across boundaries
that stay) or `seam` (areas share a bridge-word cluster but resolve to
different algorithms — a boundary diagnostic).

### 6 — Quote verification

Every candidate rests on its quotes, so verify them before presenting:

1. Search each quote verbatim in the source it cites.
2. Found → keep it. Not found → drop the quote; a candidate left without a
   quote goes with it.
3. A quote from a source that cannot be searched — requirements recovered from
   the conversation rather than from a file — stays, marked `unverified: true`,
   and is named in the key findings.
4. A location follows its source: a line number only where the source carries
   stable line numbers, otherwise a section or heading reference, otherwise
   `loc: null`.

### 7 — Present

```
Algorithm scan complete

Scanned:  <N areas | single domain>
Matched:  <algorithm_ids with ≥medium confidence>
Honest misses: <problem-signalling words with no catalog entry>

Report: <resolved report path>

Key findings:
- <top 2–3 insights: strongest candidates, cross-area unifiers, surprises>
```

## Hard rules

- **Read-only on inputs**: only write to `algorithms/scan.md` (or the caller-named
  output directory).
- **Evidence rule**: every candidate cites ≥1 verbatim quote with source + loc.
  No quote → no candidate.
- **Verified quotes**: search every quote verbatim in its source before
  presenting. A quote that is not found is dropped; a quote that cannot be
  searched carries `unverified`. Never invent a location.
- **Language**: English YAML keys, Markdown headers, and table columns; quotes
  verbatim in their source language; never translate quotes. Prose written by
  the scan — `rationale`, `note` — follows the language of the evidence it
  cites; on a mixed-language corpus, the language of the quotes it rests on.
- **No prescription**: do not recommend tactical patterns or draw boundaries.
  Surface candidates only.
- **No invented bridge-words**: never claim a word appears that does not. This
  bounds the *lexical* mechanism only — you may still raise a candidate by
  structure alone (`match_basis: structural`), evidenced by a quote that shows
  the structure.
- **Two equal mechanisms, union semantics**: a candidate is matched if either the
  lexical pass OR your structural reading fires. Never require both; agreement
  only raises confidence.
- **Honest misses**: if a word that signals a canonical problem matches no
  catalog entry, list it under `honest_misses[]` (and its rendered **Honest
  misses** bullet list) — it is data, not noise.
- **Confidence calibration**: do not inflate. `low` is honourable; `high`
  requires both mechanisms on the same algorithm, each carrying its own quote.

## Anti-patterns

| Anti-pattern | Why wrong | What to do instead |
|---|---|---|
| Score every area "high" on Topological Sort because everything has dependencies | A "dependent" bridge-word is too common; requires a specific structural match | Require an explicit precedence/structural cue for high |
| Claim NP-hard reductions on weak evidence | Solving and pattern-picking is a deeper step, not this advisory scan | Stop at "candidate algorithm + mapping". Do not solve. |
| Use single-language keyword search | Sources are often mixed-language | Semantic match across all languages present |
| Drop the bridge-word verbatim quote | A later reader needs to verify the trigger | Always preserve the quote |
| Require both mechanisms to agree, or drop a lexical-only hit because your reading "didn't see" the structure | Union, not intersection — demanding agreement silently drops fits only one lens caught | Either mechanism firing = candidate; keep a word-only fit at `medium`/`low` with `match_basis: bridge-word`; only a word with NO catalog entry goes to `honest_misses` |
| Map across areas without flagging the seam | Cross-area mappings are valuable boundary input | Always add a `consolidation` or `seam` row to the cross-area observations table |

## How to run and verify

1. Provide the domain requirements (argument, conversation context, or an areas
   file in the working directory).
2. Confirm the report:
   - each area carries a YAML block followed immediately by its Markdown
     rendering (candidates table + a Domain → Algorithm Mapping table per
     candidate), and the two agree
   - every quote resolves verbatim in the source it cites, or carries
     `unverified`
   - `honest_misses[]` / **Honest misses** is non-empty if any
     problem-signalling word in the input has no catalog hit
   - no writes outside `algorithms/` (or the caller-named output directory)
