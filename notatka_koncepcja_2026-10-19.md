# Szybowiec holowany z automatycznym powrotem

Notatka z rozmowy, modelarnia lotnicza na Bródnie · 19.10.2026 · Paweł Nawrocki

Zadanie

dobrać elektronikę do istniejącego szybowca holowanego wzorowanego na F1H, tak aby sam wracał nad miejsce startu, gdy zacznie odlatywać albo zabraknie mu wysokości. Przepisy F1H zabraniają takiej automatyki (żyroskop, akcelerometr, GPS, kontroler lotu), więc będzie to model eksperymentalny do treningu, nie do zawodów. Model już jest, więc teraz chodzi tylko o dobór i montaż elektroniki. Poniżej opisuję koncepcję i proszę o opinię oraz wskazówki.

## Problem

Szybowce wyciągane wyciągarką często odlatują daleko już przy słabym wietrze i giną. Najczęściej dzieje się to wtedy, gdy model wejdzie w komin i wiatr znosi go razem z noszeniem. Celem projektu jest elektronika, która w takiej sytuacji przejmie sterowanie i sprowadzi model nad start.

## Ograniczenia fizyczne

Powrót jest możliwy tylko wtedy, gdy model ma zapas wysokości i jest szybszy od wiatru. Z tego wynikają trzy założenia:

1. **Szybowiec nie utrzyma pułapu.** Bez noszeń opada ok. 0,4–0,6 m/s, w kominie się wznosi. Chodzi więc o lot swobodny, dopóki są warunki, i powrót, zanim zabraknie wysokości.
2. **Model jest wolny.** Leci 4–6 m/s, więc przy wietrze 5 m/s pod wiatr praktycznie stoi. W trybie powrotu autopilot musi go rozpędzić do ok. 7–9 m/s kosztem większego opadania.
3. **Sama wysokość to zły wyzwalacz.** Gdy model spadnie poniżej progu, może być już za daleko. Powrót powinna uruchamiać odległość albo brak zapasu wysokości, w zależności od tego, co nastąpi pierwsze.

Przybliżony zapas wysokości potrzebny na powrót (d – odległość od startu, w_s – opadanie, v – prędkość lotu, w – wiatr czołowy):

```
H ≈ d × w_s / (v − w) + zapas
```

Przykład: 200 m od startu, wiatr 3 m/s, prędkość 7 m/s i opadanie 0,8 m/s dają ok. 40 m plus zapas.

## Logika lotu

![Logika lotu: 4 fazy i tryb awaryjny](logika_lotu.svg)

1. **Hol na wyciągarce.** Stery w pozycji do holu, autopilot nieaktywny. → odczepienie (przełącznik haka)
2. **Lot swobodny.** Stały trym, krążenie jak w wolnym locie, elektronika tylko mierzy. → odległość > R albo brak zapasu wysokości, albo koniec czasu, albo za wysoko
3. **Powrót (RTL).** Rozpędzenie do 7–9 m/s, lot nad start, krążenie nad startem. → nad startem, poniżej ok. 15 m
4. **Lądowanie lub DT.** Przechylenie statecznika albo krążenie do ziemi.

**Awaria:** utrata GPS w dowolnej fazie oznacza natychmiastowy DT. Pilot RC może przejąć sterowanie zawsze.

## Sprzęt

Cała elektronika waży ok. 30–45 g. Czy zmieści się w modelu, zależy od jego obecnej masy, miejsca w kadłubie i wyważenia, dlatego najpierw trzeba je zmierzyć. Lotki nie są potrzebne, wystarczy klasyczny układ wolnolatający: ster kierunku i statecznik.

| Element | Przykład | Masa (g) |
| --- | --- | --- |
| Kontroler lotu | płytka H743 (np. Matek H743-mini lub WLITE), H7 ze względu na skrypty Lua | 5–8 |
| GPS z kompasem | mały moduł u-blox M10 | 5–10 |
| Barometr | zwykle wbudowany w kontroler | – |
| 2 serwa | ster kierunku i statecznik (wysokość i DT), micro 2–4 g | 5–8 |
| Odbiornik RC | ExpressLRS nano, do ręcznego przejęcia sterowania | 1–2 |
| Akumulator | 1S 300–450 mAh z małą przetwornicą 5 V | 10–15 |
| Przełącznik haka | mikroprzełącznik wykrywający odczepienie | 1 |

## Oprogramowanie

Planuję użyć ArduPilota (ArduPlane) z własnym skryptem Lua. Daje on więcej swobody niż INAV, który jest prostszy w konfiguracji, ale słabiej się rozszerza.

- `RUDDER_ONLY = 1` pozwala sterować samym sterem kierunku, bez lotek.
- Wbudowane są geofence z akcją powrotu (RTL), krążenie nad startem (`RTL_RADIUS`), lot bez czujnika prędkości i parametry szybowca.
- Skrypt Lua wykrywa odczepienie, liczy potrzebny zapas wysokości (ArduPilot sam szacuje wiatr), włącza powrót i na końcu uruchamia DT.

**Wariant uproszczony na początek:** automatyczny DT z geofence. Wystarczą ESP32 lub Arduino, GPS i jedno serwo. Po przekroczeniu np. 150 m od startu albo zadanej wysokości determalizator sprowadza model w pobliżu startu, choć nie dokładnie na nie. Takie rozwiązanie jest tanie, lekkie i niezawodne.

## Przepisy i bezpieczeństwo

Z autopilotem model staje się w świetle prawa UE bezzałogowym statkiem powietrznym. Jeśli waży poniżej 250 g i nie ma kamery, mieści się w kategorii otwartej.

- Lot w zasięgu wzroku, a pilot musi móc przejąć sterowanie, dlatego odbiornik RC jest konieczny.
- Wymagania ULC co do rejestracji operatora trzeba sprawdzić przed pierwszym lotem.
- Testy stopniowe: najpierw lot ręczny na RC, potem samo RTL z ręki, na końcu pełny cykl z wyciągarką.
- Loty z dala od ludzi i lotnisk.

## Plan prac

1. Pomiar modelu: masa, środek ciężkości, miejsce w kadłubie, istniejące serwa, timer i DT.
2. Dobór elektroniki i bilans masy pod ten model.
3. Montaż i uruchomienie ArduPilota na stole: stery, kierunki wychyleń, failsafe.
4. Loty ręczne na RC i trymowanie z elektroniką na pokładzie.
5. Test powrotu i geofence przy słabym wietrze.
6. Skrypt Lua z wyliczaniem zapasu wysokości.
7. Pełne loty z wyciągarką.

## Pytania otwarte

- Ile waży model, gdzie ma środek ciężkości i ile wolnego miejsca jest w kadłubie?
- Czy ma już timer, DT albo serwa, które można wykorzystać lub trzeba zdemontować?
- Czy model udźwignie dodatkowe 30–45 g bez utraty osiągów, czy trzeba szukać lżejszych podzespołów?
- Czy zacząć od wariantu uproszczonego (auto-DT), a pełny powrót zrobić w drugim etapie?
