---
name: contradiction-finder
description: "Find statements in an evidence base that cannot all be true — a rule stated two incompatible ways, two numbers/thresholds that disagree for the same thing, a decision that violates a stated constraint, mutually exclusive requirements, or a later statement that silently reverses an earlier one. Returns each conflict with every clashing statement quoted verbatim and anchored by source + loc, one line on why they cannot co-exist, and a clarifying question. Detects and asks — never resolves, never edits the source. Reusable and project-agnostic."
---

# Contradiction Finder

Find statements that **cannot all be true at once**. This is not a summary
and not a decision — it surfaces conflicts a naive read would launder into a
consistent-looking whole, and hands each one back as a conflict and a
clarifying question. Whoever decides which statement holds is the human, not
this skill.

This skill is self-contained capability knowledge. It does not know which
caller invokes it, nor where its outputs are stored — the caller owns that.

## Core principle

**Detection is not resolution.** A contradiction means two or more statements
that cannot co-exist; the material itself is ambiguous by definition, so the
skill can never say which one is right. Its only job is: find the conflict,
quote every side verbatim, and ask. Picking a winner is forbidden.

## Inputs

The evidence base is either:

- a **curated quote bank** — quotes carrying `source` + `loc` references; or
- one or more **raw source documents** (narrative or structured).

Use whichever the caller provides, and cite every clashing statement by
`source + loc`. Scan **within each source AND across all sources** — a
cross-source contradiction (one source says X, another says not-X) is the
highest-value find and is invisible if you only check one source at a time.

Contradictions can involve any statement-bearing material — interview lines,
meeting notes, a number on a board export, a labelled diagram. This check
applies regardless of whether the source is narrative; a figure on a structured
export that disagrees with an interview is a genuine conflict.

## What counts as a contradiction

Two or more statements that cannot all hold. Look for:

- the **same rule stated two incompatible ways** ("always X" here, "X except Y"
  there);
- **two numbers / thresholds / limits / deadlines for the same thing** that
  disagree;
- a **decision that violates a constraint** stated elsewhere;
- **mutually exclusive requirements** (both demanded, both cannot be satisfied
  together);
- a **later statement that silently reverses an earlier one** without
  acknowledging the change.

## What does NOT count (avoid crying wolf)

- **Two statements about different things** that merely use similar words —
  verify they refer to the same subject before calling it a conflict.
- **An acknowledged change** — a statement that openly supersedes an earlier
  one ("we used to do X, now we do Y") is a decision, not a contradiction. If
  it is unclear whether the change was deliberate, that uncertainty is itself
  the clarifying question.
- **Differing opinions or preferences** — two people wanting different things is
  not a contradiction in what the system must do. But two people stating
  incompatible **rules the system must honour** *is* one. Distinguish a clash of
  preference from a clash of requirement.
- **A vague statement next to a precise one** on the same point — that is an
  under-specification, not a contradiction; it is out of scope for this skill.

Report only genuine conflicts. Never manufacture one to look thorough.

## Procedure

1. **Inventory the statements.** Per source, list the rules, constraints,
   decisions, thresholds, behaviours, deadlines, and responsibilities each one
   asserts (explicit or clearly implied). Tolerate transcription artifacts — do
   not over-read exact ordering when a quote came from speech-to-text.
2. **Compare for incompatibility** — within each source and across all of them.
   For each pair or cluster of statements that touch the same subject, ask: can
   these all be true at the same time? If not, you have a candidate.
3. **Confirm and record.** For each genuine conflict, capture **every** clashing
   statement verbatim with its `source + loc`, state in one line why they cannot
   co-exist, set a severity, and write a clarifying question asking **which one
   holds** (or how they reconcile).

## Output contract (shape, not location)

The caller decides where this is written and who reads it. This skill only
fixes the shape. One entry per conflict:

```json
{
  "id": "<stable slug>",
  "title": "<short: the two sides, e.g. 'retention 30d vs 90d'>",
  "severity": "critical | high | medium | low",
  "statements": [
    {"source": "<source file>", "loc": "<line range or section>", "quote": "<verbatim, source language>"},
    {"source": "<source file>", "loc": "<line range or section>", "quote": "<verbatim, source language>"}
  ],
  "why_incompatible": "<one line: why they cannot all hold>",
  "diagnostic_question": "<which one holds, or how do you reconcile them?>"
}
```

- `statements` carries **≥2** entries; entries may name **different** sources
  (that is the cross-source case, and the most valuable).
- **Severity**: a conflict a boundary, an owner, or a core rule depends on =
  `high`/`critical`; a peripheral one = `low`/`medium`.

A human-readable report should order conflicts by severity and render each as a
row that lists, per conflict, the `source loc` of every statement it rests on —
so each row links straight back to the materials.

## Diagnostic-question rules

Every conflict **must** carry one. It must be:

- **specific** — names both clashing statements, not "tell me more";
- **verifiable** — asks for a decision or a fact, not an opinion;
- **non-accusatory** — opens the question, never assigns blame for the conflict.

For a contradiction the question always asks **which statement holds** (or how
the two reconcile). Never drop a conflict without one — the question is the
entire point.

## Strict rules

- **Never resolve a contradiction.** Detect it, then hand it back as a
  clarifying question — only the human can say which statement holds.
- **Never edit the source or quote store.** Return conflicts; the caller acts.
- **Never translate quotes.** Verbatim, source language preserved.
- **All prose English.** No language-specific keyword lists — semantic
  understanding only.

## Pitfalls

- **Crying wolf.** Differing preferences, acknowledged changes, and merely
  similar wording are not contradictions. When in doubt, verify the two
  statements share a subject before reporting.
- **Same-subject test.** Two thresholds disagree only if they govern the *same*
  thing. Confirm the subject matches before calling it a conflict. When the
  evidence base is a quote bank and the source file is at hand, read the few
  lines around each quote first: an "always X" often carries its "except Y" in
  the next sentence, which the quote alone hides.
- **Transcription artifacts.** Speech-to-text can split or reorder speech; do
  not read a contradiction into an ordering the transcription invented.
- **Assuming bad intent.** A reversal may be an honest update, not a mistake —
  flag and ask, never assert that someone contradicted themselves on purpose.
