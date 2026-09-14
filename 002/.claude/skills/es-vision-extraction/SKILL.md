---
name: es-vision-extraction
description: "Vision-based extraction of structured content from Event Storming board images, read as a numbered flow sequence per lane (not bucketed by sticky type). Defines the extraction prompt, caching strategy, and user review gate. Used when the materials contain ES exports as images."
---

# Event Storming Vision Extraction

## Purpose

Turn an Event Storming board image into structured, citeable content read
**as a process narrative**: each lane (swim-lane / area) walked in flow order,
every sticky numbered and tagged with its type inline. The result is cached and
reviewed by the user before it feeds the corpus.

The board tells a story — e.g. `order placed` (event) → `reserve stock`
(command) → `is the item still available?` (risk). Extracting that ordered
sequence preserves the cause-and-effect chain that type-bucketing throws away.

## When to invoke

- A materials file is an Event Storming board image (`.png` / `.jpg` / `.jpeg`).
- No matching `<basename>.es.md` cache exists (or its `source_hash` doesn't match
  the current file).

## Extraction prompt (use verbatim)

```
Look at this Event Storming board image. Extract its content exhaustively,
as an ordered flow — do not summarize, and do not bucket stickies by type.

Step 1 — Identify lanes (areas).
Find the swim-lanes / areas on the board: horizontal bands, labeled regions,
or clearly separated clusters. Name each one (use the lane's own label
verbatim if it has one; otherwise describe its position, e.g. "top band,
left"). If the board has no visible lanes, treat the whole board as a single
lane in temporal flow order.

Step 2 — Walk each lane as a numbered sequence.
For each lane, follow the flow (left-to-right, along arrows / temporal
direction) and list every sticky in order. Number them starting at 1. After
each sticky's verbatim text, tag its type in parentheses.

Recognized types (pick the closest match; the sticky's colour usually
indicates it — use the board's own legend if one is present):
event, command, actor, policy, read model, external system,
aggregate / business rule, hot spot / pain, question, risk,
pivotal event, opportunity, note.

Output sections (use exactly these headings):

## Legend
If the board shows a colour key / legend, transcribe it verbatim (colour →
meaning). Otherwise write "none visible".

## Lanes
- <lane 1 name>
- <lane 2 name>
...

## Flow
### Lane: <lane 1 name>
1. <verbatim sticky text> (event)
2. <verbatim sticky text> (command)
3. <verbatim sticky text> (risk)
...

### Lane: <lane 2 name>
1. <verbatim sticky text> (event)
...

## Pivotal events
Index only — point into the flow, do not re-describe. For each pivotal event
(process turning point), reference it as "<lane> #<n>", repeat the verbatim
text, and note what flows into and out of it.

## Hot zones
Index only. For each cluster of pain / question / risk stickies: reference the
flow steps it spans ("<lane> #<n>–#<m>"), give its spatial location on the
board, repeat the verbatim sticky text, and name the topic the pain is about.

## Vocabulary
Domain terms visible on stickies that did NOT become a numbered flow step
(e.g. floating labels, annotations). Preserve original language.

## Layout observations
Spatial structure beyond individual stickies: overall flow direction, how the
lanes relate to each other, gaps, and anything structural.

Rules:
- Preserve text verbatim in its original language. Do not translate.
- Keep flow order. If the order between stickies is ambiguous, list them by
  left-to-right position and mark "[order unclear]".
- Do not invent content. Mark uncertainty with [illegible] or [partial].
- Do not interpret a sticky. Tag its type and record its text; do not explain
  what it means.
```

## Cache file format

`<basename>.es.md` (next to the corpus):

```markdown
---
source: <path to source image>
extracted_at: <ISO timestamp>
source_hash: <sha256 of source image bytes>
extraction_skill_version: es-vision-extraction-v2
---

## Legend
- ...

## Lanes
- ...

## Flow
### Lane: ...
1. ... (event)
2. ... (command)
3. ... (risk)

## Pivotal events
- ...

## Hot zones
- ...

## Vocabulary
- ...

## Layout observations
...
```

## User review gate (mandatory before the extractions feed the corpus)

After extracting all new ES images in one pass, present them as a batch. The
caller owns the gate UI; whatever shape it takes, it must let the user accept the
batch, send selected images back for re-extraction, or stop. A caller may also
offer excluding an image from the corpus.

Show what a reader can check in a minute: the counts, a sample, and how much of
the board came back unreadable. A gate that asks someone to skim six 100-line
files is waved through unread, and a bad extraction then travels into everything
downstream.

```
## Event Storming extraction review

Extracted content from <N> board image(s). Each is cached as
<basename>.es.md.

- <basename1>.es.md — {lane_count} lanes, {step_count} flow steps,
  {pivotal_count} pivotal events, {unreadable_count} stickies marked
  [illegible] / [partial]
  Sample: "<sticky>" ({lane} #{n}) · "<sticky>" ({lane} #{n}) · "<sticky>" ({lane} #{n})
- <basename2>.es.md — ...

Check the three sampled stickies against the board. Then accept, or name the
files to re-extract.
```

Draw the three sampled stickies at random from the flow steps, from different
lanes when the board has more than one — a sample the caller picks is a sample
that hides its own mistakes.

**Unreadable threshold.** When more than roughly one sticky in ten came back
`[illegible]` or `[partial]`, say so at the gate and ask for a better input
before accepting: a board export, or one crop per lane. Re-extracting the same
downscaled photo cannot recover text the image does not hold.

Iterate until the user accepts. When the gate takes a dictated correction, write
it into the cache file and append `patched_at` to the frontmatter.

## Re-entry behavior

- If the source image changes (hash mismatch with cache): invalidate just that
  cache file and re-extract. Other images and their reviews are unaffected.
- If this skill's prompt changes (version bump): warn the user that existing
  caches came from an older prompt version. Do not auto-invalidate — let
  the user decide whether to re-extract.

## Cross-references

- The `evidence-corpus` skill reads these cache files as corpus sources. The
  numbered flow steps are citeable as `<basename>.es.md → <lane> #<n>`; the
  Pivotal events and Hot zones indexes are indexed for whoever cites them.
- Used by the `requirements-analyzer` agent during ES-image extraction.
