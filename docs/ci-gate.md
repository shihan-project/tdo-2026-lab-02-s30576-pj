# Warunkowe CI

## Wspólna nazwa required

PR #20 był próbą porównawczą, bez merge. Pozostałe wymagane checki aplikacji były zielone.

| Próba | Wyniki required | Merge | Commit |
| --- | --- | --- | --- |
| zero |  | BLOCKED | [6d15f90](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/commit/6d15f90c3bef5a009a6b47a484f3f87c125eb106) |
| one | success | CLEAN | [d578d50](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/commit/d578d509f795d4e4b7447437a670412301a8258c) |
| two-pending | pending + success | BLOCKED | [add3dc3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/commit/add3dc34f36ea4b20e93e3fa251c5e5dc471a835) |
| two-success | success + success | CLEAN | [add3dc3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/commit/add3dc34f36ea4b20e93e3fa251c5e5dc471a835) |
| two-failure | failure + success | BLOCKED | [7cf4430](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/commit/7cf443048eb6f5213fc66fdb350051a65b1720fc) |
| two-fixed | success + success | CLEAN | [ab9cdb5](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/commit/ab9cdb5edec258dd722635b4b93b5215268edc9c) |

Dane: [porównanie](evidence/required-comparison-stage19.json). Tymczasowe workflow usunięto po próbie.

## Gate

Detekcja musi się udać, a flagi mają mieć wartość true lub false. Wymagany komponent musi zakończyć się success; niewymagany musi być skipped. Pozostałe kombinacje odrzucane.

Przed wdrożeniem sprawdzono [11 przypadków kontraktu](evidence/gate-local-contract.json), w tym cancelled, brak flagi i nieoczekiwane pominięcie.


## Proby CI

| Próba | Backend / frontend | Gate | Run |
| --- | --- | --- | --- |
| both | success / success | success | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42298904565) |
| zero | skipped / skipped | success | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42299333534) |
| backend | success / skipped | success | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42299412336) |
| frontend | skipped / success | success | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42299616269) |
| regression | failure / skipped | failure | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42299768940) |
| detector-failure | skipped / skipped | failure | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42299952227) |
| missing-flag | success / skipped | failure | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42300034695) |
| unexpected-skip | skipped / success | failure | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42300271809) |
| only-gate-blocked | skipped / success | failure | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42300271809) |

W trakcie prób wymagane były również poprzednie nazwy checków. Po ich zakończeniu jedynym wymaganym wynikiem został ci-required. Czerwony gate sprawdzono także przy tej końcowej regule. Po usunięciu błędów końcowy run docs-only zapisano w dzienniku README.

[Aktywna ochrona](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/rules/26724075): PR, strict, ci-required, bez bypass. Reguły Classroom zachowane.
