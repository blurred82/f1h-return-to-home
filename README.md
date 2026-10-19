# Szybowiec F1H z automatycznym powrotem

Eksperymentalny szybowiec holowany wzorowany na klasie F1H, który sam wraca nad miejsce startu, gdy zaczyna odlatywać albo kończy mu się zapas wysokości.

Przepisy F1H zabraniają automatyki (żyroskop, akcelerometr, GPS, kontroler lotu), więc to model treningowy i badawczy, nie do zawodów.

## Założenia

- Hol na wyciągarce, potem lot swobodny ze stałym trymem, jak w klasycznym wolnym locie.
- Elektronika przejmuje sterowanie, gdy model przekroczy promień od startu, zabraknie mu zapasu wysokości na powrót, minie zadany czas albo wejdzie za wysoko.
- Powrót nad start (RTL), krążenie, a na końcu lądowanie lub determalizator (DT).
- Przy utracie GPS natychmiastowy DT. Pilot RC może przejąć sterowanie w każdej fazie.

Zapas wysokości potrzebny na powrót:

```
H ≈ d × w_s / (v − w) + zapas
```

gdzie d to odległość od startu, w_s opadanie, v prędkość lotu, a w wiatr czołowy.

## Sprzęt (wstępnie)

- Kontroler lotu H743 z ArduPilotem (ArduPlane, `RUDDER_ONLY = 1`) i skryptem Lua.
- GPS u-blox M10 z kompasem, barometr na kontrolerze.
- 2 serwa micro (ster kierunku i statecznik z DT), odbiornik ExpressLRS, akumulator 1S.
- Mikroprzełącznik na haku do wykrywania odczepienia.

Masa elektroniki to ok. 30–45 g.

Wariant uproszczony na początek: automatyczny DT z geofence (ESP32 lub Arduino, GPS i jedno serwo).

## Pliki

- [notatka_koncepcja_2026-10-19.md](notatka_koncepcja_2026-10-19.md): pełny opis koncepcji, fizyki, logiki lotu, przepisów i planu prac.

## Bezpieczeństwo i przepisy

Model z autopilotem jest w świetle prawa UE bezzałogowym statkiem powietrznym (kategoria otwarta, poniżej 250 g). Lataj tylko w zasięgu wzroku, z możliwością ręcznego przejęcia sterowania i z dala od ludzi i lotnisk.
