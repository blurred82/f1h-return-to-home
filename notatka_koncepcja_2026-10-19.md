# Szybowiec holowany z automatycznym powrotem

Notatka z rozmowy, modelarnia lotnicza na Bródnie · 19.10.2026 · Paweł Nawrocki

Zadanie

dobrać elektronikę do istniejącego szybowca holowanego wzorowanego na F1H, tak aby sam wracał nad miejsce startu, gdy zacznie odlatywać albo zabraknie mu wysokości. Przepisy F1H zabraniają takiej automatyki (żyroskop, akcelerometr, GPS, kontroler lotu), więc będzie to model eksperymentalny do treningu, nie do zawodów. Model już jest, więc teraz chodzi tylko o dobór i montaż elektroniki. Poniżej opisuję koncepcję i proszę o opinię oraz wskazówki.

**Aktualizacja 20.10.2026** po uwagach instruktora modelarni: hol ręczny, powrót po zakończeniu czasu zadania, lądowanie bez krążenia nad startem, RC w czasie zadania tylko do awaryjnego DT, masa elektroniki ok. 30 g.

**Aktualizacja 21.10.2026:** zegar i kontroler działają osobno, a na końcu zadania zegar oddaje sterowanie jednym sygnałem. Kontroler stoi w komorze na krawędzi. Szczegóły w sekcjach Sprzęt, Integracja z zegarem i Oprogramowanie.

## Problem

Szybowce holowane ręcznie często odlatują daleko już przy słabym wietrze i giną. Najczęściej dzieje się to wtedy, gdy model wejdzie w komin i wiatr znosi go razem z noszeniem. Celem projektu jest elektronika, która w takiej sytuacji przejmie sterowanie i sprowadzi model nad start.

## Ograniczenia fizyczne

Powrót jest możliwy tylko wtedy, gdy model ma zapas wysokości i jest szybszy od wiatru. Z tego wynikają trzy założenia:

1. **Szybowiec nie utrzyma pułapu.** Bez noszeń opada ok. 0,4–0,6 m/s, w kominie się wznosi. Chodzi więc o lot swobodny przez czas zadania i powrót, zanim zabraknie wysokości.
2. **Model jest wolny.** Leci 4–6 m/s, więc przy wietrze 5 m/s pod wiatr praktycznie stoi. W trybie powrotu autopilot musi go rozpędzić do ok. 7–9 m/s kosztem większego opadania.
3. **Głównym wyzwalaczem jest czas.** Powrót zaczyna się po zakończeniu zadania: 60–120 s w F1H, 180–240 s w F1A. Odległość od startu i zapas wysokości to dodatkowe zabezpieczenia, które mogą uruchomić powrót wcześniej.

Przybliżony zapas wysokości potrzebny na powrót (d – odległość od startu, w_s – opadanie, v – prędkość lotu, w – wiatr czołowy):

```
H ≈ d × w_s / (v − w) + zapas
```

Przykład: 200 m od startu, wiatr 3 m/s, prędkość 7 m/s i opadanie 0,8 m/s dają ok. 40 m plus zapas.

## Logika lotu

![Logika lotu: 4 fazy i tryb awaryjny](logika_lotu.svg)

1. **Hol ręczny.** Stery w pozycji do holu, autopilot nieaktywny. → odczepienie (przełącznik haka)
2. **Zadanie: lot swobodny.** 60–120 s (F1H), stały trym jak w wolnym locie. Elektronika tylko mierzy, RC tylko do awaryjnego DT. → koniec czasu zadania, albo wcześniej: odległość > R lub brak zapasu wysokości
3. **Powrót (RTL).** Rozpędzenie do 7–9 m/s, lot nad start. Pilot RC może od tej chwili przejąć stery. → model nad startem
4. **Lądowanie.** Od razu po dolocie, bez krążenia nad startem.

**Awaria:** utrata GPS w fazie powrotu oznacza natychmiastowy DT.

Przez czas zadania model leci jak zwykły wolnolatający szybowiec, a RC służy tylko do awaryjnego DT. Autopilot włącza się po zakończeniu zadania albo wcześniej, gdy model przekroczy granice. Po dolocie nad start od razu ląduje, a przy utracie GPS w fazie powrotu uruchamia determalizator (DT).

## Sprzęt

Docelowa masa elektroniki to ok. 30 g, a 45 g to już za dużo. Dla porównania istniejący w modelarni system (zegar z serwami, RDT, przełącznik haka i zasilanie) waży 12–15 g. Czy zmieści się w modelu, zależy od jego obecnej masy, miejsca w kadłubie i wyważenia, dlatego najpierw trzeba je zmierzyć. Lotki nie są potrzebne, wystarczy klasyczny układ wolnolatający: ster kierunku i statecznik.

| Element | Przykład | Masa (g) |
| --- | --- | --- |
| Kontroler lotu | płytka H743 (np. Matek H743-mini lub WLITE), H7 ze względu na skrypty Lua | 5–8 |
| GPS z kompasem | mały moduł u-blox M10 | 5–10 |
| Barometr | zwykle wbudowany w kontroler | – |
| 2 serwa | ster kierunku i statecznik (wysokość i DT), micro 2–4 g | 5–8 |
| Odbiornik RC | ExpressLRS nano, do ręcznego przejęcia sterowania | 1–2 |
| Akumulator | 1S 300–450 mAh z małą przetwornicą 5 V | 10–15 |
| Przełącznik haka | mikroprzełącznik wykrywający odczepienie | 1 |

**Komora na elektronikę** (płoza kadłubowa) ma 200 mm długości, 55 mm wysokości i 18 mm szerokości. Hak, akumulator i serwa zajmują część, na kontroler zostaje 1/3–1/2 długości.

- **Kontroler (np. Matek H743-MINI, 36 × 28 × 6,5 mm) stoi na krawędzi**, wzdłuż kadłuba: ok. 10–12 mm szerokości z przewodami, 28 mm wysokości. Przewody lutowane do padów, bez wysokich wtyczek. Ułożenie płytki ustawia się w oprogramowaniu (`AHRS_ORIENTATION` w ArduPilocie). Mocowanie na miękkiej piance.
- **Antena GPS musi leżeć poziomo**, płaską stroną do góry, nie pod węglem ani metalem, z dala od serw i przewodów zasilania. W komorze zmieści się moduł o boku do ok. 15 mm. Większy moduł z kompasem idzie na grzbiet kadłuba lub na skrzydło.
- Kompas może okazać się zbędny, bo ArduPilot w samolotach ustala kierunek z GPS. Przy wolnym szybowcu i silnym wietrze kompas pomaga, więc to do przetestowania.

## Integracja z zegarem

Największy problem to połączenie z istniejącym zegarem. Dziś serwa są wpięte w zegar, a kontroler też musi nimi sterować w fazie powrotu. Zegar ma wolne wyjścia, które mogą uruchomić kontroler.

| Wariant | Jak działa | Plusy | Minusy |
| --- | --- | --- | --- |
| 1. Kontroler przejmuje funkcje zegara | Kontroler sam ustawia stery do holu i lotu swobodnego, odlicza czas i odpala DT | najlżejszy, jeden układ, jeden program | trzeba odtworzyć w kontrolerze wszystko, co robi zegar |
| 2. Multiplekser serw | W czasie zadania serwami steruje zegar. Po czasie zegar daje sygnał z wolnego wyjścia i kontroler przejmuje serwa | zegar bez zmian, szybki prototyp | kilka gramów więcej i dodatkowe połączenia |
| 3. Zegar jako źródło sygnału | Wyjścia zegara wchodzą do kontrolera, który przekazuje je do serw, a potem przejmuje sterowanie | jedna ścieżka do serw | najbardziej skomplikowany |

Wariant 2 nadaje się na szybki prototyp z obecnym zegarem, wariant 1 na rozwiązanie docelowe. Do omówienia z elektronikiem modelarni: jak zrealizować dwa programy, czyli (1) hol i lot swobodny oraz (2) tryb automatyczny.

### Ustalenia z 21.10.2026

**Zegar modelarni:** mikrokontroler ATtiny1634 (16 KB pamięci programu, 1 KB RAM, 256 B EEPROM, 2 porty szeregowe, 4 kanały PWM). Wejścia: hak i RDT (zwierane do minusa), potencjometr. Wyjścia: jedno serwo (wysokość), buzzer i oświetlenie (przez tranzystory), 6-pinowe złącze wyjść programowalnych, 4-pinowe złącze wprowadzania danych (Bluetooth). Zegar ma 3 programy, jeden zostanie przeznaczony na współpracę z kontrolerem. Kod źródłowy ma elektronik modelarni, który go napisał i może zmieniać.

**Nastawy zegara:** 6 położeń serwa wysokości (PWM), każde z własnym czasem trwania. Zmienia się je na treningu przez moduł Bluetooth podpinany na chwilę do zegara i smartfon.

**Ster kierunku** dziś nie ma serwa: ustawia go hak holowniczy (prosto na holu, po wyczepieniu i sekwencji bunt odchylenie do krążenia). Elektronik dopisze do zegara drugie serwo na kierunek.

**Decyzja: zegar i kontroler działają osobno, kod zegara nie jest potrzebny.** Normalnie zegar po zadaniu odpala DT sterem wysokości. W programie współpracy zamiast tego zawsze oddaje sterowanie kontrolerowi i na tym kończy pracę. Kontroler decyduje: powrót albo od razu DT.

| Cecha sygnału przekazania | Propozycja |
| --- | --- |
| Rodzaj | stan logiczny na jednym wolnym wyjściu programowalnym, bez PWM |
| Znaczenie | niski = steruje zegar, wysoki = steruje kontroler |
| Kiedy | zegar ustawia stan wysoki zamiast odpalić DT i trzyma go do końca lotu |
| Po stronie kontrolera | wejście z rezystorem do masy, więc odłączony przewód oznacza „steruje zegar” |
| Napięcie | przy zegarze na 5 V dzielnik rezystorowy (np. 10 kΩ i 20 kΩ) do 3,3 V |
| Zatrzask | kontroler przejmuje sterowanie raz i już go nie oddaje |

**Przełączanie serw:** 2-kanałowy multiplekser (wysokość i kierunek) sterowany tym samym sygnałem. W czasie zadania kontroler fizycznie nie może ruszyć sterami, co upraszcza zgodność z przepisami sportowymi. Możliwa realizacja: układ 74LVC1G3157 na każdy kanał albo gotowy multiplekser serw (masę trzeba sprawdzić).

**Na przyszłość:** drugi przewód w odwrotną stronę. Kontroler zwiera do minusa wejście zegara, gdy model odlatuje za daleko lub traci wysokość, a zegar od razu oddaje sterowanie. Na start niepotrzebne.

## Oprogramowanie

Planuję użyć ArduPilota (ArduPlane) z własnym skryptem Lua. Daje on więcej swobody niż INAV, który jest prostszy w konfiguracji, ale słabiej się rozszerza.

- `RUDDER_ONLY = 1` pozwala sterować samym sterem kierunku, bez lotek.
- Wbudowane są geofence z akcją powrotu (RTL), lądowanie, lot bez czujnika prędkości i parametry szybowca. Krążenie nad startem nie jest potrzebne: po dolocie model od razu ląduje.
- Skrypt Lua wykrywa odczepienie, odlicza czas zadania, pilnuje granic (odległość, wysokość), włącza powrót i lądowanie, a w razie potrzeby DT.
- W czasie zadania skrypt ignoruje sterowanie z RC poza awaryjnym DT. Przejęcie sterów odblokowuje dopiero faza powrotu.

**Wariant uproszczony na początek:** automatyczny DT z geofence. Wystarczą ESP32 lub Arduino, GPS i jedno serwo. Po przekroczeniu np. 150 m od startu albo zadanej wysokości determalizator sprowadza model w pobliżu startu, choć nie dokładnie na nie. Takie rozwiązanie jest tanie, lekkie i niezawodne.

**Trymowanie z RC na treningu.** Model oblatuje się i trymuje z RC, a system zapamiętuje położenia sterów dla holu i lotu swobodnego. Potem odbiornik można wyjąć i model lata na zapisanych nastawach. Podobnie działa już rosyjski zegar zintegrowany z RC. Bardzo przyspiesza to oblatywanie i testy.

Gotowy autotrim w INAV (tryb AUTO TRIM, ciągły `servo_autotrim`) i ArduPilocie (`SERVO_AUTO_TRIM`) robi coś innego niż opisany wyżej system:

| | Autotrim w INAV / ArduPilocie | Zapamiętywanie nastaw z RC |
| --- | --- | --- |
| Cel | lot prosto i poziomo, bez stałego wychylenia drążka | nastawy lotu swobodnego, np. krążenie ze stałym wychyleniem steru kierunku |
| Jak działa | w locie ze stabilizacją kontroler przesuwa środki serw | pilot ustawia stery z RC, system zapisuje ich położenia |
| Fazy lotu | jedna nastawa | osobne nastawy dla holu i lotu swobodnego |
| Zastosowanie | faza powrotu | faza zadania |

- Autotrim przyda się do powrotu: model raz wytrymowany do lotu prostego autopilot lepiej prowadzi nad start.
- Do nastaw lotu swobodnego się nie nadaje, bo wyprowadzi model do lotu prostego i skasuje krążenie.
- W INAV nastawy holu i lotu swobodnego da się zrobić półręcznie: po oblataniu z RC odczytać położenia serw w konfiguratorze lub z telemetrii i wpisać je jako stałe wartości w miksie albo w Programming Framework. Automatyczny zapis jednym przełącznikiem wymaga skryptu Lua w ArduPilocie.
- Nazwy i działanie opcji zależą od wersji oprogramowania, więc trzeba je sprawdzić w dokumentacji używanej wersji.

**Zmiany nastaw na polu.** W kontrolerze sekwencja zegara to 12 parametrów skryptu Lua o czytelnych nazwach, np. `FF_P1`…`FF_P6` (położenia) i `FF_T1`…`FF_T6` (czasy), oraz czas zadania.

| Sposób | Zalety | Wady |
| --- | --- | --- |
| Nadajnik RC w locie, zapis przełącznikiem | najszybszy, efekt widać od razu, bez lądowania | tylko na treningu, trzeba napisać zapis nastaw |
| Strona w przeglądarce telefonu (ESP32, Wi-Fi) | każdy telefon, bez instalowania, najbliżej dzisiejszego Bluetooth | trzeba napisać stronę |
| QGroundControl przez Bluetooth (ArduPilot) | działa od razu, darmowy | tylko Android (na iOS tylko Wi-Fi), rozbudowany |
| SpeedyBee App (INAV) | darmowy, Android i iOS | nie wiadomo, czy pokaże wszystkie nastawy |

Proponowana kolejność: na start QGroundControl, przy oblatywaniu trymowanie z RC, docelowo dla klubu strona na ESP32.

## Przepisy i bezpieczeństwo

Z autopilotem model staje się w świetle prawa UE bezzałogowym statkiem powietrznym. Jeśli waży poniżej 250 g i nie ma kamery, mieści się w kategorii otwartej.

- Model startuje jako swobodnie latający. Automatyczne sterowanie w oparciu o GPS działa dopiero w fazie powrotu i od tego momentu pilot musi móc przejąć stery z RC, w zasięgu wzroku.
- Przepisy sportowe zabraniają użycia RC do czegokolwiek poza awaryjnym odpaleniem DT, więc w czasie zadania zawodnik nie może włączyć sterowania z RC.
- Czy taki podział spełnia przepisy o bezzałogowcach, trzeba potwierdzić w ULC. Tam też należy sprawdzić wymagania co do rejestracji operatora.
- Testy stopniowe: najpierw lot ręczny na RC, potem samo RTL z ręki, na końcu pełny cykl z holu.
- Loty z dala od ludzi i lotnisk.

## Plan prac

1. Pomiar modelu: masa, środek ciężkości, miejsce w kadłubie, istniejące serwa, zegar i DT.
2. Wybór wariantu integracji z zegarem (wspólnie z elektronikiem modelarni).
3. Dobór elektroniki i bilans masy, cel ok. 30 g.
4. Montaż i uruchomienie kontrolera na stole: stery, kierunki wychyleń, failsafe, blokada RC w czasie zadania.
5. Loty ręczne na RC i trymowanie z elektroniką na pokładzie.
6. Test powrotu i lądowania przy słabym wietrze.
7. Skrypt z wyzwalaczem czasowym i dodatkowymi granicami.
8. Pełne loty z holu ręcznego.

## Pytania otwarte

- Które wyjście programowalne zegara da sygnał przekazania i jakim napięciem pracuje zegar (3,3 V czy 5 V)?
- Czy 6-pinowe złącze to wyjścia, czy złącze do programowania procesora?
- Multiplekser serw: własny układ elektronika czy gotowy moduł? Ile waży?
- Który moduł GPS: mały w komorze bez kompasu czy większy z kompasem na zewnątrz?
- Ile waży model i gdzie ma środek ciężkości? Czy udźwignie ok. 30 g elektroniki bez utraty osiągów?
