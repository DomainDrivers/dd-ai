---
name: guided-conversation
description: "The inline Q&A gate pattern. How to batch evidence-anchored questions, present them to the user, wait, process the answers, loop until resolved, and terminate safely. The agent runs inline and asks the user directly."
---

# guided-conversation: the inline Q&A gate

Each agent reaches a point where it must check its analysis with the user
before freezing anything (the manual gate). This skill defines how that gate
behaves. Because the agent runs **inline** in the conversation, it asks the user
directly and waits. The loop is just: present → wait → process → re-check →
repeat until done.

## The turn loop

1. **Analyze.** Do the agent's work; identify the decisions that still need the
   user (areas to confirm, a classification to challenge, a gap to fill).
2. **Generate questions.** Turn each open decision into a question that follows
   the rules below. Batch related ones.
3. **Present and wait.** Show the batch in one message and stop. Do not freeze
   any artifact, do not guess past a blocking question.
4. **Process answers.** Apply each answer: update the working state, record the
   decision and what it was based on, and remove the resolved question.
5. **Re-check.** If the answers created new open items (a split with no "how", an
   override needing a rationale), generate only those next turn and loop from 3.
   Otherwise the gate is **done** — proceed to freeze.

## Question generation rules

- **Anchor every question to evidence** — a quote, a signal, a conflict. The
  user must understand *why* you're asking. A question with no evidence is one
  the user can't evaluate.
- **Batch related questions** (same area, related concept). Don't fragment one
  decision across several prompts. When one batch would mix two kinds of decision
  and run past the cap below, split it over two turns, the blocking kind first.
- **Limit open questions to ~3–7 per turn.** A wall of questions kills
  engagement. A review batch — one line per item in a list the user walks
  through — may be longer.
- **Provide options** when the answer space is small (yes/no, choose-one).
- **Provide a default** for the skip case, so the user can move fast.
- **Mark whether an answer blocks progress** — i.e. whether you genuinely can't
  freeze without it, versus a nice-to-have you'd otherwise default.

## When NOT to ask

- Confidence is high **and** the decision doesn't block anything → decide it and
  record the decision; don't ask.
- The answer is already in the materials → re-read the source first.
- It's stylistic (naming, formatting) → pick a sensible default and note it.
- It's mechanical / deterministic → apply the rule, don't ask.

## Always ask if

- The decision is strategic / value-based (is this a real business area? what is
  this area's knowability?).
- The materials contain an explicit conflict.
- A classification would rest on thin or low-confidence evidence.
- The user's context could plausibly overturn the AI's read.

## Termination guarantees

- **Turn cap.** After ~10 turns without convergence, stop: surface the
  unresolved items, apply the stated defaults, and ask the user to accept the
  frozen state.
- **No-progress check.** If after ~3 turns the same questions keep coming back
  unresolved, accept the defaults for them and note it, rather than looping
  forever.
- **Escape hatch.** Honour a "skip" (apply all defaults and freeze), a "stop"
  (abort without freezing), and a "redo" (re-run the current turn from scratch).

## Anti-patterns (do not do)

- A long list of open questions on the first turn → the user disengages. Hold
  the ~7 limit.
- "Do you agree with the analysis?" → too vague. Ask about specific facts.
- A question with no evidence → the user can't evaluate it. Always cite the
  source.
- No default → forces an answer. Always include the skip case.
- The same questions every turn → an infinite loop. After 3 turns, accept the
  default and note it.

## Tone

- Second person, concrete, short.
- Every question carries its evidence.
- After an answer, give a one-line "OK — based on X, I'll do Y" so the user sees
  their input landed.
- Gate templates are English; free-form conversation may follow the user's
  language. Keep the written artifacts' structure English and their quoted
  content in the source language.
