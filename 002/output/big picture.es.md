---
source: intake/big picture.png
extracted_at: 2026-07-07T18:40:48Z
source_hash: abb0e4ea0778b38997abb98d01dab3816e8a1d57e4c44936667d4f9214afa834
extraction_skill_version: es-vision-extraction-v2
---

## Legend

- white → notatka (note)
- pink / magenta → ryzyko (risk)
- orange → zdarzenie (event)
- (additional colours present on the board without a legend entry: green stickies
  used for value / "what do we earn or save on" notes and a dashboard read-model;
  one salmon/red sticky; grey/white notes. Recorded inline below with their colour.)

## Lanes

- aktor stormingu: dyrektor (director) — top band
- aktor: maneger projektu (project manager) — bottom band
- value / legend area — left-centre green notes ("na czym zarabiamy/oszczedzamy")

## Flow

### Lane: aktor stormingu: dyrektor (director)
1. zaplanowano etapy projektu (długość, zależności między etami, ewentualne zasoby) (event) [fed from external system: Excel]
2. oszacowano ryzyko (risk)
3. zaparkowano projekt (event)
4. wystartowano projekt/etap (event)
5. monitoring postępu (razem z kosztami) (event) [fed from external system: monitorowanie czasu pracy / settlement time]
6. monitoring ryzyka (risk)
7. optymalizacja rozkładu zasobów (risk) [arrow up into #6 monitoring ryzyka]
8. symulacja stanu innych projektów gdyby zamknięto ten (risk)
9. dashboard czy się jeszcze opłaca? przekroczono budżet/zwiększon o ryzyko/etc (read model — green) [fed from monitorowanie czasu pracy / settlement time]
10. ZMIANA: inna funkcja porównująca opłacalność (np prestiż) (note — grey) [annotation attached to #9]
11. zamknięcie projektu (event)
12. zapłacenie kary umownej (event)
13. przeniesienie zasobów do innego projektu (event) [connected to external system: systemy do zarządzania zasobami (inventory, trainig, vacation)]
14. monitoring ryzyka (risk)
15. monitoring sumarycznych kosztów (event)
16. optymalizcja zasobów przez przenoszenie pomiędzy projektami (risk) [connected to external system: systemy do zarządzania zasobami]

[order unclear] Within the director band the settlement/monitoring cluster (#5–#10)
and the close-out cluster (#11–#16) are two loosely-ordered groups; exact sequence
between individual stickies is approximate.

### Lane: aktor: maneger projektu (project manager)
1. czy w ogóle są zasoby? (czy jest potencjał?) (risk)
2. zaplanowano etapy projektu (event)
3. zasymulował daty startu ewentualnie biorąc po uwagę dostępność zasobów (event — salmon) [fed from external system: Środowisko korporacyjne (dzieje się poza systemem)]
4. negocjacje z managarami (event)
5. przejrzał ryzyka swojego projektu (event)
6. wystartowano projekt/etap (event)
7. monitoring postępu (event)
8. wykryto opóźnienie (risk)
9. dodano zasób do kolejki projektu (event)
10. dodano zasoby (event)
11. wykryto przekroczenie budżetu (risk)
12. eskalacja (event)
13. cięcie zakresu (event)
14. wykryto braki (risk)
15. wyszukano zamiennik zasobu (event)

### Lane: aktor: maneger projektu (project manager) — planning sub-flow (below main band)
1. czy w ogóle są zasoby? (czy jest potencjał?) jeszcze nie warto planować etapów (risk) [arrow up into main-lane #2 zaplanowano etapy projektu]
2. zebrano wymagane zasoby (event)
3. przejrzał ryzyka swojego projektu (event)

### Lane: value / legend area (green notes)
1. na czym zarabiamy/oszczedzamy (note — green, value question)
2. np. na tym, że zapłaciłem karę umowną za zerwanie umowy i zatrzymanie projektu (note — green, value example)

## Pivotal events

- dyrektor #11 "zamknięcie projektu" — flows in from monitoring/dashboard decision
  (#6–#10, the "czy się jeszcze opłaca?" read model); flows out to #12 "zapłacenie
  kary umownej" and #13 "przeniesienie zasobów do innego projektu". This is the
  turning point where a project is judged no-longer-worth-continuing and is stopped.
- maneger projektu #12 "eskalacja" — flows in from #8 "wykryto opóźnienie" /
  #11 "wykryto przekroczenie budżetu"; flows out to #13 "cięcie zakresu". Turning
  point where in-project trouble is handed up / scope is cut.
- maneger projektu sub-flow #1 "czy w ogóle są zasoby? ... jeszcze nie warto
  planować etapów" — gates whether stage planning (main-lane #2) begins at all.

## Hot zones

- Director settlement/monitoring cluster — spans dyrektor #6–#10 (centre-top of
  board). Stickies: "monitoring ryzyka", "optymalizacja rozkładu zasobów",
  "symulacja stanu innych projektów gdyby zamknięto ten", "dashboard czy się
  jeszcze opłaca? ...", "ZMIANA: inna funkcja porównująca opłacalność (np prestiż)".
  Topic: is the project still worth continuing, and how would closing it change the
  rest of the portfolio.
- Project-manager trouble cluster — spans maneger projektu #8–#14 (bottom-right of
  board). Stickies: "wykryto opóźnienie", "wykryto przekroczenie budżetu",
  "eskalacja", "cięcie zakresu", "wykryto braki". Topic: detecting delay / budget
  overrun / resource gaps and reacting (escalate, cut scope, find replacement).
- Resource-availability gate — maneger projektu #1 and sub-flow #1 (bottom-left).
  Stickies: "czy w ogóle są zasoby? (czy jest potencjał?)", "... jeszcze nie warto
  planować etapów". Topic: whether resources/potential exist before planning stages.
- Value question — value area (left-centre). Stickies: "na czym
  zarabiamy/oszczedzamy", "np. na tym, że zapłaciłem karę umowną za zerwanie umowy
  i zatrzymanie projektu". Topic: where the business makes or saves money.

## Vocabulary

- Excel (external system)
- systemy do zarządzania zasobami (inventory, trainig, vacation) (external system)
- monitorowanie czasu pracy / settlement time (external system)
- Środowisko korporacyjne (dzieje się poza systemem) (external system)
- notatka, ryzyko, zdarzenie (legend terms)

## Layout observations

- Two horizontal actor bands: dyrektor (top), maneger projektu (bottom), each
  anchored by a white "aktor" sticky on the left.
- Four external systems drawn as rounded rectangles across the top and middle
  (Excel, resource-management systems, work-time monitoring/settlement,
  corporate environment), with thin connector lines down into event stickies —
  Excel → director stage planning; settlement/time → director monitoring; resource
  systems → director resource transfer/optimization; corporate environment →
  manager start-date simulation.
- Overall flow reads left-to-right and top-to-bottom, but this is a big-picture
  board: clusters are grouped by theme (planning, monitoring, close-out, trouble)
  rather than a single strict sequence. Curved connectors link the manager band
  up to the director monitoring cluster.
- Colour usage beyond the legend: green = value/opportunity notes and a dashboard
  read-model; salmon/red = one "simulate start dates" sticky; grey = a "ZMIANA"
  change-note annotation.
