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
