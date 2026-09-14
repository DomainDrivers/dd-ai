---
name: evidence-corpus
description: "Quote bank format and citation rules. Every claim must cite >=1 corpus quote; naming a candidate area requires 3 strong quotes from >=2 sources. Preserves quotes verbatim in their source language. A standalone quote bank any analysis step can cite."
---

# Evidence Corpus

## Purpose

The complete, single source of truth for everything citeable in the domain
materials. Any step that reasons about the domain works **only** from this
corpus — a statement that is not here is invisible to it. The corpus is
therefore **exhaustive, not a representative sample**: every concrete,
domain-bearing statement in every source must appear, or it is silently lost.
Inline user answers become first-class corpus entries.

## File location

`corpus.md` in the working directory (or the output path you chose).

## Format

```markdown
# Evidence Corpus

**Sources**: <N> files
**Coverage**: <one line on what was carried in full, e.g. "every lane in flow
order, every pivotal-event chain, every hot-zone question">
**Last updated**: <ISO timestamp>

## <relative path to source>

**Type**: <interview | docs | meeting | storming-image | storming-export | notes | user-supplied>
**Language**: <ISO code, e.g. pl, en, de, multi>

> Quote text here, preserved verbatim in original language.
— L42–L45

> Another quote.
— L88

> Multi-line quote
> kept as a single block.
— L120–L124

## <basename>.es.md  (vision-extracted from an ES board image)

**Type**: storming-image (vision-extracted)
**Original**: <path to source image>

> Event: "Return request submitted"
— Pivotal events section

> Hot zone: "Warehouse and support disagree on refund eligibility"
— Hot zones section, central cluster

## user-supplied (inline answers)

**Type**: user-supplied
**Authored**: <ISO timestamp>

> The regional manager optimizes the whole store network; the store manager
> optimizes a single store.
— Q2 inline answer
```

## What the corpus must contain

The default is **include**. Completeness is the goal: anything left out cannot
be cited and is lost to every step that reads the corpus, so when in doubt
whether a statement carries domain information, include it.

A statement is in-scope when it is:

- **Concrete** — references named things, numbers, processes, decisions, or
  domain language. Not "the system is complex" but "rules change monthly".
- **Evaluable** — it can be used as evidence for or against a claim.
- **Domain-language** — uses words the business uses, not generic filler.

Exclude **only** genuine non-information:

- Meta-commentary about the meeting itself ("let's continue", "back to you").
- Verbal filler ("um", "you know", "right?").
- Pure agreement with no information ("yeah, exactly", "true").
- Tooling/process talk ("can you share your screen?").
- Pure notation — a board's own legend / colour key, or a lane's actor label.
  These describe the source's structure, not a domain requirement, and the
  per-source extraction already records them.

Everything else goes in. Quote length: typically 1–3 sentences. Longer only if
needed to preserve meaning. Never paraphrase — if it's not in the source, don't
write it.

### Already-structured sources (Event-Storming extractions, glossaries, specs)

Some sources arrive already curated: an ES extraction has had the noise removed,
so every remaining sticky is a concrete command, event, policy, aggregate,
condition, pivotal event, or hot-zone question. For these, the selectivity above
does not apply — there is no filler left to drop, so any sticky you omit is a
lost requirement. Carry the source in **full**: every lane's flow, every
pivotal-event chain, every hot-zone question.

Preserve the lane sequence — capture each lane's flow as one ordered block (one
sticky per line, verbatim), `loc` = the lane name. Carry each pivotal-event
chain and each hot-zone question as its own entry, since each is a first-class
signal that something else may cite directly.

## Citation rules

Every claim that touches the domain must cite the corpus. The bar scales with
the claim's weight:

| Claim type | Minimum citation |
|---|---|
| Mentioning a concept | 1 quote |
| Naming a candidate area | 3 strong quotes from ≥2 sources |
| Marking a strategic question **answered** | every item of its gap checklist covered by ≥1 quote |

A claim below the bar is reported as a gap, carrying the quotes it does have,
rather than asserted with weakened confidence.

### What makes a quote strong

A quote is **strong** evidence for a claim when it states something evaluable
about the subject: a rule, a number, a threshold, a decision, a process step, a
pain, a deadline, or who is responsible. A quote that only names the subject in
passing is a **mention** — it shows the vocabulary exists, not that a business
area sits behind it.

Mentions stay in the corpus and stay citeable; they do not count towards the
candidate-area bar. Ten mentions of a word are still no evidence that the word
names an area.

## Citation format

In prose:

```
Refund Eligibility is Complex — rules change frequently (`<source>` L42) and
teams disagree on the criteria (`<source>` L88).
```

In JSON output:

```json
{
  "evidence": [
    {"source": "<path>", "loc": "L42", "quote": "..."},
    {"source": "<path>", "loc": "L88", "quote": "..."}
  ]
}
```

`loc` formats:

- `L42` — single line in a text file
- `L42-45` — line range
- `<section-name>` — for files with headed sections (e.g. ES extractions)
- `<json-path>` — for `.json` exports (e.g. `events[3].label`)
- `region:<description>` — for image-derived quotes (e.g. `region:top-left`)
- `p12` / `p12-13` — page or page range in a PDF; add `¶3` for the paragraph
- `Q<id> inline answer` — for user-supplied

## Superseded quotes

When the user resolves a contradiction, the statement that lost stays in the
corpus — nothing is deleted — but it carries a marker under its `loc`, so a
later step does not cite it as the rule in force:

```
> <the statement that no longer holds, verbatim>
— L66–L69
[superseded by C3 — see the caller's answer log]
```

A marked quote stays quotable as history — what the source said — never as what
holds today. The resolution itself enters the corpus as a user-supplied entry.

## Source language preservation

Quotes are stored in their source language verbatim. Never translate.

If another artifact needs a different language (e.g. an English ADR citing
a non-English interview), it includes the verbatim quote in its original
language and may optionally add a bracketed translation, marked as such:

```
> "<verbatim quote in source language>"
> [translation: <English gloss>]
— <source-file> L<line>
```

The translation is not part of the corpus and not a citation source.

## Cross-references

- Any analysis step that makes a domain claim cites this corpus by `source +
  loc`, at the strength set in the citation-rules table above.
