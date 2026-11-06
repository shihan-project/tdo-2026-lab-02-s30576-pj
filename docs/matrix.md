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
