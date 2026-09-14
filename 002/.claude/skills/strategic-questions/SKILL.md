---
name: strategic-questions
description: "Strategic question set used to validate that the materials are complete enough to model. Detects gaps in evidence about value drivers, risks, integrations, regulations, and metrics. Default set is the show-me-the-money five — swap this file's `questions:` block to use a different set per project."
---

# Strategic Questions (intake validation)

## Purpose

Define the strategic-question set used to scan the evidence corpus for blind
spots. Each question gets a status (answered / partial / missing / deferred /
n_a). The set is pluggable: change `questions:` to swap.

## The default question set (show-me-the-money)

```yaml
provenance: "Default strategic discovery set — adapt freely per project."
questions:
  - id: q1
    short: "Revenue and savings"
    full: "Where does revenue come from? Where does the project save money?"
    semantic_intent:
      - statements connecting features or processes to money in or out
      - explicit monetary figures or unit economics
      - cost-benefit comparisons, margin claims, savings rationale
      - identification of who is paying or being charged

  - id: q2
    short: "Risks and opportunities"
    full: "Where are the business and technical risks? Where are the opportunities?"
    semantic_intent:
      - statements about what might go wrong (failure modes, exposure)
      - dependency risks (key person, single supplier, single tenant)
      - language about untapped markets, growth, leverage, expansion
      - regulatory or reputational hazards

  - id: q3
    short: "Integrations"
    full: "What external systems do you integrate with — now and in the future?"
    semantic_intent:
      - mentions of third-party systems by name or type
      - import / export / sync / replication of data
      - protocol mentions (HTTP API, file drop, message queue, webhook)
      - statements about data flowing in or out of the system

  - id: q4
    short: "Regulatory constraints"
    full: "What regulations, compliance regimes, or legal constraints apply?"
    semantic_intent:
      - named regulations or industry-specific rules
      - data residency, retention, deletion, or access rules
      - audit-trail or traceability requirements
      - consent, licensing, or contractual obligations

  - id: q5
    short: "Optimization target and business metrics"
    full: "What business metric is the project optimizing? How is it measured, what threshold matters, and who decides when metrics conflict?"
    semantic_intent:
      - business metric definitions (margin, utilization, time-to-X, win rate,
        churn, throughput-of-business-outcomes, error/rework cost, NPS)
      - explicit numbers: thresholds, targets, baselines, units
      - ownership of the metric (which role watches it, where it lives —
        dashboard, monthly report, board deck)
      - conflict-resolution rules when two stakeholders optimize different
        metrics (e.g. a regional manager's network-wide margin vs a store
        manager's single-store margin — which wins, by what rule, with what
        arbitration mechanism)
    not_this_question:
      - pure technical SLAs (request latency, uptime %, API p99) — those are
        engineering NFRs, not the business metric the system optimizes for.
        Capture them later if they constrain the architecture, but don't let
        them satisfy Q5
```

## Status rubric

For each question, classify status using the corpus:

Write the question's gap checklist first — the items an answer has to cover, in
the `gap_description` style below — then set the status from how much of that
checklist the corpus covers:

| Status | Rule |
|---|---|
| **answered** | every checklist item is covered by ≥1 corpus quote |
| **partial** | some items covered, others open — name the open ones |
| **missing** | nothing in the corpus covers any item |

Coverage decides, not volume. Ten quotes repeating one fact leave the question
`partial` while the rest of the checklist stays untouched, and one precise quote
closes a one-item checklist. What stays unticked is both the status and the text
the user reads at the gate.

The rule is also **source-agnostic**. Intake materials are often single-source
(one interview plus board images), so counting source diversity here would flag
thin materials rather than a genuine evidence gap.

Two states reachable only through user interaction:

| Status | Rule |
|---|---|
| **deferred** | user chose to defer — flag areas touching this as carrying thinner evidence |
| **n_a** | user marked the question not applicable, with reasoning |

## Output schema

Each question's result:

```json
{
  "id": "q1",
  "status": "partial",
  "evidence": [
    {"source": "<path to source>", "loc": "L42-45", "quote": "..."}
  ],
  "gap_description": "Need: (1) named revenue streams, (2) what the system itself is supposed to optimize for (savings? margin? time-to-decision?), (3) who owns the P&L target."
}
```

### gap_description style (mandatory)

Phrase `gap_description` as a **concrete checklist of what evidence would close
the gap**, not as meta-prose about what is absent. The reader uses this column
to know what to answer inline.

Bad (vague meta-prose):
- "Lacks management-tier perspective on portfolio-level economics."
- "Insufficient evidence about metrics."

Good (specific, answerable):
- "Need: (1) the business metric (margin / utilization / win rate / etc.),
  (2) numeric threshold that triggers a decision, (3) the role that owns the
  metric, (4) rule for the network-vs-store conflict when both optimize
  different metrics."
- "Need: list of integrated systems by name, direction of data flow (in/out),
  and whether integration is online or batch."

Rule: a `gap_description` is well-phrased if the reader could answer it in one
paragraph and unambiguously close the gap. If too vague to answer in one
paragraph, rewrite it.

## When to swap this skill

Replace the `questions:` block when:

- The default set doesn't fit the domain (e.g. an internal compliance tool where
  Q1 "revenue" is meaningless — replace with mission/mandate questions).
- The project is in a non-business domain (research software, open-source
  infrastructure) — replace with domain-appropriate questions.
- A specific industry has standard discovery questions that should be used.

Keep the rubric and output schema unchanged across swaps — the consuming logic
depends only on their structure, not on the questions themselves.

## Cross-references

- Used by the `requirements-analyzer` agent.
- Emits a `gap_report`; the caller decides where it lands.
