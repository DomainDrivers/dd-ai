---
description: Rozpocznij nową sesję treningową Event Stormingu — tryb light/full, brief scenariusza, przedstawienie person, opcjonalny board Miro, id sesji.
argument-hint: "[--light] [--no-miro] [--auto-sync]"
---

# /es-start

> **Zanim zaczniesz**: wczytaj kontrakt orkiestratora — `.claude/skills/event-storming-orchestration/SKILL.md` — i trzymaj się go (jesteś skrybą/orkiestratorem, nigdy nie wcielasz się w personę).

Rozpocznij nową sesję treningową Event Stormingu.

## Co robisz

**Najpierw ustal tryb** z `$ARGUMENTS`:
- `--light` → **tryb light**: persony tylko się wypowiadają — bez kartek, bez Miro. Szybki trening samej rozmowy facylitacyjnej.
- brak `--light` → **tryb full**: persony naklejają kartki (`event` / `hot_spot`); Miro opcjonalne (patrz `--no-miro`).

1. **Wygeneruj id sesji** w formacie `YYYY-MM-DD-HHMM` (np. `2026-05-17-1842`).
2. **Utwórz folder** `./sessions/<id>/` z trzema plikami:
   - `state.json` — zainicjowany jak niżej
   - `transcript.md` — pusty nagłówek
   - `board.md` — pusty (używany, gdy Miro niedostępne)
3. **Początkowy state.json** — zależnie od trybu:

   **full:**
   ```json
   {
     "id": "<id>",
     "mode": "full",
     "level": "big_picture",
     "started_at": "<ISO-8601>",
     "round": 0,
     "seed": <suma cyfr z id>,
     "today": { "<persona>": <numer wariantu>, "...": 0 },
     "memory": {},
     "stickies": [],
     "hot_spots": [],
     "miro": {
       "board_id": null,
       "enabled": false,
       "auto_sync": false,
       "url": null,
       "frames": { "big_picture": null }
     }
   }
   ```

   **light** (bez kartek i Miro):
   ```json
   {
     "id": "<id>",
     "mode": "light",
     "level": "big_picture",
     "started_at": "<ISO-8601>",
     "round": 0,
     "seed": <suma cyfr z id>,
     "today": { "<persona>": <numer wariantu>, "...": 0 },
     "memory": {}
   }
   ```

   **`seed` i `today` — losowanie dnia person** (oba tryby, patrz „Wariancja
   sesji" w kontrakcie orkiestratora). Policz raz, przy starcie:

   ```
   seed    = suma cyfr z id sesji     (2026-08-10-1655 → 36)
   wariant = ((seed + pozycja_persony - 1) mod 8) + 1
   ```

   Pozycje: prezes-japinski 1, ewa-pm 2, marek-brygadzista 3, kasia-junior-dev 4,
   tomek-architekt 5, agnieszka-ksiegowa 6, piotr-mecenas 7, gerome-don-kichot 8.
   Wpisz osiem par `"<slug>": <wariant>` do `today` i **nie ruszaj ich** do końca
   sesji. Facylitatorowi nie pokazujesz tych numerów — ma sam wyczuć, z czym kto
   dziś przyszedł.
4. **Miro** — tylko w trybie **full** i bez `--no-miro`:
   - **tryb light albo `--no-miro`**: pomiń całe sprawdzanie Miro. W full zostaw `miro.enabled = false` i `miro.url = null` (stickies → `./sessions/<id>/board.md`, `/es-sync-miro` no-op). W light Miro w ogóle nie dotyczy. Przejdź dalej.
   - **full bez flagi — sprawdź dostępność Miro**: spróbuj wywołać `mcp__miro__board_search_boards(limit=20, owned_by_me=true)`. Jeśli działa:
     - Pokaż listę boardów do wyboru (np. via `AskUserQuestion`) lub zaproponuj utworzenie nowego przez `mcp__miro__board_create(name="Event Storming - sesja <id>")`.
     - Po wyborze: wyciągnij board ID z `miro_url` (segment po `/board/`, włącznie z trailing `=`). Zapisz `miro.board_id`, `miro.url`, `miro.enabled = true` oraz `miro.auto_sync = true`, jeśli w `$ARGUMENTS` było `--auto-sync`.
     - Jeśli Miro nie zadziała (serwer niepodłączony, brak OAuth albo błąd wywołania) lub user mówi nie — wyjaśnij, że wszystkie stickies trafią do `./sessions/<id>/board.md`, a `/es-sync-miro` będzie no-opem. Jeśli brakuje OAuth, wskaż userowi `/mcp` → `miro` do uwierzytelnienia. Jeśli serwera w ogóle nie ma na liście, podaj linię rejestracji: `claude mcp add --transport http --scope project miro https://mcp.miro.com`.
5. **Briefing** (wyświetl userowi):

   ```
   ═══════════════════════════════════════
   Sesja Event Storming — <id>
   Poziom: BIG PICTURE  ·  Tryb: <FULL: persony naklejają kartki | LIGHT: persony tylko mówią>
   ═══════════════════════════════════════

   Kontekst dla Ciebie (facylitator):
   • Firma budowlana Prezesa Janusza Japińskiego
   • Cel: zrozumieć domenę, żeby zaprojektować system "smart-schedule"
   • Czas: typowo 60-90 minut realnej sesji
   • Twoja rola: facylitować, NIE proponować rozwiązań
   • Sala odpowiada po polsku, także gdy Ty piszesz w innym języku

   Uczestnicy na sali (gotowi do reakcji):
   ┌──────────────────────────────────────────────────────────────────┐
   │ Prezes Japiński   — właściciel; strategiczny, w biegu            │
   │ Ewa Nowak         — PM 7 lat; operacyjna, uporządkowana          │
   │ Marek Wiśniewski  — brygadzista 25 lat; konkretny, sceptyczny    │
   │ Kasia Lewandowska — junior dev 8 mies; cicha, ale trafna         │
   │ Tomek Kowalski    — senior architekt; świeżo po DDD              │
   │ Agnieszka Wójcik  — księgowa; liczby, kary, leasing              │
   │ P. Mecenas Zawisza — radca prawny; compliance, BHP               │
   │ Gerome «Don Kichot» — analityk z zewnątrz; "wszystko to dokumenty"│
   └──────────────────────────────────────────────────────────────────┘

   Komendy podczas sesji:
   • /es-round       — Twoje wejście jako facylitator → reakcja sali
   • /es-invite X    — zaproś konkretną personę imiennie
   • /es-sync-miro   — wypchnij stickies na Miro   (tylko tryb full; w light pomiń)
   •   ...--reflow   — rozłóż timeline równo, gdy ściana się zagęściła
   • /es-coach       — krótki feedback coacha (na żądanie)

   Powodzenia. Pierwsze wejście robisz Ty — w formacie /es-round.
   ```

6. **Ramka tworzona leniwie** (tylko tryb full z Miro) — nie tworzymy ramki w `/es-start`. Powstanie ona przy pierwszym `/es-sync-miro` dla danego poziomu (jeden dokument SVG niesie ramkę razem z kartkami). Zobacz `.claude/skills/miro-card-spec/SKILL.md`.

## Argumenty

- **`--light`** — tryb light: persony tylko się wypowiadają, bez kartek i bez Miro. Trening samego prowadzenia sali — rozdzielania głosu, wyciągania cichych, pilnowania języka; coach ocenia w nim trzy punkty checklisty (2, 3, 10). Bez tej flagi działa tryb **full** (persony naklejają kartki).
- **`--no-miro`** — (dotyczy trybu full) pomiń Miro całkowicie. `miro.enabled` zostaje `false`, stickies lecą do `./sessions/<id>/board.md`. Szybciej — brak round-tripów do Miro po rundzie. W trybie light Miro i tak nie działa.
- **`--auto-sync`** — (dotyczy trybu full z Miro) ustaw `miro.auto_sync = true`: każda runda kończy się pushem na board. Wygodne, gdy ktoś patrzy na board na żywo w trakcie sesji. Domyślnie kartki czekają w stanie, a facylitator wypycha je sam przez `/es-sync-miro` — jeden push zamiast jednego na rundę.

## Idempotencja

Sesja jest **aktywna**, dopóki jej `state.json` nie ma pola `ended_at`. To pole
jest jedynym znacznikiem zamknięcia — bez niego każdy kolejny `/es-start`
traktowałby poprzednią sesję jako trwającą.

Jeśli istnieje już aktywna sesja, zapytaj:
- **Kontynuować ostatnią?** — wczytaj stan, pokaż streszczenie
- **Nowa sesja?** — dopisz staremu stanowi `"ended_at": "<ISO-8601>"` oraz
  `"ended_as": "finished"`, dopiero potem utwórz nową z nowym id
- **Zamknij starą jako porzuconą** — dopisz `"ended_at": "<ISO-8601>"` oraz
  `"ended_as": "abandoned"` i twórz nową

## Nie rób

- Nie inicjuj rundy. Czekaj na `/es-round`.
- Nie wymyślaj persony spoza listy.
- Nie roleplayuj żadnej persony w wiadomości startowej.
