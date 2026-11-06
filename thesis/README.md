# Raport LaTeX

Własny tekst: `engineer.tex`. Klasa `sprz` pochodzi z [pantadeusz/pjatk_latex](https://github.com/pantadeusz/pjatk_latex/tree/e660b9ee1e637a843c9084774abdc9b3ecdaae4c), rewizja `e660b9ee1e637a843c9084774abdc9b3ecdaae4c`.

Autor szablonu: Tadeusz Puźniakowski; współautorzy według README: Ian Canals-Wąsik i Mateusz Budzisz. Przypięta rewizja nie zawiera wyraźnej licencji. Klasa jest pobierana i kopiowana bez zmian wyłącznie do roboczego katalogu runnera. Raport zawiera własny tekst i tabelę wyników.

Budowanie w Bash po pobraniu przypiętego upstream do `upstream-latex/`:

```bash
cp upstream-latex/sprz.cls thesis/sprz.cls
cd thesis
rm -f engineer.pdf
latexmk -norc -pdf -interaction=nonstopmode -halt-on-error -file-line-error -no-shell-escape engineer.tex
test -s engineer.pdf
```

PDF jest artefaktem runa i assetem wydania. Prerelease dotyczy PR z tego repozytorium przy zgodnym z aktualnym main drzewie `.github/workflows`; push/main tworzy pełne wydanie. Tag zawiera ID i attempt runa, a jego cel pochodzi ze zdarzenia.

## Sprawdzenie publikacji

| Próba | Wynik | Link |
| --- | --- | --- |
| PR zmieniający workflow | PDF, publikacja skipped | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42304629777) |
| PR z forka | PDF, oba joby publikacji skipped | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42306978724) |
| TeX, wersja 1 | prerelease z PDF | [wydanie](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/releases/tag/thesis-pr-27-run-42307797189-attempt-1) |
| Błąd TeX | failure, 0 artefaktów, brak wydania | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42308395878) |
| TeX, wersja 2 | nowy prerelease z PDF | [wydanie](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/releases/tag/thesis-pr-27-run-42308667940-attempt-1) |
| main po merge | pełne wydanie z PDF | [wydanie](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/releases/tag/thesis-v1.42309136735.1) |

Pobrane PDF-y otwierają się i mają jedną stronę; układ sprawdzono na renderach. Cele tagów potwierdzono przez API Git refs. Warunki dla autora Dependabot przy rerun właściciela, starego drzewa workflow i błędów API sprawdzono testem funkcji policy.
