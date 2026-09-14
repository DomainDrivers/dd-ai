---
name: event-storming-method
description: Metodyka Event Stormingu według Brandoliniego w zakresie potrzebnym coachowi — co niesie ściana Big Picture, kolory kartek, semantyka hot spot kontra pivotal event, metaprogramy komunikacyjne i zbiór zwrotów facylitatora. Ładowana przez facilitator-coacha.
---

# Event Storming — dokumentacja metodyki

Źródło: *Introducing EventStorming* (Brandolini, 2021).

## Big Picture — co niesie ściana

**Cel**: chaotycznie zbadać cały biznes. Na razie bez struktury.

- Wszyscy piszą domain events (pomarańczowe) w ciszy, równolegle, bez ustalonej kolejności
- Chaotyczna eksploracja trwa tyle, ile trzeba — u Brandoliniego bywa 30–45 minut. Chronologię wymuszasz, gdy tempo nowych kartek wyraźnie spada, nie po zegarku
- Wyłaniają się hot spoty (różowe) — „nie wiemy", „to jest błędne", „nie ma zgody"
- Pivotal events (pomarańczowe, oznaczone pionową linią) = zmiany stanu biznesu

Pivotal events i swimlane'y są **pierwszym śladem granic**. Kontekstów jeszcze
nie rysujesz, ale już tu widać, gdzie historia zmienia bohatera i słownik. Po to
ich szukasz, a nie dla ozdoby osi.

**Gotowe, gdy**: oś czasu opowiada spójną historię; widoczne są główne hot spoty;
zespół zgadza się co do kilku pivotal events.

Dalsza droga metody prowadzi przez Process Modeling (commands, policies, read
modele, agregaty) do Software Design (bounded contexty, wzorce taktyczne). Ten
słownik należy do tamtych poziomów — gdy pada w Big Picture, coach czyta to jako
**przedwczesne projektowanie** albo **mieszanie poziomów** i sprawdza, czy
facylitator zaparkował temat.

## Kartki na ścianie Big Picture

| Typ | Kolor | Kształt | Kiedy |
|---|---|---|---|
| Domain event | 🟧 pomarańczowy | kwadrat | Zawsze. Czas przeszły, istotny biznesowo |
| Pivotal event | 🟧 pomarańczowy + pionowa linia | kwadrat + linia | Duża zmiana stanu |
| Hot spot | 🩷 różowy + 🟥 czerwona kropka | karteczka z czerwoną kropką | Wszędzie tam, gdzie brak zgody, zamieszanie, brakujące informacje |

**Hot spot kontra pivotal event**: hot spot znaczy „tu jest napięcie" — spór,
niewiedza, ryzyko. Pivotal event znaczy „tu historia zmienia bieg". Kartka może
być jednym albo drugim z zupełnie innego powodu; nie mieszaj ich w ocenie.

## Metaprogramy komunikacyjne

Metaprogramy to filtry poznawcze wyższego rzędu, które kształtują to, **jak**
człowiek sortuje informacje i na nie reaguje — niezależnie od tego, **co** wie.
Uruchamiają się **kontekstowo** i można je wykryć po wzorcach mowy
(kwantyfikatory, charakterystyczne zwroty). W Event Stormingu tłumaczą,
dlaczego ten sam ruch facylitacyjny działa inaczej w zależności od tego, z kim
pracujesz. Każda persona w tym środowisku (sandbox) ma **charakterystyczny
profil metaprogramów** (wskazany we własnym pliku agenta); zadaniem facylitatora
jest odczytywać żywy sygnał w każdej rundzie i się dostosowywać. Źródło:
Shelle Rose Charvet, *Words That Change Minds*.

### 1. Podobieństwo ↔ Różnica (sortowanie informacji)
- **Podobieństwo** — szuka tego, co pasuje; nie lubi/boi się zmiany; podkreśla
  analogie; zgadza się przez brak różnicy. Markery: *„dokładnie tak"*,
  *„analogicznie"*, *„na tej samej zasadzie"*, *„już to mieliśmy"*.
- **Różnica** — nastawiona na przeciwieństwa; uwielbia zmianę; wybiera przez
  odrzucenie. Markery: *„to zupełnie co innego"*, *„nie, bo…"*, *„tego nie chcę"*.
- **Taktyka ES** — Gdy osoby o nastawieniu na **podobieństwo** modelują
  **TO-BE**, klonują stary system; przełam ślad pamięciowy zakotwiczony w
  rzeczownikach, pracując wokół **pivotal events** i formułując eventy jako
  **czasowniki w czasie przeszłym**. Gdy osoby o nastawieniu na **różnicę**
  modelują **AS-IS**, model odrywa się od rzeczywistości; obramuj go
  **eventami początkowym i końcowym** jako granicą celu.

### 2. Ogólność ↔ Szczegółowość (opisywanie rzeczywistości)
- **Ogólność** — ogólne kwantyfikatory, uogólnia z jednego przypadku, ignoruje
  kolejność, frustruje ją szczegół. Markery: *„zawsze"*, *„nigdy"*,
  *„ogólnie to jest tak"*, *„dokąd ty zmierzasz?"*. Ryzyko: event zbyt
  ogólnikowy (sparafrazowany command).
- **Szczegółowość** — szczegółowe kwantyfikatory, sekwencje, liniowa kolejność,
  potrafi zgubić się w szczegółach. Markery: *„istnieją takie przypadki"*,
  relacjonowanie krok po kroku.
- **Taktyka ES** — Zbyt ogólne eventy → zadawaj pytania o **behaving / becoming**
  (kto to zmienia, dlaczego, jaki jest efekt, czym się staje). Aby wciągnąć
  osobę o nastawieniu na **ogólność** w szczegół, najpierw wyraź **pozytywną
  wspólną intencję** (*„żeby lepiej zrozumieć i zredukować ryzyko, muszę zejść
  głębiej"*) i sprawdź, czy to właściwy moment. Aby podnieść osobę o nastawieniu
  na **szczegółowość**, zadaj pytanie **W GÓRĘ** (*„dlaczego to jest ważne?"*) i
  zakotwicz w **celu procesu** (ostatni event) oraz **celu spotkania**.

### 3. Unikanie ↔ Dążenie (orientacja wobec świata)
- **Unikanie / od (problem)** — filtruje zagrożenia; motywują ją problemy,
  których chce uniknąć; wie, czego NIE robić. Markery: *„żeby tylko nie było…"*,
  *„uważaj"*, *„to się posypie"*.
- **Dążenie / do (cel)** — filtruje cele/korzyści; motywują ją nagrody; ślepa na
  problemy; traktuje je jako zwykłe przeszkody. Markery: *„jest dobrze, ale może
  być lepiej"*.
- **Taktyka ES** — Produktywnie ukierunkuj osobę nastawioną na unikanie:
  poproś, aby **zaznaczyła w procesie miejsca zagrażające celowi** (jej problemy
  stają się hot spotami). Nie mów jej „będzie dobrze"; buduj pewność, że
  *cokolwiek się stanie, będziemy wiedzieć, co zrobić*. Eksperci techniczni,
  którzy wcześnie wyliczają problemy, są odbierani przez biznes (dążenie) jako
  **defetyzm** — zanim podniesiesz problem, zapytaj *„czy to najlepszy moment?"*
  i powiąż go z **zagrożeniem dla konkretnego celu**. Nie zarzucaj problemami
  osoby nastawionej na dążenie.

### 4. Wewnętrzny ↔ Zewnętrzny układ odniesienia
- **Wewnętrzny** — dowód czerpie z własnej refleksji; trudny do zarządzania;
  pochwała buduje status, nie potwierdzenie. Markery: *„ja wiem"*,
  *„sam sprawdzę"*, *„u nas to inaczej"*.
- **Zewnętrzny** — polega na opiniach/danych innych; potrzebuje potwierdzenia.
  Markery: *„bo klienci"*, *„bo wszyscy mówią"*, *„bo Vernon / badania mówią"*.
- **Taktyka ES** — Osoby o **zewnętrznym** układzie odniesienia nie przekona
  twoja niepoparta opinia, o ile nie masz uznanego autorytetu — przynieś
  **dane, badania lub nazwany autorytet**. Osoby o **wewnętrznym** układzie
  odniesienia nie przekona cudze doświadczenie — zaproponuj jej **własny
  eksperyment** (*„masz repo, sprawdź sam"*). Wobec **wszystkowiedzącego**
  (ekspert: wewnętrzny + unikanie) uczyń go **właścicielem problemu** — zapytaj,
  jak **on** by go rozwiązał — i zostaw **złoty most** (wyjście z twarzą):
  zapytaj, z ciekawości a nie z uporu, czy jego twierdzenie obowiązuje także
  **w innym kontekście**.

> Metaprogramy nie są stałymi etykietami osobowości — uruchamiają się
> **kontekstowo**. Odczytuj żywy sygnał w każdej rundzie; nie zakładaj z góry.

## Zbiór zwrotów facylitatora

Materiał na konkretne sugestie w debriefie („następnym razem spróbuj…").

| Sytuacja | Zwrot |
|---|---|
| Rozpoczęcie w ciszy | "Zaczynamy w ciszy — każdy pisze sam" |
| Odłożenie tematu na później | "Zaparkujmy to na razie" |
| Pytanie o kolejny krok | "Co się dzieje potem?" |
| Zaproszenie innych głosów | "Dziękuję. Co inni widzą inaczej?" |
| Wyciągnięcie cichego uczestnika | "Kasiu, a Twoim zdaniem co się dzieje na początku?" |
| Prowokacja, gdy wszystko się zgadza | "A co jeśli to nie tak działa?" |
| Porządkowanie w czasie | "Teraz układamy w czasie — co najpierw, co potem" |
| Oznaczenie hot spota | "Daj różową karteczkę — wracamy do tego" |
| Szukanie pivotal eventu | "Które zdarzenie tu zmienia bieg sprawy?" |
| Przybliżenie na szczegół | "Wjeżdżamy w szczegół" |
| Wstrzymanie projektowania | "Jeszcze nie projektujemy. Wracamy do zdarzeń" |
