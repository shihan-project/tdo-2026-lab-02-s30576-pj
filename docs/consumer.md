# Konsument CI

Inspect CI akceptuje udany push/main z tego repozytorium i potwierdza ID definicji ci.yml. Pobiera artefakt przez ID i attempt producenta, a następnie sprawdza schemat, SHA i oba identyfikatory.

Ręczne tryby valid/invalid na gałęzi testowej uruchamiają ten sam walidator na fixture, bez pobierania artefaktu. Wspólny krok jest zapisany jako kotwica YAML.
