# Macierze backendu

`matrix.yml` uruchamiamy ręcznie. Każdy wariant wykonuje locked restore, build i 19 testów API z MySQL Testcontainers. Summary akceptuje wyłącznie success wszystkich swoich grup. Te jobs nie są required checks PR-a.

## Zadanie 22

| Próba | SHA | Run | Wynik |
| --- | --- | --- | --- |
| Debug i Release | `27d6f55` | [42310362077](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42310362077) | Oba success, po 19 testów i osobny TRX. |
| Błąd Debug po testach | `804882a` | [42310847797](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42310847797) | Debug failure, Release success, summary failure; oba raporty dostępne. |
| Poprawka i include/exclude | `a8d2b62` | [42311007461](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42311007461) | Trzy warianty success, każdy 19 testów. |

Po exclude nie ma Debug/bez raportu. Pozostają Debug/z raportem oraz Release/z raportem i bez raportu. Diagnostyka działa tylko dla Release/z raportem. Wyłączenie uploadu nie pomija testów; pobrano dwa odrębne TRX, oba 19/19.

## Zadanie 23

`include-debug` wybiera jeden lub dwa warianty dynamiczne. Job Plan waliduje tekst inputu, liczbę wpisów, pary konfiguracja/lane i unikalność lane przed zapisaniem outputu JSON. Release jest zawsze obecny. Grupa statyczna pozostaje do porównania.

| Próba | SHA | Run | Wynik |
| --- | --- | --- | --- |
| main, false | `677106e` | [42311973480](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42311973480) | Release; summary success, 3 raporty. |
| main, true | `677106e` | [42311977719](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42311977719) | Release i Debug; summary success, 4 raporty. |
| Preview | `d3ec268` | [42312383775](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42312383775) | Plan failure przed outputem, dynamic skipped, summary failure. |
| Błąd Debug | `dccb725` | [42312638645](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42312638645) | Dynamic Debug failure, Release success; summary failure, 4 raporty. |
| Poprawka, false | `1beca31` | [42312879732](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42312879732) | Release; summary success, 3 raporty. |
| Poprawka, true | `1beca31` | [42312884008](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42312884008) | Release i Debug; summary success, 4 raporty. |

Każdy uruchomiony wariant zaliczył 19 testów; kontrolowany błąd Debug następował po testach. Statyczna grupa ma zawsze trzy warianty i dwa raporty, dynamiczna jeden lub dwa warianty z osobnymi raportami. Pobrane TRX potwierdzają 19/19.

Lokalnie sprawdzono 10 przypadków dokładnego kodu walidatora, m.in. niepoprawny input, brak Release, powtórzone lane i niezgodną parę. To próby funkcji walidującej; powyższa tabela zawiera rzeczywiste runy GitHub Actions.

Po usunięciu błędów drzewo gałęzi ćwiczeniowej jest identyczne z main. PR #30 zamknięto bez merge; działająca definicja jest na main po PR #29. Wymagana ochrona PR pozostała bez zmian.
