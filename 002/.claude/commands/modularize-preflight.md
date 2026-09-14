---
description: "Pre-flight gate — decide whether a domain is worth modularizing at all. Scans materials, asks three safeguard questions, produces a proceed/refuse verdict."
argument-hint: "[materials-path] [output-path]"
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
---

Use the **pre-flight-gate** skill (registered in `.claude/skills/`): follow it
exactly as written, against these arguments: $ARGUMENTS

The first argument is the materials path (default: the current directory); the
optional second is the output directory (default: the current directory). The
skill owns the whole procedure — the three safeguard questions, the verdict
table, the mandatory approval gate, and the two output files. This command is
just the launcher.
