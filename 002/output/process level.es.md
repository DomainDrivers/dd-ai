---
source: intake/process level.png
extracted_at: 2026-07-07T18:40:48Z
source_hash: c3a7e6a9af9ca77bafd9891f49eaa79e9c515311628da0f7dce4015e70c0aa22
extraction_skill_version: es-vision-extraction-v2
---

## Legend

No explicit legend on the board. Colour usage inferred from sticky content and
standard Event Storming convention:
- blue → command
- orange → event
- green → read model
- yellow → policy / condition (business rule gate)
- purple → policy
- teal / turquoise → question / hot spot
- grey / white → floating question note

## Lanes

- Project intake & risk detection (top-left)
- Scheduling flow — automatic mode (centre, main line)
- Scheduling flow — manual mode (centre, below main line)
- Release resources (centre)
- Reschedule / reallocation flow (right)
- Offer evaluation (far right)
- Resources and capability management (bottom-left)
- Employee / device / material capability lane (bottom-centre)
- Floating questions (top-right and bottom)

## Flow

### Lane: Project intake & risk detection
1. Add a new project (command)
2. New project added (event)
3. Project card (read model)
4. Add demands (command)
5. demands added (event)
6. risk detected (event)
7. manager informed (event)
8. Risk dashboard (riskPushNotification) (read model)
9. resource training/moving from other project simulated (event)

### Lane: Scheduling flow — automatic mode
1. Stages (tree) (read model)
2. Define stages (tree) (command)
3. stages parallized (event)
4. stages defined (event)
5. Add resource to stage (command)
6. resource added (event)
7. Project card (read model)
8. resource training/moving from other project simulated (event)
9. Define project start date (command)
10. date >= now (policy)
11. start date saved (event)
12. stage dates (schedule) calculated (event)
13. project end date calculated (event)
14. project end date calculated (event) [duplicate sticky]
15. create project allocations (command)
16. create project allocations policy (policy)
17. allocate resources (command)
18. is resource available? (policy) [circled/highlighted on the board]
19. project allocations scheduled (event)
20. resources allocated (event)
21. equivalnet found (event)
22. resources not aquired (event)
23. demand queued (event)

### Lane: Scheduling flow — automatic mode — cost/earnings branch
1. add income and cost (command)
2. earnings recalculated (event)
3. schedule demands (command)
4. project allocations demands scheduled (event)

### Lane: Scheduling flow — manual mode
1. Define project start date (command)
2. date >= now (policy)
3. start date saved (event)
4. stage dates (schedule) calculated (event)
5. define schedule (command)
6. stage dates saved (event)
7. stage dates (schedule) manually defined (event)

### Lane: Release resources
1. release resources (command)
2. resources released (event)

### Lane: Reschedule / reallocation flow
1. edit stage duration (command)
2. Stage duration set (event)
3. later stages datess calculated (event)
4. project end date calculated (event)
5. reallocate resources (command)
6. availability check (policy)
7. resources aquired (event)
8. equivalnet found (event)
9. resources not aquired (event)
10. demand queued (event)
11. risk policy (weekly) (policy)
12. equivalnet found (event)
13. risk management policy (policy)
14. resource automatically added (event)
15. notified about equivalent found (event)

### Lane: Offer evaluation
1. is the offer evaluation profitable? (question) [arrow down into the flow]
2. new offer appeared (event)
3. offer accessed (event)
4. needed resources checked (event)

### Lane: Resources and capability management
1. resource definitions (read model)
2. Add resource definition (command)
3. Resource definition added (event)
4. resource summaries (read model)
5. Add resource instantion of given definition (command)
6. Resource instantion added (event)

### Lane: Employee / device / material capability lane
1. Employee summaries (read model)
2. Device summaries (read model)
3. Material summaries (read model)
4. Add vacation (command)
5. Vacation added (event)
6. Add sick leave (command)
7. Sick leave added (event)
8. Change skills (command)
9. Skills changed (event)
10. capability unlocked (event)

### Lane: Floating questions
1. is resource available? (question) [top-right, grey]
2. what is it the order of stages? (question) [top-right, grey]
3. does it pay off to move resource from another project? (question) [teal, over automatic scheduling flow]
4. does it pay off to terminate a project? (question) [teal, bottom]
5. does it pay to hire a contractor? (question) [teal, bottom]
6. does it pay of to remove someone? (question) [teal, bottom]

## Pivotal events

- Scheduling flow — automatic mode #18 "is resource available?" — flows in from
  #17 "allocate resources"; flows out to #20 "resources allocated" (yes) vs
  #21 "equivalnet found" / #22 "resources not aquired" → #23 "demand queued" (no).
  Turning point deciding whether allocation succeeds or a demand is queued.
- Reschedule flow #6 "availability check" — flows in from #5 "reallocate
  resources"; flows out to #7 "resources aquired" vs #9 "resources not aquired"
  → #10 "demand queued". Turning point of the reallocation branch after a stage
  duration edit shifts later dates.
- Reschedule flow #13 "risk management policy" — flows in from #11 "risk policy
  (weekly)" / #12 "equivalnet found"; flows out to #14 "resource automatically
  added" and #15 "notified about equivalent found". Turning point where the risk
  policy auto-substitutes a resource.
- Offer evaluation #1 "is the offer evaluation profitable?" — gates whether a
  newly-appeared offer is accessed and its needed resources checked.

## Hot zones

- Resource-availability decision — automatic flow #18 and reschedule flow #6.
  Stickies: "is resource available?", "availability check", "resources aquired",
  "resources not aquired", "equivalnet found", "demand queued". Topic: whether the
  resources a schedule needs are actually available, and what happens when not.
- Move-resource profitability — teal question over the automatic flow. Sticky:
  "does it pay off to move resource from another project?" (arrow into "resources
  allocated"). Topic: economic worth of pulling a resource off another project.
- Portfolio decision questions — bottom teal cluster. Stickies: "does it pay off
  to terminate a project?", "does it pay to hire a contractor?", "does it pay of
  to remove someone?". Topic: cost/benefit of terminate / hire / remove moves.
- Risk auto-management — reschedule flow #11–#15. Stickies: "risk policy
  (weekly)", "risk management policy", "resource automatically added", "notified
  about equivalent found". Topic: weekly risk policy automatically substituting
  and notifying about equivalent resources.
- Stage ordering — top-right grey question. Sticky: "what is it the order of
  stages?". Topic: the sequence/dependency ordering of project stages.

## Vocabulary

- riskPushNotification (annotation on "Risk dashboard" read model)
- Stages (tree) — stages modelled as a tree structure
- parallized (as in "stages parallized")
- equivalnet (as spelled on board; "equivalent" resource)
- aquired (as spelled on board; "acquired")
- datess (as spelled on board; "dates")

## Layout observations

- Board reads as several parallel process lanes rather than one line. The dominant
  centre line is the scheduling flow, which forks into an "automatic mode" upper
  path and a "manual mode" lower path (both start at "Define project start date"
  → "date >= now").
- Two named mode labels sit as free text on the board: "manual mode" (centre) and
  "Resources and capability management" (bottom-left).
- Read-model stickies (green) recur across lanes: "Project card" appears in both
  the intake lane and the scheduling lane; "Stages (tree)", "resource
  definitions", "resource summaries", "Employee/Device/Material summaries" anchor
  their respective lanes.
- Teal question stickies are placed above/around the flow with arrows pointing
  into it ("does it pay off to move resource from another project?" → "resources
  allocated"; "is the offer evaluation profitable?" → offer lane), marking
  economic decision points overlaid on the mechanical flow.
- Several stickies are duplicated across lanes ("project end date calculated",
  "resource training/moving from other project simulated", "equivalnet found",
  "resources not aquired", "demand queued", "date >= now"), reflecting shared
  sub-steps reused by the automatic, manual, and reschedule paths.
