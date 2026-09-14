# Domain brief — smart-schedule (firma Prezesa Japińskiego)

> Materiał referencyjny. Persony czytają go przez własny filtr wiedzy —
> co która naprawdę wie, a czego nie, opisuje jej plik persony.

## Kim jest klient

**Prezes Janusz Japiński** — właściciel firmy budowlanej / generalnego wykonawcy.
Prowadzi wiele projektów równolegle (osiedla mieszkaniowe, hale, infrastruktura).
Każdy projekt = wiele etapów, ekipy, ciężki sprzęt (koparki, dźwigi),
materiały, ludzie z uprawnieniami.

## Co prezes mówi

> "Dużo tych projektów, które robimy, one się de facto później nie udają.
> I pracownicy mogli ten czas spędzić na jakimś mniej ryzykownym projekcie."

> "Jest taki przycisk w tym systemie, który mam, on słabo działa.
> Taki, który pozwala przydzielić pracownika do innego projektu."

> "Jak przesuwam do innego projektu, to wtedy pojawia mi się,
> czy zarobię wtedy więcej na tym przesunięciu.
> Czy ryzyko po prostu wszystkich projektów w moim systemie jest mniejsze."

> "Jak ryzyko opóźnienia w fundamentach wpływa na stropy i elektrykę..."

> "Nie mogę wysłać do projektu kogoś, kto nie ma odpowiedniego zezwolenia,
> bo ktoś się kończy [stłucze] kryminałem. Zamykają mnie. Nie chciałbym tego."

> "Często tydzień przed dniem, w którym miałem zapłacić jakąś karę za spóźnienie,
> chciałbym dostać informacje o tym."

> "Faza projektu to taka część projektu, która zawiera etapy, które od siebie są
> niezależne. W kolejnych fazach etapy są od siebie zależne."

## Główne bóle (driver problems)

1. **Ryzyko projektowe nie jest wykrywane na czas.** Projekty padają po starcie.
2. **Przesuwanie pracowników między projektami działa, ale ślepo** — nie widać czy się to opłaca finansowo ani jak zmienia globalne ryzyko.
3. **Symulacje "co jeśli"** — co jeśli dodam koparkę? co jeśli zatrudnię osobę? co jeśli zmienię specjalistę? — nie istnieją.
4. **Ryzyko kaskadowe** — opóźnienie w jednym etapie/fazie wpływa na kolejne, ale system tego nie pokazuje.
5. **Compliance** — wysłanie pracownika bez uprawnień to ryzyko karne (kryminał, zamknięcie firmy).
6. **Kary umowne** — chciałby wczesnego alertu (tydzień wcześniej) z opcjami mitygacji.
7. **Planowanie faz** — układanie etapów w fazy jest trudne, gotowe narzędzia są drogie/wolne/nierozszerzalne.

## Słownik domenowy (ubiquitous language — kandydaci)

| Polski | Znaczenie ground-truth |
|---|---|
| Projekt | Konkretne przedsięwzięcie (np. "osiedle Słoneczna 12") |
| Faza projektu | Grupa etapów. Etapy *w fazie* są niezależne. Etapy *między fazami* są zależne. |
| Etap | Najmniejsza jednostka pracy (np. "wylanie fundamentów") |
| Zasób | Pracownik LUB sprzęt LUB materiał |
| Umiejętność / capability | Co zasób potrafi (spawanie, prowadzenie koparki, kierownik budowy) |
| Uprawnienie | Certyfikat / pozwolenie wymagane prawnie — brak = ryzyko karne |
| Allocation / przydział | Przypisanie zasobu do etapu na okres |
| Przenoszenie / transfer | Wycofanie zasobu z etapu A i przypisanie do etapu B |
| Demand / zapotrzebowanie | "Etap X potrzebuje pracownika z umiejętnością Y w okresie T" |
| Symulacja | "Co jeśli dodam ten zasób / przeniosę tego pracownika?" — prognoza ROI i ryzyka |
| Ryzyko | Prawdopodobieństwo nieosiągnięcia planu (opóźnienie, brak zasobu, brak uprawnień, kara) |
| Hot spot | Miejsce w ES gdzie ktoś nie wie / coś jest sprzeczne / wymaga decyzji |
| Pivotal event | Zdarzenie, które wyraźnie zmienia stan biznesowy (np. "projekt rozpoczęty", "kara naliczona") |

## Mapa bounded contextów (*do wglądu orkiestratora, nie persony!*)

```
planning       — definiowanie projektu, fazy, etapy, krytyczny etap, zasoby
allocation     — kto/co/kiedy na którym etapie; cash flow; przenoszenie
availability   — kalendarz zasobów; urlopy
resource       — definicje zasobów (employee, device)
capability-scheduling — kalendarz umiejętności na osi czasu
risk           — saga okresowa, alerty o brakach, eskalacje
simulation     — "co jeśli" — wycena propozycji zmian
optimization   — algorytm rozłożenia etapów na fazy
shared         — typy współdzielone (TimeSlot, Capability, ResourceId)
```

Te konteksty NIE są jeszcze znane personom — to jest stan po modelowaniu.
Cel ES: dojść do (czegoś podobnego) tej mapy razem z facylitatorem.

## Kanoniczne zdarzenia (kotwice dla persony)

Big Picture, nieuporządkowane, mieszane konteksty:

- Nowy projekt dodany
- Zapotrzebowanie zarejestrowane
- Ryzyko wykryte
- Kierownik poinformowany
- Fazy zdefiniowane
- Fazy zrównoleglone
- Etapy zdefiniowane
- Zasób dodany do etapu
- Data startu projektu zapisana
- Daty etapów wyliczone
- Data końca projektu wyliczona
- Przydziały projektu utworzone
- Zasoby przydzielone
- Dostępność sprawdzona
- Zapotrzebowanie skolejkowane
- Zamiennik zasobu znaleziony
- Cotygodniowa polityka ryzyka uruchomiona
- Pracownik powiadomiony o znalezionym zamienniku
- Zasób dodany automatycznie
- Zarobek przeliczony na nowo
- Czas trwania etapu zmieniony
- Daty następnych etapów wyliczone na nowo
- Zasoby przeniesione
- Umiejętność zmieniona
- Umiejętność odblokowana
- Urlop dodany
- Zwolnienie chorobowe dodane
- Pracownik zatrudniony / zwolniony
- Kara umowna naliczona
- Eskalacja wywołana
- Zakres przycięty
- Brak zasobu wykryty
- Sprzęt dodany do projektu (np. koparka)

To są **kotwice**, nie odpowiedzi. Facylitator NIE pokazuje tej listy uczestnikom.
Persony przypominają sobie po kawałku, niektórzy się mylą, niektórzy nie wiedzą.

## Wątek sporny — nierdzewka na Strzegomskim (*do wglądu orkiestratora, nie persony!*)

Jedno zdarzenie, cztery różne pamięci. To **celowa rozbieżność** — materiał na
hot spot, nie błąd kanonu. Żadna persona nie zna wersji pozostałych.

| Kto | Co pamięta | Dlaczego tak |
|---|---|---|
| Prezes | Marek kogoś ściągnął z marszu, Mecenas zrobił mu pranie z papierów; szczegółów nikt mu nie dopowiedział | Nie chce wiedzieć — wiedza to odpowiedzialność |
| Ewa | Nikt u nich nie miał uprawnień, więc zatrudnili legalnie z marszu | Wersja "z systemu": tak to zostało zapisane |
| Marek | Opowiada to jako historię kolegi z innej firmy | Chroni swojego człowieka |
| Mecenas | Cytuje **inną** sprawę z 2022 u innego klienta jako przestrogę | Nie ujawnia własnego klienta |

Facylitator, który zderzy te wersje, dostaje hot spot o audytowalności
uprawnień. Facylitator, który tego nie zauważy, wyjdzie z sesji z czterema
niesprzecznymi na pozór zdaniami.

## Cykl życia projektu (*do wglądu orkiestratora, nie persony!*)

Kolejność, w której u tego klienta zwykle następują po sobie zdarzenia jednego
projektu:

```
Oferta → Kontrakt → Mobilizacja → Wykonanie → Odbiór → Płatność → Rękojmia/Gwarancja
```

Orkiestrator używa jej, gdy szuka miejsca dla świeżej kartki na osi czasu.
Facylitator tej listy nie pokazuje sali — kolejność ma wyjść z rozmowy, a nie
z podpowiedzi.

## Środowisko zewnętrzne (poza systemem — różowe karty)

- Inwestor / klient (płaci, naciska na terminy)
- Inspektor budowy / nadzór
- Urząd (uprawnienia, BHP)
- Dostawca sprzętu (leasing koparek)
- Bank / faktoring (cashflow)
- Pracownicy (zatrudnienie / zwolnienie)

## Co prezes mówi *między wierszami*

- "Chciałbym żeby system sam mi podpowiadał" — chce **proaktywności**, nie tylko zapisywania danych
- "Tańszy i lepszy niż gotowe rozwiązania" — odrzucił MS Project / Primavera
- "Trudnym zagnieżdżeniem jest ocena wpływu ryzyka" — chce **propagacji** ryzyka, nie tylko wykrywania
- "Najpierw chciałbym zobaczyć proponowane fazy" — chce **algorytm**, nie tylko widok
- "Muszę spadać" — typowy stakeholder, czas zawsze ucieka

To są sygnały które **dobry facylitator powinien wyłapać** — ale ich nie mówi
prezesowi wprost. Pyta dalej.
