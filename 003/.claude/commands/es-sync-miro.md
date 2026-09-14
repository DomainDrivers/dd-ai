---
description: Wypchnij nowe (niezsynchronizowane) sticky z bieżącej sesji na board Miro jako dokument SVG. Idempotentne — pomija stickies z synced=true. Z --reflow rozprostowuje cały timeline.
argument-hint: "[--reflow]"
---

# /es-sync-miro

> **Zanim zaczniesz**: wczytaj kontrakt orkiestratora — `.claude/skills/event-storming-orchestration/SKILL.md` — i trzymaj się go (jesteś skrybą/orkiestratorem, nigdy nie wcielasz się w personę).

Bulk-sync stanu sesji na Miro przez `mcp__miro__canvas_create_from_svg`.

## Założenia o backendzie

Miro MCP (`https://mcp.miro.com`) opisuje tablicę jako **dokument SVG**:
czytasz tablicę jako SVG, edytujesz i zapisujesz z powrotem.

- `mcp__miro__canvas_get_canvas_composer_skill` — aktualna specyfikacja SVG (zawołać **raz** na sesję Claude Code, cache w pamięci)
- `mcp__miro__canvas_create_from_svg` — tworzy elementy z dokumentu SVG
- `mcp__miro__canvas_read_as_svg` — czyta tablicę jako SVG; każdy element niesie `data-miro-id`
- `mcp__miro__canvas_update_from_svg` — nakłada dokument SVG jako różnicę

Pełna specyfikacja layoutu i kolorów: `.claude/skills/miro-card-spec/SKILL.md`.
Dialekt SVG bierzesz z composera, nie stąd — ta komenda opisuje przebieg.

## Przebieg

### 1. Załaduj stan
- Odczytaj `./sessions/<latest>/state.json`.
- Jeśli `state.mode == "light"` → sesja prowadzi samą rozmowę, kartek nie ma.
  Wypisz *"Tryb light — brak kartek do synchronizacji."* i **STOP**.
- Jeśli `state.miro.enabled == false` → **fallback markdown** (sekcja *Fallback*).
- Zaczytaj `level = state.level`. Wszystkie niezsynchronizowane stickies w
  tym sync trafią do frame'a poziomu `level`.
- `$ARGUMENTS` zawiera `--reflow` → włącz tryb reflow (krok 2.6).

### 1.5 Załaduj specyfikację SVG (raz)
- Jeśli orkiestrator nie ma jeszcze specyfikacji w pamięci sesji → zawołaj
  `mcp__miro__canvas_get_canvas_composer_skill`. Zachowaj wynik do końca tej
  sesji Claude Code. Nazwy elementów, atrybuty pozycji, sposób ustawienia
  koloru i treści wielolinijkowej bierzesz **stamtąd**. Każda ścieżka, która
  zapisuje na tablicę — zwykły sync i `--reflow` — potrzebuje tej specyfikacji.

### 1.6 Odczytaj pozycje z tablicy

Facylitator układa oś czasu **przeciągając kartki w Miro** — to tam zapada
kolejność, nie w stanie sesji. Zanim policzysz miejsce dla czegokolwiek nowego,
sprawdź, gdzie kartki naprawdę stoją.

- Pomijasz ten krok, gdy `state.miro.frames[level] == null` (na tablicy nie ma
  jeszcze nic) albo Miro jest nieaktywne.
- `mcp__miro__canvas_read_as_svg(miro_url = state.miro.url)` — jeden odczyt na sync.
- Dopasuj węzły do kartek po `miro_item_id` (`data-miro-id`) i **nadpisz w stanie
  `x`/`y`** wartościami z tablicy, zaokrąglonymi do liczb całkowitych.
- Policz kartki, które zmieniły pozycję od ostatniego syncu — powiesz o nich w kroku 6.
- Kartka `synced == true`, której nie ma w odczycie (ktoś skasował ją w Miro):
  zostaw ją w stanie, zaloguj do `sync.log` i wymień w wyjściu. Nie stawiasz jej
  drugi raz.
- Odczyt zawiódł → nie przerywaj syncu. Pracuj na `x` ze stanu i uprzedź
  facylitatora, że pozycje mogą być nieaktualne.

### 2. Wyznacz niezsynchronizowane stickies
- Stickies do wysłania: `s.synced != true` (oraz nie `"local"`).
- Jeśli brak i nie ma `--reflow` — wypisz *"Nic nowego do synchronizacji."* i **STOP**.
- Wstępne sortowanie stabilne: `(round ASC, type ASC, author_persona ASC, id ASC)`.

### 2.5 Przyznanie pozycji — tylko nowym kartkom

*(W trybie `--reflow` pomiń ten krok — pozycje przyznaje 2.6.)*

Kartki już postawione zostają tam, gdzie stoją — po kroku 1.6 stan zna ich
prawdziwe `x`/`y`, także te po ręcznym przeciągnięciu. Liczysz pozycje wyłącznie
dla kartek z tej partii.

**Eventy** — mały reasoning pass (orkiestrator, własny kontekst):

- Weź listę rozstawionych eventów ze stanu (`synced == true`), z ich tekstami
  i `x`. To odczyt lokalny — bez wywołania do Miro.
- Dla każdego nowego eventu spytaj *„który moment biznesowy on opisuje?"* i
  znajdź mu **sąsiadów** w sekwencji cyklu życia z
  `.claude/skills/domain-knowledge/domain-brief.md` (sekcja „Cykl życia
  projektu") — to wiedza o domenie, nie o layoucie. Wynikiem są `left_x` i
  `right_x`; brak sąsiada z którejś strony = brzeg ściany (`None`).
- Pogrupuj nowe eventy po luce, w którą trafiły, i dla każdej luki wywołaj
  `gap_positions(left_x, right_x, n)` (patrz `miro-card-spec`) — `n` kartek w
  jednej luce rozkłada się w niej równomiernie.
- Eventy nie pasujące do timeline'a (codzienna rutyna, cykliczne — np. "Telefon
  o 6:30") idą do klastra off-timeline na prawym końcu, 400 px od wiersza cyklu
  życia. Gdy taki klaster urośnie w **osobny proces** biegnący równolegle do
  cyklu życia, przenieś go do wiersza wtórnego (`SECONDARY_EVENT_Y`, hot spoty
  w `SECONDARY_HOT_Y` — patrz `miro-card-spec`).
- Gdy dwa eventy z tej samej partii opisują **wykluczające się wyniki** tego
  samego momentu ("Projekt zaakceptowany" / "Projekt odrzucony"), ułóż je
  kolumnowo przez `fork_pos(branch_x, outcome_idx)` zamiast wciskać oba na
  kręgosłup. W razie wątpliwości zostaw je na kręgosłupie.
- Policz eventy z luk, w których `step` wyszedł poniżej `X_MIN_GAP` — o tym
  powiesz facylitatorowi w kroku 7.

**Hot spoty** — `x` bierz z zapisanego `x` eventu wskazanego przez `relates_to`,
`y` z `hot_spot_pos(anchor_x, stack_idx)`, gdzie `stack_idx` to liczba hot
spotów już zakotwiczonych do tego eventu. Bez `relates_to` — kotwica przy
evencie o największym `x`.

Do jitteru wchodzi `id` kartki, więc pozycja jest deterministyczna i powtarzalna.

### 2.6 Tryb `--reflow`

Wywołanie z `--reflow` przestawia całą ścianę zamiast dokładać kartki. Nadpisuje
tym samym ręczne ułożenie — jeśli facylitator sam przeciągał kartki, powiedz mu
to, zanim przestawisz:

1. Posortuj **wszystkie** eventy chronologicznie — postawione i te z `synced ==
   false` razem — i przyznaj im gęste `x` przez `reflow_positions` (patrz
   `miro-card-spec`). Hot spoty idą za `x` swoich kotwic.
2. `mcp__miro__canvas_read_as_svg` — świeży odczyt tablicy. Obowiązkowy:
   dokument różnicowy budujesz na tym, co Miro ma teraz, nie na samych
   zmienianych węzłach.
3. W odczytanym SVG przesuń **tylko te kartki, które faktycznie zmieniły
   pozycję** — każdą po jej `miro_item_id` — i zapisz całość przez
   `mcp__miro__canvas_update_from_svg`.
4. Zapisz nowe `x`/`y` w stanie.
5. Wyjście: *"Reflow: przesunięto N kartek, M zostało na miejscu."*

Kartki `synced == false` reflow domyka normalną ścieżką z kroków 3–5 w tym samym
przebiegu.

### 3. Złóż dokument SVG
Per algorytm z `miro-card-spec` — jeden timeline eventów z zakotwiczonymi hot
spotami, każda kartka podpisana autorem w drugiej linii, pozycje z kroku 2.5.
Kształt każdego węzła (nazwa elementu, atrybuty, kolor, treść wielolinijkowa)
składasz według specyfikacji z kroku 1.5.

**Pierwszy sync danego poziomu** (`state.miro.frames[level] == null`) — dokument
zawiera, w tej kolejności:

1. **ramkę** `Big Picture — sesja <id>` w `x=0, y=0, w=12000, h=4000`;
2. **strzałkę czasu** wewnątrz ramki: `x=6000, y=350, w=11200, h=80`, strzałka
   w prawo, etykieta `Czas →`;
3. **kartki** z partii — każda wewnątrz ramki, z pozycją i treścią z `card_spec`.

**Kolejne syncy** (`state.miro.frames[level] != null`) — same kartki, dokładane
do istniejącej ramki adresowanej `state.miro.frames[level]`. Ramki ani strzałki
nie emitujesz drugi raz.

Cała partia idzie w **jednym** dokumencie — jedno wywołanie na sync, niezależnie
od liczby kartek.

### 4. Zapisz przez `canvas_create_from_svg`

```python
result = mcp__miro__canvas_create_from_svg(
    miro_url = state.miro.url,
    svg      = svg_document,
)
```

### 5. Odczytaj identyfikatory i zaktualizuj state.json

Po sukcesie zawołaj `mcp__miro__canvas_read_as_svg(miro_url=state.miro.url)` i
dopasuj postawione węzły do kartek z partii — po pozycji, w razie remisu po
treści. Z każdego dopasowania weź `data-miro-id`.

- Dla każdej dopasowanej kartki: `synced = true`, `miro_item_id = <data-miro-id>`
  oraz `x`/`y` **z odczytu** (zaokrąglone do liczby całkowitej) — gdy Miro
  postawiło kartkę inaczej, niż liczyłeś, obowiązuje tablica. Bez `x`/`y`
  następny sync nie ma względem czego liczyć pozycji.
- Kartki, których nie postawiono albo nie dopasowano, zostają `synced = false`,
  bez `x`/`y` — pójdą w następnym syncu. Zaloguj je do `sync.log`.
- **Pierwszy sync** dodatkowo: znajdź w odczycie ramkę o tytule `Big Picture —
  sesja <id>` i zapisz jej `data-miro-id` jako `state.miro.frames[level]`.

### 6. Wyjście

Link do ramki złóż z `state.miro.url` i `state.miro.frames[level]` —
`?moveToWidget=<id-ramki>` to parametr adresu aplikacji Miro w przeglądarce.

```
Sync Miro: +<n> kartek (ramka: <level>).
Tablica: https://miro.com/app/board/<board-id>=/
Ramka:   https://miro.com/app/board/<board-id>=/?moveToWidget=<id-ramki>
Zostało na później: <m> (pójdą w następnym syncu)
```

Gdy krok 1.6 znalazł przesunięte kartki, dopisz linię:

```
Na tablicy przesunięto <j> kartek — biorę te pozycje za obowiązujące.
```

Gdy krok 2.5 naliczył kartki poniżej `X_MIN_GAP`, dopisz jedną linię:

```
<k> kartek trafiło w ciasne miejsce na timeline — /es-sync-miro --reflow rozłoży go równo.
```

### Fallback (Miro nieaktywne)
- Append wszystkie unsynced stickies do `./sessions/<id>/board.md` jako wiersze
  markdown (kolumny: Runda, Typ, Kolor, Treść, Autor).
- Ustaw `synced = "local"` żeby nie były retried.
- Wyjście: *"Miro nieaktywne. {n} stickies zapisanych lokalnie do
  ./sessions/<id>/board.md. /es-sync-miro będzie no-op."*

## Obsługa błędów

- **Zapis zwraca błąd HTTP / network** → zaloguj do `./sessions/<id>/sync.log`,
  **nie udawaj sukcesu**, raportuj userowi co padło.
- **Część kartek nie postawiona albo niedopasowana w odczycie z kroku 5** →
  wymień je userowi, zostaw `synced = false`. Następny sync je dorzuci.
- **Brak ramki w odczycie po pierwszym syncu** → ostrzeż usera i przerwij — coś
  poszło źle, prawdopodobnie dokument z ramką nie został zaaplikowany.
- **Odczyt po zapisie zawiódł, choć zapis się udał** → kartki stoją na tablicy,
  ale bez `miro_item_id`. Zostaw `synced = false`, zaloguj i powiedz userowi:
  ponowny sync postawi duplikaty, więc najpierw trzeba sprawdzić tablicę.
- **Event bez zapisanego `x`, choć `synced == true`** (stan sprzed wprowadzenia
  pozycji w stanie) → normalnie uzupełnia go odczyt z kroku 1.6. Gdy odczyt się
  nie udał, potraktuj ścianę jako nieznaną i zaproponuj `--reflow`, który
  przyzna `x` wszystkim eventom naraz.
- **Sticky bez `text`** → pomiń, zaloguj.

## Reguły
- **Najpierw bulk** — wszystkie unsynced stickies w jednym dokumencie SVG.
- **Przyrostowo** — liczysz pozycje tylko dla nowych kartek; postawione zostają
  na `x`/`y` odczytanych z tablicy w kroku 1.6. Koszt syncu zależy od liczby
  nowych kartek, nie od rozmiaru ściany.
- **Odczyt po każdym zapisie** — stamtąd biorą się `data-miro-id`. Przy
  `--reflow` odczyt jest też **przed** zapisem: różnicę budujesz na aktualnym
  stanie tablicy.
- **Adresuj po id, nie po treści** — kartkę rusza jej `miro_item_id`. Dwie
  kartki o tym samym tekście są rozróżnialne.
- **Przestawianie tylko na żądanie** — kartki rusza wyłącznie `--reflow`.
- **Leniwa ramka** — ramka powstaje przy pierwszym sync danego poziomu, nie w `/es-start`.
- **Idempotentnie** — `synced=true` pomijane.
- **Deterministyczna pozycja** — ten sam input → ten sam x,y.
- **Bez konektorów** — linki hot_spot↔event zostawiamy facylitatorowi do
  narysowania w Miro UI.

## Nie rób
- Nie usuwaj kartek z Miro automatycznie, nawet jeśli zniknęły ze `state.json`.
  Ostrzeż usera, niech decyduje.
- Nie buduj dokumentu różnicowego z samych zmienianych węzłów — element
  pominięty w różnicy może zniknąć z tablicy. Zawsze wychodź od świeżego
  `canvas_read_as_svg`.
- Nie zmieniaj kolorów istniejących kartek bez prośby usera.
- Nie zapisuj niczego bez wcześniejszego `canvas_get_canvas_composer_skill`
  w tej sesji.
