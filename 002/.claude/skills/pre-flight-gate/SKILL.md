---
name: pre-flight-gate
description: "Pre-flight gate — decide whether a domain is worth modularizing at all, before any analysis. Scans materials, asks three safeguard questions (via AskUserQuestion), and produces a proceed/refuse verdict. Writes nothing until you approve."
---

# Pre-flight Gate

Core stance: *modularization is a tool for fighting complexity, not a
goal.* Refuse early when modularizing would just be paying a structure tax on a
problem with no real complexity ("robbing the project sponsor"). This decides
exactly one thing — **proceed** or **refuse** (with a reason). It does not
enumerate areas or classify Cynefin.

## Inputs (passed by the caller)

A path to the domain materials to scan (default: the directory you run from),
and optionally a second path for where the verdict files should land (default:
the same directory). Materials are read-only — never edit them.

## Step 1 — Scan the materials (hints only)

Read the materials path recursively (skip `.DS_Store` and OS junk). Note file
types and a rough word count. Scan for the three refusal signals below and
collect them as **hints that seed the conversation — they never decide the
verdict**. Human context outranks AI signals.

| Case | What to look for |
|---|---|
| **PoC / throwaway** | "PoC", "proof of concept", "prototype", "MVP to validate", "throwaway", "spike", "we'll rewrite it later" |
| **Pre-scoped as one module** | scope limited to one component / microservice / plugin / library / "extension of the existing X" |
| **No complexity** | very narrow scope (a few screens), CRUD-only language, no expert disagreement, no rule churn, single stakeholder perspective |

## Step 2 — Ask the three safeguard questions

Always ask all three, even when the signals look clear — the human has context
the materials don't. Ask them in **one `AskUserQuestion` call** (three
questions). The auto-provided "Other" option covers any free-text answer.

1. **Lifecycle** — "Is this a PoC / prototype / throwaway?"
   - `Production` — ships to production, developed for years
   - `PoC / throwaway` — validate an idea, then discard
   - `Ad-hoc tool` — internal tool, no development plan
2. **Scope** — "Is the scope pre-determined as a single module?"
   - `Free hand` — I decide how to divide it
   - `Extends one module` — must extend one existing module
   - `Single unit` — must be one library / microservice / plugin
3. **Complexity** — "Do you expect real business complexity?" (signals:
   multiple stakeholder perspectives, frequently changing rules, experts
   disagree, edge cases outnumber rules, long-term maintenance)
   - `Yes — clear` — clear complexity signals
   - `Probably not` — small / well-understood / procedural
   - `Not yet known` — still in initial exploration

If the user picks **"Other"** on any question, ask one clarifying follow-up
before deciding — do not auto-decide on a free-text answer.

## Step 3 — Compute the verdict

| Lifecycle (Q1) | Scope (Q2) | Complexity (Q3) | Verdict |
|---|---|---|---|
| Production | Free hand | Yes — clear | **proceed** — the method is justified |
| Production | Free hand | Not yet known | **proceed** but warn — Q3 may collapse to "no complexity" once the actual areas are on the table |
| PoC / Ad-hoc tool | * | * | **refuse — throwaway** — no long-lived development plan to repay the structure tax |
| * | Extends / Single unit | * | **refuse — pre-scoped** — suggest tactical work inside the existing module |
| * | * | Probably not | **refuse — no complexity** — suggest skipping modularization |

When two refusals both fire, report both but pick the most fundamental as
primary: **throwaway > pre-scoped > no-complexity**.

## Step 4 — Manual gate (mandatory)

Print the proposed verdict, the three answers, and a one-paragraph rationale
stating the complexity-tool stance. Then call `AskUserQuestion` once to gate it:

- `Confirm` — write the verdict files
- `Override verdict` — I disagree; set a different verdict (your context
  outranks the signals)
- `Adjust an answer` — change one of Q1–Q3 and recompute

**Write nothing until the user approves.** On "Adjust", loop back to the
relevant question; on "Override", take the user's verdict verbatim.

## Step 5 — Write the verdict (only after approval)

Write `pre-flight.md` to the output directory:

```markdown
# Pre-flight Gate Verdict

**Verdict**: <proceed | refuse-throwaway | refuse-pre-scoped | refuse-no-complexity>
**At**: <ISO timestamp>

## Answers

- Q1 (PoC / throwaway?):  <answer + the user's elaboration>
- Q2 (pre-scoped?):       <answer + the user's elaboration>
- Q3 (real complexity?):  <answer + the user's elaboration>

## Rationale

<one paragraph stating the stance: modularization is a tool for fighting
complexity, not a goal.>

## If you disagree

Re-run this gate with different answers, or override the verdict directly. A
`proceed` says the answers justify modularizing this system.
```

And `pre-flight.json` (the machine-readable mirror — self-contained, so it can be
read on its own without this skill for context):

```json
{
  "schema_version": "1.0",
  "artifact": "pre-flight-gate-verdict",
  "description": "Result of the pre-flight gate: a proceed/refuse decision on whether this domain is worth modularizing, with the three safeguard questions, their chosen answers, and the rationale. Self-contained — readable on its own, without this skill for context.",
  "verdict": "proceed",
  "at": "<ISO timestamp>",
  "questions": [
    {
      "id": "q1",
      "topic": "Lifecycle",
      "question": "Is this a PoC / prototype / throwaway?",
      "answer_code": "a",
      "answer_label": "Production",
      "answer_meaning": "ships to production, developed for years",
      "elaboration": "<user free text, or null>"
    },
    {
      "id": "q2",
      "topic": "Scope",
      "question": "Is the scope pre-determined as a single module?",
      "answer_code": "a",
      "answer_label": "Free hand",
      "answer_meaning": "I decide how to divide it",
      "elaboration": null
    },
    {
      "id": "q3",
      "topic": "Complexity",
      "question": "Do you expect real business complexity?",
      "answer_code": "a",
      "answer_label": "Yes — clear",
      "answer_meaning": "clear complexity signals",
      "elaboration": null
    }
  ],
  "secondary_reasons": [],
  "warnings": [],
  "rationale": "<one paragraph English>",
  "signals_seen": ["<e.g. 'word: prototype @ notes.md L12'>"]
}
```

Fill every `questions[]` entry from what was actually asked and answered: copy
the `question` text and the chosen option's `answer_label` + `answer_meaning`
verbatim from Step 2, set `answer_code` positionally (first option = `a`, second
= `b`, third = `c`), and put any free-text answer in `elaboration` (with
`answer_code: "other"` when the user picked "Other"). The point is that the JSON
stands on its own — a reader gets the full question and answer, not a bare code.
`signals_seen` is the raw scan output, kept for audit only — the
verdict comes from the answers, not the signals. A "proceed but warn" row puts
its warning text in `warnings[]`.

## Step 6 — Report

State the verdict in one line and where the two files landed. On a refusal, give
the recommended alternative. On a `proceed`, say the materials are clear to
analyze.

## Do not

- Do not enumerate business areas — out of scope for this gate.
- Do not skip any of the three questions, even when signals look clear.
- Do not refuse over a language barrier — materials in any language are fine.
- Do not decide from signals alone — signals open the conversation; the user's
  answers close it.
- Do not write the artifacts before the user approves at the gate.
