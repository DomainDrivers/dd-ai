# Przegląd narzędzi AI (dd-ai)

Repo zawiera 54 artefakty, wszystkie w `NNN\.claude\` (po jednym folderze na odcinek kursu): 16 agentów, 12 komend i 26 skilli. Nie ma plików Copilota (`.github`, `*.prompt.md`, `AGENTS.md`). Narzędzia są przeznaczone dla **Claude Code**: używają `AskUserQuestion`, `Agent`/`Task` i `$ARGUMENTS`, więc wymagają adaptacji do Copilota. README ostrzega, że wyniki są niestabilne, a narzędzia nie nadają się do produkcji.

Aby użyć narzędzi z danego odcinka, skopiuj jego folder `.claude\` do katalogu głównego projektu. Foldery z odcinków 002, 004, 005, 006 i 007 można połączyć, bo nazwy ich plików się nie pokrywają.

## Sposób wywoływania
| Typ | Lokalizacja | Wywołanie |
|---|---|---|
| Komenda | `.claude\commands\X.md` | tylko użytkownik: `/X args` |
| Skill | `.claude\skills\X\SKILL.md` | model ładuje go automatycznie; `/X` też działa (żaden plik tego nie blokuje) |
| Agent | `.claude\agents\X.md` | wywoływany przez komendę lub innego agenta; użytkownik może też poprosić o niego z nazwy |

## Inwentarz
### 002 – Analiza wstępna: transkrypty → corpus → obszary
| Nazwa | Typ | Bezpośrednio? | Cel |
|---|---|---|---|
| `/modularize-preflight` | komenda | ✅ | Bramka: czy warto modularyzować? Zadaje 3 pytania (cykl życia, zakres, złożoność). Werdykt: kontynuacja albo odmowa z możliwością nadpisania. Zapisuje `pre-flight.md/json` |
| `/modularize-areas` | komenda | ✅ | Uruchamia `requirements-analyzer` |
| `requirements-analyzer` | agent | przez komendę lub po nazwie | Pipeline: ekstrakcja tekstu i zdjęć tablic ES → `corpus.md` → sprzeczności → 5 pytań strategicznych → grupowanie obszarów → `areas.md/json` |
| `sources-critic` | agent | przez analizator lub po nazwie | Znajduje sprzeczne wypowiedzi i zapisuje `contradictions\critique.md`. Nie rozstrzyga sprzeczności |
| `contradiction-finder` | skill | automatycznie | Określa, co jest sprzecznością. Działa też na surowych transkryptach |
| `evidence-corpus` | skill | automatycznie | Format bazy cytatów dosłownych ze źródłem i lokalizacją. Obszar wymaga ≥3 mocnych cytatów z ≥2 źródeł |
| `es-vision-extraction` | skill | automatycznie | Zamienia zdjęcia tablic Event Storming na `*.es.md` |
| `strategic-questions` | skill | automatycznie | 5 pytań biznesowych: przychody, ryzyka, **integracje**, regulacje, metryka. Listę pytań należy dostosować |
| `guided-conversation` | skill | automatycznie | Zasady tur pytań: 3–7 pytań na turę, maksymalnie ok. 10 tur |
| `pre-flight-gate` | skill | automatycznie | Logika komendy preflight |

### 003 – Symulator treningowy facylitatora Event Stormingu
- **Komendy (bezpośrednie ✅):** `/es-start`, `/es-round`, `/es-invite`, `/es-coach`, `/es-sync-miro`
- **Agenci:** 8 person klienta i `facilitator-coach`
- **Skille:** `event-storming-orchestration`, `event-storming-method`, `domain-knowledge`, `miro-card-spec`
- **Zastosowanie:** ćwiczenie prowadzenia warsztatu z symulowanymi personami.
- **Ograniczenie:** domena jest wpisana na sztywno. Dla innego klienta trzeba przygotować nowy `domain-brief.md` i nowe persony.

### 004 – `/algorithm-scan` ✅ (ze skillem `algorithm-catalog`, 42 algorytmy)
Odnajduje znane algorytmy za słownictwem biznesowym, np. szeregowanie zadań, przydział i sortowanie topologiczne. Wynik: `algorithms\scan.md`.

### 005 – `/connascence A B` ✅
- **Działanie:** ocenia sprzężenie dwóch modułów i szereguje najkosztowniejsze powiązania.
- **Kiedy użyć:** działa jeszcze przed powstaniem kodu, na podstawie mapy kontekstów lub opisu.
- **Elementy pomocnicze:** agent `connascence-scanner`; skille `connascence-evidence`, `connascence-taxonomy` i `connascence-scoring`.

### 006 – `/shake-the-model` ✅
- **Działanie:** sprawdza, jak dobrze model modułów przyjmuje zmiany. Dla podanej metryki biznesowej:
  - zapisuje brief
  - generuje wymagania „na ślepo”, bez dostępu do architektury
  - klasyfikuje wpływ każdego wymagania jako localized, distributed lub new-module
- **Elementy pomocnicze:** agent `blind-requirements-generator`; skille `business-brief`, `competitor-scan` i `change-impact`.

### 007 – skill `problem-classifier` ✅ (`/problem-classifier ...`)
- **Działanie:** przypisuje funkcję do jednej z 4 klas: CRUD, Transformation & Presentation, Integration lub Resource Contention. Wskazuje też, czego nie robić, i proponuje podział.
- **Wynik:** tylko w terminalu.

### 008 – `/restyle --target=<module>` ✅ (praca z kodem)
- **Działanie:** przekształca moduł z bogatego agregatu w styl „decider”, zachowując działanie dzięki testom.
- **Elementy pomocnicze:** agenci `idiom-analyzer`, `test-harness-architect` i `idiom-transformer`; 4 skille.

### 009 – `/sync-to-async` ✅ (praca z kodem)
- **Działanie:** zamienia synchroniczne wywołanie na komunikację asynchroniczną przez outbox, krokami opartymi na testach.
- **Elementy pomocnicze:** 4 skille dotyczące typu integracji, mechaniki outboxa, atomowości i testów komunikatów.

## Przebieg od początku do końca
```
transkrypty/notatki/zdjęcia ES
 → /modularize-preflight → /modularize-areas (corpus, sprzeczności, luki, obszary)
 → /algorithm-scan + /problem-classifier            (niezależne od siebie)
 → [RĘCZNIE] modules.md + context-map.md istniejącego systemu
 → /connascence, /shake-the-model                   (wpływ na istniejący system)
 → /sync-to-async, /restyle                         (implementacja)
```
Foldery `input`/`output` zawierają przykładowe przebiegi: 002 to pełna analiza wywiadu z prezesem, 004 korzysta z obszarów z 002, a 006 zawiera brief i raport. Żadne narzędzie w repo nie generuje `context-map.md` ani `modules.md`; trzeba je napisać ręcznie.

## Przykłady użycia (transkrypty klienta, nowa funkcja w istniejącym systemie)
1. **Nowy obszar czy rozszerzenie istniejącego modułu?**
   - Uruchom `/modularize-preflight .\discovery\transcripts .\discovery\out`.
   - Odpowiedź „Extends one module” daje `refuse-pre-scoped`. To użyteczny sygnał, ale werdykt można nadpisać.
2. **Pełna analiza transkryptów.**
   - Uruchom `/modularize-areas .\discovery\transcripts .\discovery\out`.
   - Wyniki: `corpus.md`, `critique.md` (pytanie do klienta dla każdego konfliktu), lista luk i `areas.md`.
   - Wskazówka: wcześniej zastąp pytania w `strategic-questions\SKILL.md` pytaniami o integrację: punkty styku z istniejącymi modułami, źródło prawdy, spójność i opóźnienia, migrację danych.
3. **Tylko sprzeczności między spotkaniami.**
   - Poproś: „Zastosuj skill contradiction-finder do wszystkich plików w `.\discovery\transcripts`; podaj konflikty ze źródłem, lokalizacją i pytaniem diagnostycznym”.
   - Wynik: lista pytań na kolejne spotkanie z klientem.
4. **Dobór skali projektu.**
   - Uruchom `/problem-classifier <opis funkcji: kto co robi, który moduł jest powiadamiany, użytkownicy równocześni…>`.
   - Wynik: klasa problemu, antywzorce do uniknięcia i otwarte pytania.
5. **Ukryte algorytmy.**
   - Uruchom `/algorithm-scan .\discovery\out\areas.md`.
   - Sprawdź, czy „przydział” lub „kolejność” to znany rozwiązany problem, czy rzeczywiście coś nowego.
6. **Wpływ na istniejący system.**
   - Najpierw napisz `modules.md` i `context-map.md`, wzorując się na `006\input\modules.md`.
   - Potem uruchom `/shake-the-model .\docs\context-map.md "<metryka klienta>" 8` i `/connascence new-feature billing`.
   - Jeśli integracja ma być asynchroniczna, użyj `/sync-to-async <miejsce wywołania>`.

---
# Aneks: raport szczegółowy
## Raport: artefakty AI w `dd-ai`

### 0. Najważniejsze ustalenia

- Repo zawiera **54 artefakty**. Wszystkie są w `NNN\.claude\` dla epizodów 002–009: 16 agentów, 12 komend, 26 skilli (plus jeden plik zasobu, `domain-brief.md`).
- Sprawdziłem całe drzewo z `-Force`, czyli łącznie z ukrytymi folderami. **Brak** `.github\`, `.copilot\`, `.vscode\`, `.agents\`, `AGENTS.md`, `CLAUDE.md`, `copilot-instructions.md`, `*.instructions.md`, `*.prompt.md` i `*.agent.md`. Nie ma więc żadnych plików instrukcji.
- Według README każdy numerowany folder to jeden odcinek kursu. Żeby użyć narzędzia, kopiujesz `.claude\` danego odcinka do roota projektu i uruchamiasz Claude Code. Epizody są niezależne, np. `evidence-corpus` istnieje tylko w 002.
- Narzędzia są pisane pod **Claude Code**: korzystają z `AskUserQuestion`, narzędzia `Agent`/`Task`, `mcp__miro__*` i `$ARGUMENTS`. W Copilot mogą wymagać adaptacji.
- W README jest zastrzeżenie: wyniki są niestabilne i narzędzia nie są do produkcji.

#### Jak się co uruchamia (Claude Code)

| Typ | Lokalizacja | Wywołanie |
|---|---|---|
| Komenda | `.claude\commands\X.md` | `/X args` – tylko użytkownik |
| Skill | `.claude\skills\X\SKILL.md` | Model ładuje go sam na podstawie `description`. Żaden plik nie ma `disable-model-invocation` ani `user-invocable: false`, więc według domyślnych zasad Claude Code można też wpisać `/X` |
| Agent | `.claude\agents\X.md` | Inny agent albo komenda uruchamia go narzędziem `Agent`. Użytkownik może poprosić o niego z nazwy |

Pola frontmatter `model:`, `user-invocable` i `disable-model-invocation` nie występują nigdzie. `allowed-tools` mają tylko dwie komendy z 002.

---

### 1. Inwentarz według epizodów

#### 002 – Intake: transkrypty → corpus → obszary (najważniejszy dla Ciebie)

| # | Ścieżka | Typ | Frontmatter |
|---|---|---|---|
| 1 | `002\.claude\commands\modularize-preflight.md` | komenda | `argument-hint: "[materials-path] [output-path]"`, `allowed-tools: Read, Write, Glob, Grep, AskUserQuestion` |
| 2 | `...\002\.claude\commands\modularize-areas.md` | komenda | `argument-hint: "[materials-path] [output-path]"`, `allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, Task, AskUserQuestion` |
| 3 | `...\002\.claude\agents\requirements-analyzer.md` | agent | `color: orange`, `tools: Read, Write, Edit, Glob, Grep, Bash, Agent, Task, AskUserQuestion` |
| 4 | `...\002\.claude\agents\sources-critic.md` | agent | `color: yellow`, `tools: Read, Write, Glob, Grep, Bash` |
| 5 | `...\002\.claude\skills\pre-flight-gate\SKILL.md` | skill | – |
| 6 | `...\002\.claude\skills\evidence-corpus\SKILL.md` | skill | – |
| 7 | `...\002\.claude\skills\es-vision-extraction\SKILL.md` | skill | – |
| 8 | `...\002\.claude\skills\strategic-questions\SKILL.md` | skill | – |
| 9 | `...\002\.claude\skills\guided-conversation\SKILL.md` | skill | – |
| 10 | `...\002\.claude\skills\contradiction-finder\SKILL.md` | skill | – |

**`/modularize-preflight`** to tylko launcher skilla `pre-flight-gate`.
- **Cel:** zdecydować, czy domenę w ogóle warto modularyzować.
- **Wejście:** folder materiałów i opcjonalny folder wyjścia.
- **Wyjście:** `pre-flight.md` i `pre-flight.json`.
- **Wywołanie:** bezpośrednio przez `/modularize-preflight`.

**`pre-flight-gate`** (skill):
1. Skanuje materiały pod kątem sygnałów: PoC, z góry jeden moduł, brak złożoności. Sygnały są tylko podpowiedzią.
2. Zadaje 3 pytania w jednym `AskUserQuestion`: Lifecycle, Scope, Complexity.
3. Liczy werdykt `proceed` / `refuse-throwaway` / `refuse-pre-scoped` / `refuse-no-complexity`.
4. Otwiera obowiązkową bramkę: Confirm / Override / Adjust. Nic nie zapisuje przed akceptacją.

Wywołuje go `/modularize-preflight`. `requirements-analyzer` czyta opcjonalnie wynikowy `pre-flight.json` i zatrzymuje się przy odmowie.

> ⚠️ Dla Twojej sytuacji: odpowiedź Q2 = „Extends one module” daje `refuse-pre-scoped` z sugestią pracy taktycznej w istniejącym module. Bramka pozwala na **Override**.

**`/modularize-areas`** to launcher agenta `requirements-analyzer`. Agent musi działać **inline**, nie jako sub-agent, bo zadaje pytania przez `AskUserQuestion` i sam uruchamia `sources-critic`.

**`requirements-analyzer`** (agent) – główny pipeline intake:
- **Wejście:** folder materiałów: md/txt/pdf/docx/xlsx/pptx, obrazy ES (png/jpg), eksporty ES w json. Opcjonalnie `pre-flight.json`.
- **Wyjście:** `corpus.md`, `<obraz>.es.md`, `contradictions\critique.md` + `summary.json`, `user-answers.md`, `potential-areas.md`, `areas.md` + `areas.json`.
- **Kroki:**
  1. Sanity check.
  2. Inwentarz typów plików.
  3. Ekstrakcja tekstu. Powyżej ok. 150 KB jest bramka rozmiaru.
  4. Ekstrakcja obrazów ES wizją, z bramką review.
  5. Budowa corpusu.
  6. Uruchomienie `sources-critic` (sprzeczności).
  7. Skan 5 pytań strategicznych.
  8. Bramka strategiczna: pytania plus sprzeczności. Odpowiedzi trafiają do corpusu, a zastąpione cytaty dostają `[superseded by C<n>]`.
  9. Klastrowanie obszarów. Próg: co najmniej 3 mocne cytaty z co najmniej 2 źródeł. Nazwy dosłownie z corpusu.
  10. Bramka obszarów: confirm / rename / merge / split / drop / add.
  11. Zamrożenie obszarów (freeze).
- **Używa skilli:** `strategic-questions`, `evidence-corpus`, `es-vision-extraction`, `guided-conversation`. Sub-agent: `sources-critic`.
- **Wywołanie:** przez `/modularize-areas` albo przez użytkownika („follow requirements-analyzer”).

**`sources-critic`** (agent):
- **Cel:** znaleźć wypowiedzi, które nie mogą być jednocześnie prawdziwe. Nigdy ich nie rozstrzyga.
- **Wejście:** `corpus.md` i katalog wyjścia. Może zajrzeć ok. ±3 linie wokół cytatu w źródle.
- **Wyjście:** `contradictions\critique.md` (tabela # · Conflict · Statements · Why · Diagnostic question) oraz `summary.json` z hashem `corpus_body_sha256` jako cache. Jeśli nie ma twierdzeń do sprawdzenia: `skipped:true`.
- **Używa:** `contradiction-finder`.
- **Wywołuje go:** `requirements-analyzer` w kroku 6. Można też poprosić o niego ręcznie.

**`contradiction-finder`** (skill):
- Definiuje, co jest sprzecznością: ta sama reguła dwa razy różnie, rozbieżne liczby, decyzja łamiąca ograniczenie, wymagania wzajemnie wykluczające się, cicha zmiana zdania.
- Definiuje, co nią nie jest: różne tematy, jawna zmiana decyzji, różnice preferencji, ogólnik obok precyzji.
- Wyjście to kontrakt JSON. Każdy konflikt ma pytanie diagnostyczne.
- Jest niezależny od projektu i można go użyć wprost na surowych transkryptach.

**`evidence-corpus`** (skill):
- Format bazy cytatów: sekcja per źródło, `Type`, `Language`, cytaty dosłownie w oryginalnym języku.
- Formaty `loc`: `L42`, `§`, `region:`, `p12`, `Q<id> inline answer`.
- Zasady cytowania: koncept wymaga 1 cytatu, obszar co najmniej 3 mocnych z co najmniej 2 źródeł, pytanie „answered” pełnej checklisty.
- Definicje: cytat mocny vs. wzmianka, znacznik superseded.
- Corpus ma być **wyczerpujący**, nie reprezentatywny.

**`es-vision-extraction`** (skill):
- Dosłowny prompt ekstrakcji tablicy ES jako przepływu per lane. Sekcje: Legend, Lanes, Flow, Pivotal events, Hot zones, Vocabulary, Layout.
- Cache `<basename>.es.md` z `source_hash`, wersja `es-vision-extraction-v2`.
- Bramka review: liczniki i 3 losowe karteczki. Jeśli ponad ok. 10% jest nieczytelne, prosi o lepszy obraz.

**`strategic-questions`** (skill):
- Wymienialny blok YAML z 5 pytaniami „show-me-the-money”: q1 przychody/oszczędności, q2 ryzyka/szanse, q3 integracje, q4 regulacje, q5 metryka optymalizacji.
- Statusy: answered / partial / missing / deferred / n_a.
- Pole `gap_description` ma formę checklisty.
- Plik mówi wprost: podmień `questions:` na swój zestaw pytań.

**`guided-conversation`** (skill):
- Protokół bramki inline: analizuj → pytaj → czekaj → przetwórz → sprawdź ponownie.
- 3–7 pytań na turę, każde zakotwiczone w dowodach, z wartością domyślną.
- Limit ok. 10 tur. Tokeny `skip`, `stop`, `redo`.

#### 003 – Symulator treningowy Event Stormingu (persony SmartSchedule, PL)

| Ścieżka (`003\.claude\...`) | Typ | Frontmatter |
|---|---|---|
| `commands\es-start.md` | komenda | `argument-hint: "[--light] [--no-miro] [--auto-sync]"` |
| `commands\es-round.md` | komenda | `argument-hint: "<co facylitator mówi/robi>"` |
| `commands\es-invite.md` | komenda | `argument-hint: "<persona-name> [pytanie/temat]"` |
| `commands\es-coach.md` | komenda | `argument-hint: "[--full] [pytanie do coacha]"` |
| `commands\es-sync-miro.md` | komenda | `argument-hint: "[--reflow]"` |
| `agents\prezes-japinski.md`, `ewa-pm.md`, `marek-brygadzista.md`, `kasia-junior-dev.md`, `tomek-architekt.md`, `agnieszka-ksiegowa.md`, `piotr-mecenas.md`, `gerome-don-kichot.md` | 8 agentów-person | `tools: Read`. Każda ma „Twój dzień” (8 wariantów), anegdoty, wyuczony anty-wzorzec i kontrakt YAML `stickies` + `dialogue` |
| `agents\facilitator-coach.md` | agent | `tools: Read, Grep, Glob`. Checklista 10 anty-wzorców, raport debrief |
| `skills\event-storming-orchestration\SKILL.md` | skill | Kontrakt orkiestratora: nigdy nie gra persony; `state.json`; seed/today; pamięć; echo sali; tylko `event`/`hot_spot`; reguły Miro |
| `skills\event-storming-method\SKILL.md` | skill | Metodyka Brandoliniego, ładowana przez coacha |
| `skills\domain-knowledge\SKILL.md` + **zasób** `domain-brief.md` | skill | Filtr wiedzy per persona i brief domeny SmartSchedule |
| `skills\miro-card-spec\SKILL.md` | skill | Mapowanie karteczek na SVG Miro (MCP `https://mcp.miro.com`) |

- **Cel:** trening facylitatora. Człowiek prowadzi warsztat, 8 person-subagentów odpowiada równolegle, coach ocenia na żądanie.
- **Wyjście:** `./sessions/<id>/state.json`, `transcript.md`, `board.md`, `debrief.md`, `sync.log`. Opcjonalnie board Miro.
- **Kolejność:** `/es-start` → (`/es-round` | `/es-invite`)* → `/es-sync-miro`? → `/es-coach [--full]`.
- **Zależności:** każda komenda najpierw ładuje `event-storming-orchestration`. `/es-round` i `/es-invite` uruchamiają persony. `/es-coach` uruchamia `facilitator-coach`, który ładuje `event-storming-method`. `/es-sync-miro` używa `miro-card-spec`.
- **Ograniczenie:** domena jest wpisana na sztywno (firma budowlana Japińskiego). Dla Twojego klienta trzeba by przepisać `domain-brief.md` i persony.
- Nie zasila pipeline'u danych. Folder 003 nie ma `input` ani `output`.

#### 004 – Skan algorytmów

| Ścieżka | Typ | Frontmatter |
|---|---|---|
| `004\.claude\commands\algorithm-scan.md` | komenda | `argument-hint: "[domain requirements, feature description, or path to a candidate-areas file]"` |
| `...\004\.claude\skills\algorithm-catalog\SKILL.md` | skill (53 KB) | 42 algorytmy w rodzinach Graph / Tree / Optimization / Search / Time. Każdy ma fit question, bridge words, mapowanie domena→algorytm, „Not this if” z przekierowaniem i odnośnik do Skieny |

**`/algorithm-scan`**:
- **Cel:** znaleźć kanoniczne algorytmy ukryte za słownictwem biznesu, np. topological sort albo assignment. Działa jak fail-fast: albo znany problem, albo naprawdę nowy teren.
- **Dwa mechanizmy, łączone sumą:** dopasowanie po słowach (bridge-word) i rozpoznanie strukturalne (9 pytań o kształt problemu plus fit question każdego wpisu katalogu).
- **Wejście:** tekst, `areas.md` (skan per obszar) i/lub źródła/corpus.
- **Wyjście:** `algorithms\scan.md`. Per obszar najpierw YAML, potem tabele. Pola: `top_pick`, `near_misses`, `honest_misses`, obserwacje międzyobszarowe (consolidation / seam / shared_kernel).
- Cytaty weryfikowane w źródle.
- **Wywołanie:** bezpośrednio przez `/algorithm-scan`. Używa `algorithm-catalog`.

#### 005 – Connascence (sprzężenie dwóch modułów)

| Ścieżka (`...\005\.claude\...`) | Typ | Frontmatter |
|---|---|---|
| `commands\connascence.md` | komenda | `argument-hint: "[two modules/contexts to compare — names, paths, a context-map edge, or a sentence...]"` |
| `agents\connascence-scanner.md` | agent | `tools: Read, Grep, Glob`, `color: cyan`. Read-only: stały przepis wyszukiwania, zwraca surowe miejsca `path:line` z wyszukiwaniami do powtórzenia |
| `skills\connascence-evidence\SKILL.md` | skill | Kontrakt `kind:locator` (`code:`, `map:`, `contract:`, `doc:`, `search:`, `assumption:`) |
| `skills\connascence-taxonomy\SKILL.md` | skill | 9 rodzajów (N, T, M, P, A, E, TM, V, I), reguły R1–R9 „jedno porozumienie = jeden wiersz”, boundary rule |
| `skills\connascence-scoring\SKILL.md` | skill | Reguły S1–S6, D1–D9, `cost = strength × log2(1+degree)`, profil, ranking, ledger |

**`/connascence`**:
- **Kroki:** 0 = rama i inwentarz dowodów, 1 = szew (seam) wiersz po wierszu, 2 = koszt, profil i ledger. Każdy krok kończy się bramką.
- **Wejście:** może działać na samej mapie kontekstów, kontrakcie albo opisie słownym, czyli jeszcze przed kodem. Wtedy daje zakresy.
- **Wyjście:** `connascence-<a>-<b>.frame.md` i `connascence-<a>-<b>.md` (po akceptacji), raport w terminalu.
- **Zależności:** ładuje 3 skille i uruchamia `connascence-scanner`, gdy kod jest nietrywialny.

#### 006 – Shake-the-model (test rozszerzalności modelu)

| Ścieżka (`...\006\.claude\...`) | Typ | Frontmatter |
|---|---|---|
| `commands\shake-the-model.md` | komenda | `argument-hint: "[domain / architecture material ... and a business metric; optionally competitors, count]"` |
| `agents\blind-requirements-generator.md` | agent | `tools: Read`, `color: purple`. Odcięty od architektury (firewall): czyta tylko brief i digest. Zwraca R1..Rn: Ask / Mechanism / Grounding / źródło |
| `skills\business-brief\SKILL.md` | skill | Destylacja brief bez rozwiązań: lista rzeczy do usunięcia, mirror test, invariance test |
| `skills\competitor-scan\SKILL.md` | skill | Digest konkurencji C1..Cn ze źródłem (`web:`/`doc:`/`user:`). Web tylko za zgodą |
| `skills\change-impact\SKILL.md` | skill | Build check (uses/extends/gains/no home), klasyfikacja localized / distributed / new-module, neutralne podsumowanie |

**`/shake-the-model`**:
1. Metryka (podana, nigdy wymyślona), `shake-the-model-brief.md`, opcjonalnie `shake-the-model-competitors.md`. Bramka.
2. `blind-requirements-generator` generuje wymagania, domyślnie 8 na metrykę. Bramka.
3. Wpływ na moduły i automatycznie `shake-the-model-report.md`.

Próbka w `006\output`: R1–R5, wynik localized 1 / distributed 3 / new-module 1.

#### 007 – Klasyfikator klasy problemu

| Ścieżka | Typ | Frontmatter |
|---|---|---|
| `007\.claude\skills\problem-classifier\SKILL.md` | skill (34 KB) | `argument-hint: "[business requirements or feature description]"`. W `description` są triggery: „jaka klasa problemu”, „problem class” |

- **Cel:** przypisać wymaganie do jednej z 4 klas: CRUD, Transformation & Presentation, Integration, Resource Contention.
- **Kroki:** 0 = wejście, 1 = cichy skan sygnałów (tekst i makiety UI), 2 = do 4 pytań przez `AskUserQuestion` w języku użytkownika, 3 = klasyfikacja, 4 = wynik.
- **Wyjście:** dedukcja, „Why this class”, „What NOT to do”, sugerowane podejście, otwarte pytania, dekompozycja, diagram komponentów.
- Raport tylko w terminalu.
- **Wywołanie:** jedyny skill z `argument-hint`, czyli zaprojektowany do `/problem-classifier ...`. Model może go też uruchomić sam.

#### 008 – Restyle (rich aggregate → decider)

| Ścieżka (`...\008\.claude\...`) | Typ | Frontmatter |
|---|---|---|
| `commands\restyle.md` | komenda | `argument-hint: "--target=<module-path>"` |
| `agents\idiom-analyzer.md` | agent | `tools: Read, Grep, Glob`. Krok 0b, zapisuje `idiom-card.json` |
| `agents\test-harness-architect.md` | agent | `tools: Read, Write, Edit, Glob, Grep, Bash`. Kroki 1a i 1b: DSL, Protocol Driver, `contract.md` |
| `agents\idiom-transformer.md` | agent | `tools: Read, Write, Edit, Glob, Grep, Bash`. Kroki 2 i 3: swap na decider, lower-bind, usunięcie starego idiomu |
| `skills\four-layer-test-architecture`, `idiom-catalog`, `idiom-transformation-recipes`, `behavior-preservation-contract` | 4 skille | Model testów, idiomy, przepis, inwarianty |

- **Wyjście:** zmiany w kodzie z commitem per krok oraz `.restyle\<id>\{idiom-card.json, contract.md, state.json}`.
- Dotyczy etapu implementacji, nie analizy.

#### 009 – Sync → async przez outbox

| Ścieżka (`...\009\.claude\...`) | Typ | Frontmatter |
|---|---|---|
| `commands\sync-to-async.md` | komenda | `argument-hint: "[a public API method or call site ...]"` |
| `skills\integration-classification`, `outbox-mechanics`, `transactional-atomicity`, `message-contracts-and-tests` | 4 skille | Event vs command i dyskwalifikatory; menu outboxa; atomowość; testy producenta, kontraktu i konsumenta |

- **Kroki:** 0a–0d to projektowanie. 1a, 1b, 2, 3, 4a, 4b to kod w rytmie RED/GREEN, z commitem per krok.
- Przydatne, gdy nowa funkcja ma się **integrować asynchronicznie** z istniejącym modułem.

---

### 2. Foldery numerowane: co jest wejściem i wyjściem czego

| Folder | `input\` | `output\` | Artefakt, który to wyprodukował / skonsumował |
|---|---|---|---|
| 002 | `wywiad-z-prezesem.md` (transkrypt), `big picture.png`, `process level.png` | `*.es.md`, `corpus.md` (57 cytatów), `contradictions\critique.md` (3 konflikty: 2 high, 1 medium), `user-answers.md` (rozstrzygnięcia C1…), `potential-areas.md`, `areas.md` (6 obszarów, np. „fazy projektu”, „kary umowne za opóźnienia”) | `/modularize-areas` → `requirements-analyzer` + `sources-critic` |
| 003 | – | – | Symulator, wynik w `sessions\` |
| 004 | `areas.md` i `corpus.md` (kopie z 002) | `algorithms\scan.md` (top picks: job-scheduling, transitive-closure, linear-programming, assignment, maxsat) | `/algorithm-scan` |
| 005 | – (kod dd-java) | – (terminal) | `/connascence` |
| 006 | `context-map.md`, `modules.md` (fasady modułów dd-java), `metryki.md` | `shake-the-model-brief.md`, `shake-the-model-report.md` | `/shake-the-model` |
| 007 | `areas.md` (7 obszarów) i `corpus.md` z wcześniejszego przebiegu intake | – (terminal) | `problem-classifier` |
| 008, 009 | – (kod dd-java, branche) | – (commity) | `/restyle`, `/sync-to-async` |

Uwagi:
- Corpus w 007 wspomina tagi pewności z `transcript-critique/`. To wcześniejszy wariant krytyki, którego nie ma w repo.
- `context-map.md` i `modules.md` z 006 **nie są generowane przez żadne narzędzie** w tym repo. Krok „obszary → moduły/mapa kontekstów” jest ręczny.

---

### 3. Odtworzony workflow end-to-end

```
transkrypty / notatki / zdjęcia ES
  │
  ├─(1) /modularize-preflight  → pre-flight.{md,json}           [pre-flight-gate]
  │
  ├─(2) /modularize-areas      → requirements-analyzer (inline)
  │      ├ es-vision-extraction → *.es.md (bramka review)
  │      ├ evidence-corpus      → corpus.md
  │      ├ sources-critic(contradiction-finder) → contradictions/
  │      ├ strategic-questions + guided-conversation → bramka "show-me-the-money" + sprzeczności → user-answers.md
  │      └ klastrowanie → bramka obszarów → areas.{md,json}, potential-areas.md
  │
  ├─(opc.) 003 /es-start…/es-coach — trening prowadzenia warsztatu ES z ludźmi klienta
  │
  ├─(3) /algorithm-scan areas.md → algorithms/scan.md
  ├─(4) /problem-classifier <obszar/feature> → klasa + podejście (terminal)
  │
  ├─(5) [RĘCZNIE] modules.md + context-map.md (projekt / istniejący kod)
  │
  ├─(6) /connascence A B → ocena sprzężenia granicy (działa już na mapie)
  ├─(7) /shake-the-model context-map + metryka → brief, wymagania "na ślepo", raport wpływu
  │
  └─(8) implementacja: /sync-to-async (integracja async przez outbox), /restyle (idiom modułu)
```

Kroki 3–4 są od siebie niezależne. Kroki 6–7 wymagają inwentarza modułów.

---

### 4. Przykłady użycia dla Twojej sytuacji (transkrypty klienta, nowa funkcja w istniejącym systemie)

Przygotowanie: skopiuj `.claude\` z wybranych epizodów do roota projektu. Możesz scalić `002`, `004`, `006`, `007` i `005`, bo nazwy plików się nie powtarzają.

#### UC1 – Bramka: czy to w ogóle „nowy obszar”, czy taktyka w istniejącym module?
- **Cel:** jawna decyzja przed analizą.
- **Wywołanie:** `/modularize-preflight .\discovery\transcripts .\discovery\analysis`
- **Wejście:** transkrypty. Odpowiedzi na 3 pytania. Jeśli funkcja musi rozszerzyć istniejący moduł, wybierz „Extends one module” albo „Other” z wyjaśnieniem.
- **Wyjście:** `pre-flight.md` i `.json`. Werdykt `refuse-pre-scoped` to też wartościowa informacja; można go nadpisać przez Override.

#### UC2 – Pełny intake transkryptów: corpus, sprzeczności, luki, obszary
- **Cel:** jedna baza dowodów i lista pytań do klienta.
- **Wywołanie:** `/modularize-areas .\discovery\transcripts .\discovery\analysis`
- **Wejście:** transkrypty (md/txt/docx/pdf), zdjęcia tablic, notatki.
- **Wyjście:**
  - `corpus.md`
  - `contradictions\critique.md`: każdy konflikt z pytaniem do klienta
  - tabela 5 pytań strategicznych z checklistą braków; q3 „Integracje” jest kluczowe przy wpinaniu w istniejący system
  - `user-answers.md`, `areas.{md,json}`, `potential-areas.md`
- **Wskazówka:** przed uruchomieniem podmień blok `questions:` w `strategic-questions\SKILL.md` na pytania pod integrację, np.: punkty styku z istniejącymi modułami, kto jest źródłem prawdy, wymagania spójności i opóźnienia, migracja danych.

#### UC3 – Same sprzeczności między kolejnymi rozmowami (szybki przebieg)
- **Cel:** wyłapać, że klient w rozmowie 1 mówił X, a w rozmowie 3 nie-X.
- **Wywołanie:** „Użyj agenta sources-critic na `.\discovery\analysis\corpus.md`, zapisz wynik w `.\discovery\analysis\contradictions`”. Bez corpusu: „Zastosuj skill contradiction-finder do wszystkich plików w `.\discovery\transcripts` i wypisz konflikty ze źródłem, lokalizacją i pytaniem diagnostycznym dla każdego”.
- **Wyjście:** `critique.md` + `summary.json`, czyli gotowa lista pytań na następne spotkanie. Narzędzie niczego nie rozstrzyga.

#### UC4 – Klasa problemu nowej funkcji (CRUD / T&P / Integration / Resource Contention)
- **Cel:** dobrać właściwy poziom „ciężkości” modelu i uniknąć over- lub under-engineeringu.
- **Wywołanie:** `/problem-classifier Nowa funkcja <opis z areas.md>: użytkownik X robi Y, system musi powiadomić moduł Z, a kilku dyspozytorów może jednocześnie...`
- **Wejście:** opis albo obszar z `areas.md`. Pomaga dołączenie makiety UI. Narzędzie dopyta o współbieżność i integracje.
- **Wyjście:** klasa z dedukcją, czego nie robić, sugerowana dekompozycja (np. CRUD + RC), otwarte pytania. Wynik w terminalu.

#### UC5 – Ukryte algorytmy w wymaganiach
- **Cel:** sprawdzić, czy „przydział”, „kolejność” albo „harmonogram” to znany problem.
- **Wywołanie:** `/algorithm-scan .\discovery\analysis\areas.md`
- **Wejście:** `areas.md` i `corpus.md`.
- **Wyjście:** `algorithms\scan.md`: kandydaci per obszar z mapowaniem, pewnością, near-missami i honest misses (realnie nowe tereny).

#### UC6 – Jak nowa funkcja wpłynie na istniejący system (wpływ + sprzężenie)
- **Cel:** ocenić, ile modułów istniejącego systemu ruszą przyszłe wymagania i jak mocno nowa funkcja zwiąże się z modułem docelowym.
- **Wywołania:**
  - `/shake-the-model .\docs\context-map.md .\docs\modules.md "<metryka biznesowa klienta, np. czas obsługi zlecenia ↓>" 8`
  - `/connascence new-feature and billing` (może bazować na opisie albo mapie, zanim powstanie kod)
- **Wejście:** mapa kontekstów lub moduły z publicznymi API istniejącego systemu (do napisania ręcznie, wzór w `006\input\modules.md`) i metryka z rozmów.
- **Wyjście:**
  - `shake-the-model-brief.md`, opcjonalnie `-competitors.md`, `shake-the-model-report.md` (localized / distributed / new-module per wymaganie, z source-refami)
  - `connascence-<a>-<b>.md`: tabela porozumień na granicy, koszty, ranking najdroższych sprzężeń
- **Dalej:** jeśli integracja ma być asynchroniczna, `/sync-to-async <call site>` z 009.