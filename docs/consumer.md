# Konsument CI

Inspect CI akceptuje udany push/main z tego repozytorium i potwierdza ID definicji ci.yml. Pobiera artefakt przez ID i attempt producenta, a następnie sprawdza schemat, SHA i oba identyfikatory.

Ręczne tryby valid/invalid na gałęzi testowej uruchamiają ten sam walidator na fixture, bez pobierania artefaktu. Wspólny krok jest zapisany jako kotwica YAML.

| Przypadek | Producent | Konsument / test | Wynik |
| --- | --- | --- | --- |
| push/main | [42302198517](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302198517) | [42302390876](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302390876) | SHA, ID i attempt zgodne |
| PR | [42301959269](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42301959269) | [42302148842](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302148842) | inspect skipped |
| anulowany dispatch | [42302441612](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302441612) | [42302478123](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302478123) | inspect skipped |
| fixture valid | — | [42302432908](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302432908) | success |
| fixture invalid | — | [42302437278](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302437278) | błąd schematu |
