# Contradiction report

Statements in the evidence corpus that cannot all be true. Each row anchors every
clashing statement back to the corpus by `source + loc`, states in one line why
they cannot co-exist, and carries a clarifying question. This is a
source-hygiene signal, not a settled conclusion: the conflicts are detected and
handed back as questions — only a human can say which statement holds.

- Corpus: `intake/corpus.md` (sha256 `1ef85ed8…cd0038`)
- Sources scanned: `intake/wywiad-z-prezesem.md`, `intake/big picture.es.md`, `intake/process level.es.md`
- Conflicts: 3 (high 2, medium 1) — 3 clarifying questions

| # | Conflict (severity) | Statements (corpus) | Why incompatible | Diagnostic question |
|---|---------------------|---------------------|------------------|---------------------|
| 1 | Risk-mitigation moves: suggest-only vs automatic (**high**) | wywiad L77; wywiad L53–L54; process level.es.md `Lane: Reschedule / reallocation flow`; process level.es.md `Hot zones` | The interview requires the system to only suggest/propose risk-mitigation moves for a human to decide and execute; the process-level board has a weekly policy that adds the resource automatically. The same mitigation cannot be both human-decided-only and automatic. | For risk mitigation, should the weekly risk policy automatically add/substitute a resource, or should the system only suggest the move and wait for a human to decide and execute it? Which behaviour holds? |
| 2 | Worker without qualifications/skills: forbidden vs accepted (**high**) | wywiad L20–L22; wywiad L65–L69 | One statement forbids assigning any worker lacking required qualifications (skills) as well as permits; the other explicitly accepts assigning a worker without the required skills and blocks only a missing permit. A skill-less assignment cannot be both forbidden and accepted. | Can a worker lacking the required qualifications/skills be assigned to a project as an accepted risk, or is that a hard block like a missing permit? Which rule holds for the skills case specifically? |
| 3 | Order: resource check gates stage planning vs stages planned first (**medium**) | big picture.es.md `Pivotal events`; big picture.es.md `Lane: aktor: maneger projektu — planning sub-flow`; process level.es.md `Lane: Scheduling flow — automatic mode` | The big-picture board makes a resource-availability/potential check gate whether stage planning begins at all; the process-level board defines and parallelizes stages first and adds/checks resources only afterwards at allocation. Resource availability cannot both precede and follow stage planning. | Does an upfront resource-availability/potential check gate whether stage planning begins (plan stages only once resources exist), or are stages defined and parallelized first with resource availability checked later at allocation? Which ordering holds? |
