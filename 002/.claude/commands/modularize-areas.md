---
description: "Turn domain materials into a 5-question strategic 'show-me-the-money' report answered first, then candidate business areas grounded in those answers. Builds an evidence corpus, extracts ES images, scans sources for contradictory statements, and freezes structured artifacts behind manual gates."
argument-hint: "[materials-path] [output-path]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, Task, AskUserQuestion
---

Run the **requirements-analyzer**: read `.claude/agents/requirements-analyzer.md`
and follow it exactly as written, inline, against these arguments: $ARGUMENTS

The first argument is the materials path (default: the current directory); the
optional second is the output directory (default: the current directory).

Run it inline (do not spawn it as a sub-agent): the agent needs to ask you `AskUserQuestion`
gates and to spawn the `sources-critic` sub-agent itself, neither of which
works from a headless sub-agent. The agent owns the whole procedure — corpus,
ES-image extraction, the contradiction scan, the 5-question strategic scan asked
first, the candidate-area clustering that follows, the manual gates, and the
frozen `areas.{md,json}` + `corpus.md` outputs. This command is just the
launcher.
