---
name: miro-card-spec
description: Jak zmapować kartkę Event Stormingu na element tablicy Miro w modelu canvas (SVG). Używane przez orkiestratora podczas synchronizacji stanu do Miro przez serwer MCP miro (https://mcp.miro.com).
---

# Specyfikacja kartki Miro

Serwer Miro MCP opisuje tablicę jako **dokument SVG**. Orkiestrator czyta
tablicę jako SVG, edytuje ten SVG i zapisuje go z powrotem:

- `mcp__miro__canvas_get_canvas_composer_skill` — aktualna specyfikacja SVG
  (zawołać **raz** na sesję Claude Code, cache w pamięci)
- `mcp__miro__canvas_create_from_svg` — tworzy elementy z dokumentu SVG
- `mcp__miro__canvas_read_as_svg` — czyta tablicę jako SVG; każdy element niesie
  stabilne `data-miro-id`
- `mcp__miro__canvas_update_from_svg` — nakłada dokument SVG jako różnicę
  (tworzy, aktualizuje i usuwa elementy)

> **Składnię wiążesz w runtime.** Miro nie publikuje dialektu SVG — wydaje go
> `canvas_get_canvas_composer_skill`. Ta specyfikacja mówi, **co** ma stanąć na
> tablicy: jakie typy kartek, w jakim kolorze, w jakich współrzędnych, z jakim
> podpisem. **Jak** to zapisać w węźle SVG — nazwy elementów, atrybuty, sposób
> ustawienia koloru i treści wielolinijkowej — bierzesz z odpowiedzi composera.
> Gdy jedno przeczy drugiemu, wygrywa composer: on opisuje działający format,
> ta specyfikacja opisuje zamiar.

## Zakres tej umiejętności

Ściana mieści eventy i sporadyczne hot spoty — to właśnie mapuje ten layout.
Gdy historia jest opowiedziana eventami, timeline czyta się sam.

## Dozwolone typy kartek (Big Picture)

| Typ kartki ES | Kolor | Kształt | Dodatki |
|---|---|---|---|
| `event` | pomarańczowy | kwadrat | kręgosłup timeline'u |
| `hot_spot` | różowy | kwadrat | poprzedź treść znakiem `🔥`; przyczepiony blisko eventu, którego dotyczy |

To cały wszechświat typów kartek w Big Picture. Orkiestrator **musi odrzucić**
każde wyjście persony, które używa `command`, `actor`, `aggregate`,
`read_model`, `policy`, `external_system`, `question`, `pivotal_event`,
`bounded_context` lub czegokolwiek innego — poproś personę ponownie albo odrzuć
kartkę i zaloguj to.

Pivotal eventy pojawiają się jako *flaga* na wybranych eventach później
(podczas cięcia na poziomie Process Modeling), a nie jako osobny typ na tym
etapie.

Kolor jest **zamiarem**, nie literałem: pomarańczowy dla eventu, różowy dla hot
spotu — zgodnie z legendą w `.claude/skills/event-storming-method/SKILL.md`.
Konkretną wartość (nazwa z palety, hex, atrybut wypełnienia) bierzesz ze
specyfikacji composera. Big Picture używa tych dwóch kolorów i żadnego więcej.

## Strategia frame'ów

Big Picture żyje w pojedynczej ramce, tworzonej **leniwie przy pierwszej
synchronizacji** (nie przy `/es-start`). Stan śledzi `data-miro-id` tej ramki —
to on adresuje ją przy kolejnych zapisach.

| Poziom | Tytuł frame'a | Rozmiar |
|---|---|---|
| `big_picture` | `Big Picture — sesja <id>` | **12000 × 4000** — długie poziome płótno (prawdziwa ściana Big Picture ma 30–80 eventów na jednym timeline; zamiast zmieniać rozmiar w trakcie sesji, przydzielamy płótno z góry) |

Frame jest tworzony w `x=0, y=0` (środek boardu). Przy szerokości 12000 oznacza
to lewy górny róg w (-6000, -2000) i prawą krawędź w (+6000, +2000) — mnóstwo
miejsca po prawej, by timeline mógł rosnąć bez zmiany rozmiaru.

**Szkielet pierwszej synchronizacji** — razem z ramką orkiestrator emituje
jeden element rusztowania:

- **strzałkę timeline'u** blisko góry (y=350), kształt strzałki w prawo
  (w=11200, h=80), z etykietą `"Czas →"` — wizualna wskazówka, że eventy płyną
  od lewej do prawej.

Bez nagłówka lane'u hot spotów. Bez etykiet faz (te facylitator dodaje w
interfejsie Miro, gdy fazy się wyłonią).

Schemat stanu:

```json
"miro": {
  "board_id": "uXjVHR6I38g=",
  "enabled": true,
  "url": "https://miro.com/app/board/uXjVHR6I38g=/",
  "frames": {
    "big_picture": "3458764612345678901"
  }
}
```

`frames[level]` trzyma `data-miro-id` ramki — wartość odczytaną z
`canvas_read_as_svg` zaraz po jej utworzeniu.

## Konwencja pozycji — bez lane'ów

Zapomnij o lane'ach. Big Picture to **jeden timeline eventów** plus hot spoty,
które **unoszą się blisko eventu, który kwestionują**. Współrzędne są względem
frame'a (lewy górny róg = 0,0, x,y elementu = środek elementu).

```
y= 350   strzałka timeline'u  (SHAPE right_arrow, w=11200 h=80)
y= 600   podłoga hot spotów   (hot_spots unoszą się między ~500 a ~750,
                              zakotwiczone do x powiązanego eventu)
y= 900   eventy               ← KRĘGOSŁUP TIMELINE'U (orange, w=220)
y=1700   wiersz równoległy/wtórny (opcjonalnie: gdy osobny proces biegnie
                                równolegle i orkiestrator zdecyduje się
                                zacząć nowy wiersz zamiast nakładać)
```

`y=900` to **główny kręgosłup timeline'u**. Eventy płyną od lewej do prawej
chronologicznie.

`y=600` to **typowa kotwica y dla hot spotów** — *typowa*, nie wymuszona.
X hot spotu to x eventu, którego dotyczy (plus jitter); jego y to 600 plus
jitter. Jeśli dwa hot spoty zakotwiczają się do tego samego eventu,
orkiestrator układa je pionowo (drugi w y≈400, reguły jittera trzymają je
lekko poza siatką).

`y=1700` jest zarezerwowane dla **wiersza procesu równoległego/wtórnego** —
patrz niżej.

### Eventy: chronologiczny kręgosłup

Kartki typu `event` muszą być uporządkowane **chronologicznie** wzdłuż cyklu
życia projektu. Lewo = najwcześniej (oferta, umowa); prawo = najpóźniej
(przekazanie, płatność, wygaśnięcie gwarancji).

**Tablica jest źródłem prawdy o pozycjach.** Gdy kartka trafi na ścianę, jej
`x`/`y` lądują w `state.json`, a każdy kolejny sync odświeża je odczytem z Miro.
Kartka przeciągnięta ręką facylitatora zostaje tam, gdzie ją postawił, a nowe
kartki liczą się względem tego, co naprawdę stoi na ścianie. Orkiestrator
przelicza pozycje wyłącznie dla kartek nowych — koszt syncu zależy od liczby
świeżych kartek, nie od rozmiaru ściany.

Nowy event wchodzi **między sąsiadów, w połowie wolnej przestrzeni**:

```
sąsiedzi A(x=400) i B(x=640)   →  nowy event x = 520
za ostatnim (x_max)            →  nowy event x = x_max + X_STEP
przed pierwszym (x_min)        →  nowy event x = x_min − X_STEP
```

Orkiestrator uruchamia małe przejście rozumujące **tylko na nowych eventach**,
by znaleźć im sąsiadów: czyta tekst eventu, pyta *„który moment biznesowy on
opisuje?"* i porównuje z już rozstawionymi eventami (ich teksty i `x` ma pod
ręką w `state.json` — bez wywołania do Miro). Sąsiedzi zostają tam, gdzie byli.

Gdy kilka nowych eventów trafia w tę samą lukę, orkiestrator rozkłada je
równomiernie w jej obrębie.

Kolejne wstawki w to samo miejsce zagęszczają ścianę — dokładnie tak, jak przy
prawdziwej tablicy, gdzie kartki zaczynają na siebie zachodzić. Gdy odstęp
między środkami spadnie poniżej `X_MIN_GAP`, orkiestrator kartkę i tak stawia,
ale liczy takie przypadki i mówi o nich w wyjściu synchronizacji — facylitator
decyduje, czy rozprostować timeline przez `/es-sync-miro --reflow`.

Eventy poza timeline'em (codzienna rutyna / cykliczne, np. *"Telefon o 6:30 —
kto nie przyjdzie"*) trafiają na **daleko prawą stronę** timeline'u w luźnym
klastrze z odstępem 400 px od wiersza cyklu życia. Są zapachem mieszania
poziomów; layout czyni to widocznym, żeby facylitator mógł zdecydować o ich
odstawieniu.

### Reflow — rozprostowanie timeline'u na żądanie

`/es-sync-miro --reflow` to jedyna ścieżka, w której **narzędzie** przestawia już
postawione kartki — i jedyna, która nadpisuje ręczne ułożenie facylitatora. Orkiestrator sortuje wszystkie eventy chronologicznie, przyznaje im
gęste `x` (`X_START + rank * X_STEP`), przesuwa zakotwiczone hot spoty za ich
eventami i zapisuje zmianę przez `canvas_update_from_svg` **tylko dla kartek,
które faktycznie zmieniły pozycję** — każdą adresuje jej `data-miro-id`. Nowe
`x` lądują w stanie.

Reflow jest ruchem facylitatora, nie tłem. Przestawienie ściany w środku
rozmowy dezorientuje salę — a to, kiedy wygląd tablicy nadąża za rozumieniem
domeny, jest częścią warsztatu.

### Hot spoty: zakotwiczone do eventów

Hot spot nie ma własnej pozycji — ma **relację**. Wyjście persony dla hot spotu
POWINNO zawierać `relates_to: ev_NNN` wskazujące, który event on kwestionuje.
Jeśli persona to pominie, orkiestrator albo:

- pyta personę raz o relację, albo
- umieszcza hot spot przy *najpóźniejszym* dotychczasowym evencie (luźny hot
  spot, dryfujący na prawo ściany) i oznacza go `relates_to: null` w stanie

Umiejscowienie: `x = anchor_event_x ± jitter`, `y = 600 ± jitter`. Układanie w
stos, gdy wiele hot spotów zakotwicza się do tego samego eventu: drugi hot spot
w `y=400`, trzeci w `y=200` (im wyżej na ścianie, tym bardziej „luźne" pytanie
wizualnie).

### Wiersz procesu równoległego / wtórnego

Czasem kawałek eventów opisuje **proces dziejący się równolegle** do głównego
cyklu życia — na przykład codzienna rutyna placu obok cyklu życia projektu albo
proces przetargowy nakładający się na wykonawstwo. Orkiestrator może zdecydować
o umieszczeniu ich w **wtórnym wierszu w y=1700** zamiast upychać je na główny
kręgosłup. Hot spoty zakotwiczone do eventów wtórnego wiersza siedzą w `y=1400`
(nad swoim wierszem), ta sama reguła.

Persona nie prosi o wtórny wiersz; orkiestrator decyduje na podstawie tego, czy
moment biznesowy eventu jest *poza-osiowy* na tyle, by zasłużyć na własny
wiersz. Decyzja jest zachowawcza — większość eventów zostaje na głównym
kręgosłupie.

### Rozgałęzienia: układanie kolumnowe

Gdy punkt decyzyjny ma **alternatywne wyniki** (np. *"Projekt zaakceptowany do
oferty"* vs *"Projekt odrzucony"*), oba eventy są **układane pionowo** przy
zbliżonym x — jeden nad drugim.

Pozycje (względem x rozgałęzienia):

```
y= 800 ± jitter   alternatywny wynik A (np. happy path)
y=1000 ± jitter   alternatywny wynik B (np. odrzucenie)
```

Bez wyrównania do piksela — mały jitter x jest w porządku. To wizualizuje
rozgałęzienie bez pretensji do struktury swim-lane.

Jeśli rozgałęzienie ma więcej niż dwa wyniki, ułóż je w stos w `y= 800, 1000,
1200` w kolejności wejściowej. Powyżej trzech facylitator powinien rozważyć,
czy to naprawdę alternatywy, czy stany sekwencyjne.

Persona nie musi wskazywać rozgałęzienia jawnie; orkiestrator wykrywa je, gdy
dwa eventy z tej samej partii opisują **wzajemnie wykluczające się wyniki tego
samego momentu biznesowego** (np. oba odnoszą się do tego samego poprzedzającego
eventu, ale zawierają w tekście „accepted/rejected", „won/lost",
„approved/denied"). W razie niejednoznaczności umieść oba na głównym
kręgosłupie i pozwól facylitatorowi przeciągnąć jeden w dół w interfejsie Miro.

### Jitter — ludzie, nie siatki

Wszystkie umiejscowienia dostają **deterministyczny jitter**, żeby layout
wyglądał jak prawdziwa ściana warsztatowa (ludzie nie przyklejają kartek na
idealnej siatce). To samo id kartki → ten sam jitter (idempotencja).

```python
X_START   = 400
X_STEP    = 240        # sticky 220 wide + 20 gap
X_MIN_GAP = 120        # ciaśniej → policz i zaproponuj --reflow
JITTER_X  = 50
JITTER_Y  = 30
HOT_SPOT_JITTER = (20, 10)   # ciaśniej dla zakotwiczonych hot spotów
EVENT_Y   = 900
HOT_Y     = 600
SECONDARY_EVENT_Y = 1700
SECONDARY_HOT_Y   = 1400

import hashlib
def deterministic_jitter(sid, rx=JITTER_X, ry=JITTER_Y):
    h  = hashlib.md5(sid.encode()).digest()
    jx = (h[0] | (h[1] << 8)) % (2*rx + 1) - rx
    jy = (h[2] | (h[3] << 8)) % (2*ry + 1) - ry
    return jx, jy
```

Pojemność: `(12000 − 400 − 220/2 − JITTER_X) / X_STEP ≈ 47` eventów na wiersz.
Wystarczająco na normalną sesję Big Picture.

### Duplikaty

Gdy kilka person opisuje ten sam event różnymi słowami (np. Prezesowe *"Umowa z
klientem podpisana"* i Mecenasowe *"Umowa z inwestorem podpisana"*), auto-layout
umieszcza je w sąsiednich kolumnach. Z jitterem sąsiedzi lądują lekko poza
wyrównaniem — dokładnie ten obraz, który widzisz przy prawdziwej ścianie Event
Stormingu, gdy kilka osób przykleja kartki blisko siebie. Facylitator scala
duplikaty, przeciągając je w interfejsie Miro; ta rozmowa o konsolidacji jest
sednem Big Picture i nie powinna być automatyzowana.

## Podpis autora na każdej kartce

Każda kartka **musi pokazywać, kto ją umieścił na ścianie** — prawdziwa ściana
Event Stormingu ma kartki różnymi charakterami pisma; przybliżamy to, dopisując
pod treścią krótką etykietę persony jako **drugą linię** tekstu kartki:

```
linia 1:  {text}            (hot spot: 🔥 {text})
linia 2:  — {author_short}
```

Sposób zapisania dwóch linii w węźle SVG bierzesz ze specyfikacji composera.
Miro renderuje drugą linię mniejszym krojem pod główną treścią — czyta się jak
podpis.

`author_short`:

```python
AUTHOR_SHORT = {
    "prezes-japinski":    "Prezes",
    "ewa-pm":             "Ewa",
    "marek-brygadzista":  "Marek",
    "kasia-junior-dev":   "Kasia",
    "tomek-architekt":    "Tomek",
    "agnieszka-ksiegowa": "Agnieszka",
    "piotr-mecenas":      "Mecenas",
    "gerome-don-kichot":  "Gerome",
}
```


## Umiejscowienie algorytmiczne (pseudokod orkiestratora)

```python
COLOR  = {"event": "pomarańczowy", "hot_spot": "różowy"}   # wartość → spec composera
PREFIX = {"hot_spot": "🔥 "}

def gap_positions(left_x, right_x, n):
    """Pozycje n nowych eventów w luce między sąsiadami (None = brzeg ściany).
    Zwraca (xs, step); step poniżej X_MIN_GAP znaczy, że ściana się zagęściła."""
    if left_x is None and right_x is None:
        return [X_START + i * X_STEP for i in range(n)], X_STEP
    if left_x is None:
        return [right_x - (n - i) * X_STEP for i in range(n)], X_STEP
    if right_x is None:
        return [left_x + (i + 1) * X_STEP for i in range(n)], X_STEP
    step = (right_x - left_x) / (n + 1)
    return [left_x + (i + 1) * step for i in range(n)], step

def hot_spot_pos(anchor_event_x, stack_idx=0):
    # stack_idx = ile hot spotów już wisi przy tym samym evencie
    y = HOT_Y - stack_idx * 200
    return anchor_event_x, y

def fork_pos(branch_x, outcome_idx):
    return branch_x, 800 + outcome_idx * 200

def reflow_positions(events_in_chrono_order):
    """Tylko dla /es-sync-miro --reflow."""
    return {e["id"]: X_START + rank * X_STEP
            for rank, e in enumerate(events_in_chrono_order)}

def card_spec(sticky, x_base, y_base):
    """Zamiar kartki: gdzie stoi, jakiego jest koloru, co na niej piszemy.
    Węzeł SVG z tego składasz według specyfikacji composera."""
    import hashlib
    t = sticky["type"]
    rx, ry = (HOT_SPOT_JITTER if t == "hot_spot" else (JITTER_X, JITTER_Y))
    h  = hashlib.md5(sticky["id"].encode()).digest()
    jx = (h[0] | (h[1] << 8)) % (2*rx + 1) - rx
    jy = (h[2] | (h[3] << 8)) % (2*ry + 1) - ry
    return {
        "x":      x_base + jx,
        "y":      y_base + jy,
        "w":      220,
        "kolor":  COLOR[t],
        "linie": [PREFIX.get(t, "") + sticky["text"],
                  "— " + AUTHOR_SHORT[sticky["author_persona"]]],
    }
```

Do stanu zapisujesz `x`/`y` **po jitterze** — te same liczby, które poszły do
Miro. To one opisują, gdzie kartka faktycznie stoi, więc względem nich liczą się
pozycje kolejnych kartek.

**Dokument pierwszej synchronizacji** Big Picture zawiera, w tej kolejności:

1. ramkę `Big Picture — sesja <id>` w `x=0, y=0, w=12000, h=4000`;
2. strzałkę czasu wewnątrz ramki (`x=6000, y=350, w=11200, h=80`, etykieta
   `Czas →`);
3. po jednym węźle na kartkę z partii, każdy z `card_spec` — wewnątrz ramki.

**Kolejne synchronizacje** dokładają same kartki do istniejącej ramki,
adresowanej jej `data-miro-id` ze stanu. Ramki ani strzałki nie emitujesz
drugi raz.

## Idempotencja

Każda kartka niesie flagę `synced: bool` (domyślnie `false`), przyznaną pozycję
oraz `miro_item_id` — `data-miro-id` odczytane z Miro po jej postawieniu:

```json
{ "id": "ev_001", "type": "event", "text": "...", "round": 1,
  "synced": true, "x": 400, "y": 911, "miro_item_id": "3458764612345678902" }
```

Po udanym zapisie orkiestrator odczytuje tablicę przez `canvas_read_as_svg`,
dopasowuje postawione węzły do kartek z partii (po pozycji i treści), zapisuje
im `miro_item_id`, ustawia `synced: true` i utrwala `x`/`y`. Kolejne
uruchomienia filtrują po `synced == false`.

Zapisane `x`/`y` czynią synchronizację przyrostową: pozycje nowych kartek
wyliczasz względem tych współrzędnych, prosto ze stanu — bez odczytu z Miro.

`miro_item_id` jest tym, co czyni późniejszą edycję i `--reflow` bezpiecznymi:
kartkę adresujesz identyfikatorem, nie jej tekstem. Dwie kartki o tej samej
treści są rozróżnialne, a emoji w treści niczego nie psuje.

> **Zapis różnicowy jest różnicą wobec tego, co Miro ma teraz.** Dokument dla
> `canvas_update_from_svg` buduj zawsze na świeżym `canvas_read_as_svg`, nigdy
> nie składaj go z samych zmienianych węzłów — element pominięty w różnicy może
> zostać usunięty z tablicy. Zakres tego zachowania potwierdza specyfikacja
> composera; do czasu potwierdzenia trzymaj się odczytu przed każdym zapisem
> różnicowym.

## Łagodna degradacja

Jeśli serwer MCP miro nie jest połączony (OAuth niewykonany w `/mcp`, serwer
nieosiągalny albo dowolne wywołanie `mcp__miro__*` zawiedzie):

- Dopisz niezsynchronizowane kartki do `./sessions/<id>/board.md` jako wiersze
  Markdown
- Zaloguj awarię do `./sessions/<id>/sync.log`
- Oznacz kartki `synced: "local"`, żeby nie były ponawiane przy następnej
  synchronizacji
- Powiedz facylitatorowi **raz na sesję** (nie w każdej rundzie)

```markdown
| Runda | Typ | Kolor | Treść | Autor |
|------:|-----|-------|-------|-------|
| 3 | event | 🟧 pomarańczowy | Pracownik przeniesiony do projektu | ewa-pm |
| 3 | hot_spot | 🩷 różowy | 🔥 Kto to zatwierdza?           | piotr-mecenas |
```
