# Lab 02 — plan i review

Zakres: zadania 1–23 z [instrukcji prowadzącego](https://pjmpr.github.io/TDO-lecures/labs/ci/), rewizja materiałów `442bb25`. Po każdym etapie sprawdzono konfigurację, logi i wymagane wyniki przed następnym zadaniem. Linki i SHA są w [dzienniku](../README.md#dziennik).

- [x] 1. Pierwszy workflow z dwoma własnymi krokami.
- [x] 2. Checkout i runtime z plików projektu.
- [x] 3. PR do main i osobny run po merge.
- [x] 4. Ręczne uruchomienie z inputami i diagnostyką.
- [x] 5. Kontrolowany błąd i poprawka nowym commitem.
- [x] 6. Trzy lockfiles wygenerowane lokalnie.
- [x] 7. CI backendu: locked restore, build i testy w wybranej konfiguracji.
- [x] 8. Frontend jako niezależny, równoległy job z buildem i testami.
- [x] 9. Regresja walidacji API i niespójny lockfile, osobne błędy i poprawki.
- [x] 10. Raporty dostępne także po nieudanym teście.
- [x] 11. Variable, fikcyjny sekret i minimalne uprawnienia.
- [x] 12. Ochrona main: odrzucony push i blokada błędnego PR-a.
- [x] 13. Pomiar cache cold/warm i zmiany klucza.
- [x] 14. Step outputs i GITHUB_ENV.
- [x] 15. Job outputs i pobranie raportów na nowym runnerze.
- [x] 16. Paginacja zmian PR-a, rename i jeden aktualizowany komentarz.
- [x] 17. Composite action z inputem, outputem i propagacją błędów.
- [x] 18. Reusable workflow z walidacją kontraktu.
- [x] 19. Unikalny wymagany gate dla 0/1/2 komponentów.
- [x] 20. Osobny konsument workflow_run i walidacja metadanych.
- [x] 21. Własny PDF, przypięta klasa, prerelease i pełne wydanie.
- [x] 22. Statyczna macierz, błąd Debug i include/exclude.
- [x] 23. Plan JSON, false/true, błędny plan i błąd wariantu, poprawki.

Kontrola końcowa: testy API 19/19 i UI 11/11, zielone CI main, konsument, PDF oraz ręczne macierze. Main nadal wymaga wyłącznie `ci-required`; manualne jobs nie są wymaganymi checks. Testy i walidacja aplikacji pozostają przywrócone.

- [ ] Przegląd końcowych zmian przez właściciela repozytorium.
- [ ] Ostatni commit, PR i ponowne sprawdzenie main po akceptacji.
- [ ] Formalne oddanie zadania.
