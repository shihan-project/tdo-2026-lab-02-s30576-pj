# Laboratorium 02 — GitHub Actions

Mission Control: .NET 10, Angular 21 i MySQL. Rozwiązanie zadań z [laboratorium CI](https://pjmpr.github.io/TDO-lecures/labs/ci/).

## Aplikacja Mission Control

Aplikacja .NET + Angular z bazą MySQL i gotowymi testami. Zadaniem studenta jest
wygenerowanie lockfiles, utworzenie workflow GitHub Actions od zera, dodanie cache zależności
i zabezpieczenie repozytorium.
Nie trzeba pisać testów ani konfigurować frameworków testowych.

Wersja z zajęć nr 1 (Spring Boot + PostgreSQL) jest zachowana pod tagiem `Z1` w repozytorium
materiałów kursu. Tag nie jest automatycznie kopiowany do nowych repozytoriów GitHub Classroom.

## Wymagania

- .NET SDK 10.0.401 lub nowsza poprawka w tej samej serii 10.0.4xx; wersję określa `global.json`.
- Node.js 26.1.0 i npm; wersję Node określa `.nvmrc`.
- Git i działający Docker Engine z obsługą kontenerów Linux.
- Dla Rancher Desktop: silnik **dockerd (moby)**, nie containerd. Kubernetes nie jest potrzebny.
- Internet do pobrania paczek i obrazów przy pierwszym uruchomieniu.

Testy backendu korzystają z Testcontainers: same uruchamiają odizolowany MySQL 8.4 na losowym
porcie i usuwają go po zakończeniu. Nie wymagają własnej bazy, `.env` ani prawdziwych sekretów.
Pierwsze uruchomienie może potrwać dłużej przez pobieranie obrazów. Brak działającego Dockera
powoduje błąd testów, nie ich pominięcie.

## Moduły

- `backend/` — ASP.NET Core 10, Entity Framework Core, REST API misji.
- `backend.tests/` — testy HTTP API: CRUD, walidacja, daty, brakujące rekordy, healthcheck i OpenAPI.
- `frontend/` — Angular 21; testy interfejsu z symulowanymi odpowiedziami HTTP, bez uruchamiania API.
- `MissionControl.slnx` — rozwiązanie zawierające API i jego testy.
- `docs/api.md` — kontrakt API.

## Model pracy

- Stabilny `main`, krótkie gałęzie `feature/*` i PR-y kierowane do main — lekki model w duchu GitHub Flow.
- Jedna niezależna zmiana na gałąź; CI i review przed scaleniem, potem usunięcie zakończonej gałęzi.
- Nie tworzymy `develop`, `release/*` ani osobnego procesu Git Flow na potrzeby tego ćwiczenia.
- Filtr `pull_request.branches` dotyczy main jako bazy PR-a, a nie nazwy gałęzi feature.

## 1. Sprawdzenie środowiska

Polecenia wykonuj w katalogu tego startera:

```bash
dotnet --version
node --version
npm --version
docker version
docker info
```

`docker version` musi pokazać klienta i działający serwer.
`NuGet.Config` wskazuje publiczny rejestr NuGet, niezależnie od prywatnych źródeł skonfigurowanych na komputerze.

## 2. Aktualizacja lockfiles

Trzy lockfiles są w repozytorium. Po zmianie zależności zaktualizuj je lokalnie i przejrzyj różnice.

```bash
dotnet restore MissionControl.slnx --use-lock-file
cd frontend
npm install
cd ..
```

Powstaną trzy pliki: `backend/packages.lock.json`, `backend.tests/packages.lock.json`
oraz `frontend/package-lock.json`. Dodaj je do Gita i zapisz w commicie.

## 3. Powtarzalny build i gotowe testy

Po wygenerowaniu lockfiles:

```bash
dotnet restore MissionControl.slnx --locked-mode
dotnet build MissionControl.slnx --configuration Release
dotnet test MissionControl.slnx --configuration Release
cd frontend
npm ci
npm run build
npm test
cd ..
```

Raporty: `TestResults/backend.trx`, `frontend/test-results/junit.xml`
i `frontend/test-results/results.json`. To pliki wynikowe do udostępnienia jako artefakty CI,
nie do commitowania. Nieudany test musi zachować niezerowy kod zakończenia.
Projekt testowy wskazuje `backend.tests/test.runsettings` przez `RunSettingsFilePath`.
Plik ustawia logger TRX i katalog wyników względem swojego położenia, więc zwykłe
`dotnet test MissionControl.slnx --configuration Release` zapisuje raport bez dodatkowych flag.
Gotowe zestawy obejmują 19 testów backendu i 11 testów frontendu.

W CI najpierw sprawdź obecność **wszystkich trzech** lockfiles, następnie użyj `npm ci`
i `dotnet restore --locked-mode`. CI nie może generować brakujących lockfiles ani aktualizować ich za studenta.
GitHub Actions ustawia zmienną środowiskową `CI=true`. Wtedy `Directory.Build.props`
włącza `RestoreLockedMode` także dla implicit restore wykonywanego przez zwykłe `dotnet build`
i `dotnet test`; nie trzeba dopisywać `--no-restore` ani `--no-build`.
`Directory.Build.targets` dodatkowo przerywa restore/build, jeśli projektowi brakuje
`packages.lock.json`, zamiast pozwolić NuGetowi utworzyć go w CI.
Przy odtwarzaniu CI lokalnie lub w innym systemie ustaw `CI=true`; bez tej zmiennej
początkowe `dotnet restore --use-lock-file` pozostaje zadaniem studenta.
Kontrola w MSBuild dotyczy tylko projektów .NET — workflow nadal musi sprawdzać także
`frontend/package-lock.json` przed instalacją i odczytem cache.
Job backendu musi mieć dostęp do działającego Dockera; wystarczy standardowy runner GitHub-hosted Ubuntu.
Testcontainers zarządza MySQL, więc nie trzeba dodatkowo deklarować serwisu MySQL w workflow.

Zmiana deklaracji zależności wymaga świadomego ponownego wygenerowania lockfiles,
przejrzenia różnic i commita. Zmiana zależności bez aktualizacji lockfile powinna dać czerwone CI.

### Kolejny etap: cache zależności

Najpierw doprowadź workflow do zielonego wyniku bez cache. Dopiero wtedy porównuj optymalizację:

- Cache pobranych paczek npm lub NuGet; nie zapisuj `node_modules`, `bin`, `obj` ani raportów testów.
- Sprawdź lockfiles **przed** utworzeniem klucza; uwzględnij runtime, system/architekturę i hash zależności.
- Wybierz `actions/cache` albo cache udostępniany przez `setup-node`/`setup-dotnet`, bez podwójnego cache tej samej ścieżki.
- Wykonaj dwa uruchomienia tego samego kodu i porównaj logi, `cache-hit` oraz czas instalacji.
- Zmień zależność wraz z lockfile, sprawdź nowy klucz i uruchomienie z zimnym cache.
- Nie pomijaj `npm ci`, `dotnet restore --locked-mode`, buildu ani testów po cache hit.

`cache-hit` jest tekstowym outputem action; dokładne trafienie sprawdzaj przez porównanie z `'true'`.
Cache może zniknąć lub być niedostępny — aplikacja musi się budować i przechodzić testy również bez niego.
Nie zapisuj sekretów, `.env`, `.npmrc` ani konfiguracji prywatnych rejestrów w cache.

### Rozszerzenie: reużywanie

Wydziel powtarzalne kroki do composite action albo cały job do reusable workflow.
Composite action wywołujesz w `steps`, reusable workflow w `jobs.<id>.uses`.
Zdefiniuj potrzebne inputs i outputs oraz sprawdź, czy błąd nadal daje czerwone CI.
Nie dodawaj sekretów, jeśli wydzielony proces ich nie potrzebuje.

## 4. Uruchomienie aplikacji do demonstracji

Testy nie wymagają tych kroków. Do korzystania z aplikacji w przeglądarce potrzebna jest osobna baza.

Uruchom demonstracyjny MySQL, publikując port tylko na interfejsie lokalnym:

```bash
docker run -d --name mission-control-lab-db -p 127.0.0.1:3307:3306 -e MYSQL_DATABASE=mission_control -e MYSQL_USER=mission -e MYSQL_PASSWORD=mission-local-password -e MYSQL_ROOT_PASSWORD=mission-local-root-password mysql:8.4
docker logs mission-control-lab-db
```

Poczekaj, aż baza będzie gotowa do połączeń. Podane hasła są wyłącznie przykładowymi
danymi do lokalnego ćwiczenia — nie używaj ich w innych środowiskach.

W PowerShellu:

```powershell
$env:ConnectionStrings__MissionDb = "Server=localhost;Port=3307;Database=mission_control;User=mission;Password=mission-local-password"
dotnet run --project backend --no-restore --urls http://localhost:8080
```

W Bash:

```bash
export ConnectionStrings__MissionDb='Server=localhost;Port=3307;Database=mission_control;User=mission;Password=mission-local-password'
dotnet run --project backend --no-restore --urls http://localhost:8080
```

W drugim terminalu, z katalogu startera:

```bash
cd frontend
npm start
```

Frontend: `http://localhost:4200`. API: `http://localhost:8080/api/missions`.
Health: `http://localhost:8080/health`. OpenAPI JSON: `http://localhost:8080/openapi/v1.json`.
Angular przekazuje `/api` do backendu przez proxy. Backend nie odczytuje automatycznie pliku `.env`;
połączenie przekazujesz przez konfigurację `ConnectionStrings__MissionDb`.

Po zatrzymaniu API i frontendu usuń wyłącznie własny kontener demonstracyjny:

```bash
docker stop mission-control-lab-db
docker rm mission-control-lab-db
```

## Zakres zadania CI

- Utwórz `.github/workflows/ci.yml` od pustego pliku.
- Dodaj uruchamianie dla PR-ów i zmian gałęzi głównej, osobne jobs backendu i frontendu, locked restore, build oraz testy.
- Zapisz raporty także przy niepowodzeniu testów, nie ukrywając błędu CI.
- Wprowadź kontrolowaną regresję, np. usuń walidację pustej nazwy, zobacz czerwone CI i popraw kod.
- Sprawdź niezgodność deklaracji zależności z lockfile i jej naprawę.
- Dodaj i zmierz cache zależności, zachowując poprawny build oraz testy także po cache miss.
- W wariancie rozszerzonym wydziel composite action lub reusable workflow, bez zmiany semantyki sprawdzeń.
- Skonfiguruj wymagane PR-y i checks oraz minimalne uprawnienia workflow; potrzebne uprawnienia do ochrony gałęzi ustal z prowadzącym.
- Nie dodawaj prawdziwych sekretów do kodu, logów ani historii Gita.

Starter nie zawiera gotowego workflow CI, composite action, reusable workflow, Dockerfile ani Compose.
Szkielety w `docs/mockups` są opcjonalnym materiałem do konteneryzacji obecnego wariantu .NET + MySQL,
a nie rozwiązaniem zadania CI.

## Pochodzenie startera

Kod pochodzi z [publicznych materiałów kursu, rewizja `ccc9850`](https://github.com/PJMPR/TDO-lecures/tree/ccc985008e51da48bbdf7a6d5fd889bb65383e60/projects/docker-lab-starter).
Starter uzupełniono o konfigurację i wyniki ćwiczeń CI.


## Dziennik wykonania laboratorium

Praca indywidualna, zadania 1–23 według [instrukcji](https://pjmpr.github.io/TDO-lecures/labs/ci/).
Każdy etap przechodzi przegląd konfiguracji i rzeczywistych wyników przed kolejnym zadaniem.
Kontrolowane błędy pozostają w historii gałęzi; do main trafia naprawiona wersja.

Przygotowanie lokalne: SDK .NET 10.0.401, Node 26.1.0, działający Docker Linux i udany kontener hello-world.
Narzędzia są dostępne lokalnie przez skrypt środowiska poza repozytorium; SDK/Node w CI wynikają z global.json/.nvmrc.

| Zadanie | Commit | PR / uruchomienie | Oczekiwany wynik | Wynik i review |
| --- | --- | --- | --- | --- |
| 1 | `0b10cc2094253327be1509c5e9cf4b1d40bb8087` | [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42274436477) | push, jeden job Ubuntu, dwa nazwane kroki, sukces | PASS: hello success; powitanie, katalog i Linux obecne w logach; oba wlasne kroki success |
| 2 | `3b6575ee5a6c35af22b006e2db6510822abfe334` | [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42274749350) | checkout i SDK z plikow startera, bez cache | PASS: success; dotnet 10.0.403 zgodny z latestPatch, Node v26.1.0, npm 11.13.0, oba mechanizmy cache jawnie wylaczone |
| 3 | `eed96bdf2b974370b5fa4b678d91fe286160cde9` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/1), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42275046981), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42275175248), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42275308752) | PR do main, aktualizacja checks, osobny push/main | PASS: Oba PR runy i run po merge success; brak dodatkowego push na feature. |
| 4 | `522ad1054d4c2c0d5184bfd882cc9e059f2b5832` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/2), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42276072324), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42276273678), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42276359635) | PR Release; reczne Release/false i Debug/true po bootstrapie main | PASS: Wszystkie success; konfiguracje w nazwach i logach zgodne, diagnostyka skipped/success. Wczesniejszy blad skladni YAML poprawiono osobnym commitem i sprawdzono actionlint. |
| 5 | `1b75d0eab835235537398dd88f5ac2ffdc123554` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/3), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42276610467), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42276909015), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42277034292) | Kontrolowany error/exit 1 i naprawa nowym commitem | PASS: Czerwony run: 3 adnotacje (warning/error README.md:1 i exit code 1), pozniejsze kroki skipped. Poprawka i main success, bez maskowania bledow. |
| 6 | `f11d597da20c209918b6d718ae685b04cb6f9c34` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/4), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42278127734), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42278321972) | 3 wygenerowane lockfiles i lokalne locked restore/build/test | PASS: Lokalnie .NET 10.0.401 / Node 26.1.0: 19/19 API i 11/11 UI, zero pominiec/bledow; oba buildy i locked restore pass. Manifesty bez zmian, raporty ignorowane. PR i main (kontrola SDK tego etapu) success. |
| 7 | `099d34ca001339243a55bd811abdca7cbfae6681` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/5), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42278746791), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42279073640), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42279303586) | Locked restore, build i 19 testow API; wybrana konfiguracja | PASS: PR Release, reczny Debug i main success. Logi: 19/19, start/gotowosc/usuniecie MySQL przez Testcontainers, TRX; sciezki bin/Debug potwierdzaja konfiguracje. Bez services i sekretow. |
| 8 | `a36ef3443dc6a257a8782f4335599c41286d3e5b` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/6), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42279649362), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42279865962) | Niezalezne rownolegle Backend i Frontend, 19+11 testow | PASS: Oba jobs success w PR i main; logi 19 API i 11 UI, raporty JUnit/JSON powstaly. W PR wykonania nakladaly sie przez 24 s; Frontend bez needs, API i bazy. |
| 9 | `13be88166d3aaf1f021250aa04bdf1579d1db424` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/7), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/8), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42280339603), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42280833077), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42281613319), [run 4](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42282206473), [run 5](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42282635307) | Osobne fail/pass regresji aplikacji i nieaktualnego lockfile | PASS: Regresja pustej nazwy: 17 pass/2 fail API -> przywrocona walidacja i 19/19. Sam manifest CLI .29 przy lock .24: npm ci EUSAGE -> wygenerowany lokalnie lock i pelny CI pass. Testy niezmienione, rozne pierwsze nieudane kroki, main zielony. |
| 10 | `c84de50f7ee41e2d06b62e4c13af12ed5fc60cac` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/9), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42283151729), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42283737430), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42284022816), [run 4](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42284615530) | Raport nieudanego testu dostepny, CI nadal czerwony | PASS: Pobrano poprawne TRX19/19 i UI11/11. Czerwony run: backend failure, upload success, pobrany TRX total19/pass17/fail2 z BadRequest/Created. Naprawa i main success. Dwa rozne artefakty run/attempt, retencja faktycznie7 dni. |
| 11 | `a991051e49b48d639cd1b69d1c28364b829cdbb6` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/10), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42285316145), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42285676243), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42286389081) | Jawna variable, sama obecnosc fikcyjnego sekretu, minimalny token i testy bez sekretu | PASS: Etykieta w logu, skonfigurowany/brak dla tego samego kodu. Po usunieciu wlasnego sekretu pobrane raporty19/19+11/11, CI success. Oba jobs: Contents read i Metadata read. Sekret tylko w env jednego kroku, nowa fikcyjna wartosc odtworzona, main success. |
| 12 | `b6e2e074ec404f7b02bce8339883dd5a9f65c791` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/11), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42287310163), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42288487752), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42289172604), [run 4](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42289812969) | GH013 dla direct push; czerwony PR BLOCKED; poprawka CLEAN bez bypass. | PASS: Aktywne required PR+Backend+Frontend, bypass pusty, Classroom zachowane; wszystkie trzy stany potwierdzone; main success. |
| 13 | `1cc673fc86bfdd0c729f546b00e6ff73e32bb601` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/12), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290040632), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290187724), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290532119), [run 4](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290717724), [run 5](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42291161330) | Cold/miss, warm/exact hit, nowe lockfiles i namespace/miss; testy zawsze. | PASS: Cztery pomiary obu cache zgodne; 19 API i 11 UI w każdym. Klucze, hit i czasy w docs/evidence/cache-stage13.json. Main zielony. |
| 14 | `05d1ed9a04195fb685dad9edf43d68d53fff5a7e` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/13), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42291362021) | Ten sam output w run i upload; GITHUB_ENV w kolejnym kroku. | PASS: Nazwy w logach i pobranych artefaktach zgodne dla obu komponentów; etykiety odczytane z GITHUB_ENV. Raporty 19/19 i 11/11. |
| 15 | `9f301aa6f5c0ff1ce5f177999b400f021115b2e0` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/13), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42291648985) | Nazwy przez job outputs; pliki przez download-artifact na nowym runnerze. | PASS: Summary bez checkout: brak raportów przed pobraniem, potem trzy niepuste pliki w osobnych katalogach. Outputs zgodne; 19/19 API i 11/11 UI. |
| 16 | `85cd7423b024ed1033faf549c1d5b1716d6947a7` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/14), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/15), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/16), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42292236749), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42292490582), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42293187520), [run 4](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42293407045), [run 5](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42293476344), [run 6](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42292614082), [run 7](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42293545092) | Poprawne flagi docs/backend/frontend/workflow/rename; bez zapisu dla forka. | PASS: Wszystkie zakresy zgodne, rename uwzględnia starą ścieżkę. Jeden aktualizowany komentarz; fork skipped. Pełne runy zielone, aplikacja przywrócona. |
| 17 | `03f4d65b5a28fc77f9c552860ce756016ed8fcfb` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/17), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42294132840), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42294497893), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42294862799), [run 4](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42295024013) | Composite waliduje input, przekazuje output i propaguje błąd z raportem. | PASS: Preview odrzucony; regresja 17/19, TRX dostępny, PR zablokowany. Poprawka: 19/19 i 11/11, poprawne outputs i summary. |
| 18 | `685d60b7b8f74e0f33ca076a25d31bfc046261de` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/18), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42295692005), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42296082494), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42296342041) | Typowane inputs, pełny łańcuch outputu i nowe wymagane checks. | PASS: Raport pobrany przez needs, retencja7dni. Regresja nowego Backend blokuje merge; poprawka19/19+11/11, bez starego Expected i bypass. |
| 19 | `5a79622a90104803aa8bd30f65287fa1711b443f` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/21), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/19), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/20), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/22), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42298904565), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42300271809), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42300421845) | Gate dla 0/1/2 komponentow, bledy blokuja merge. | PASS: Porównanie5stanów i scenariusze gate zgodne. Jedyny required to ci-required; czerwony BLOCKED, końcowe docs-only CLEAN. [Wyniki](docs/ci-gate.md). |
| 20 | `ce23d06f64050a980f6f79e1cd67cba98d490e34` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/23), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/24), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302198517), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302390876), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42302437278) | Dokładny producent, kontrola źródła i schematu. | PASS: Push/main przyjęty, PR/anulowanie pominięte. Wspólny walidator odrzuca fixture schema2. Token actions:read; poprawiona identyfikacja workflow. [Wyniki](docs/consumer.md). |
| 21 | `41d40c6fb25d51a3716f65d166ee95d76b75824b` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/27), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/25), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/26), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42304629777), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42308395878), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42309136735) | Własny PDF, prerelease i pełne wydanie z bezpiecznym target SHA. | PASS: 2 prerelease i pełne wydanie z poprawnymi tagami/PDF1str. Fork/workflow-edit artifact-only; zły TeX bez publikacji. Kontrola wizualna passed. [Wyniki](thesis/README.md). |
| 22 | `a8d2b62b3ad9b229e7632c2fcbac80303ffe5f80` | [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/29), [PR](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/pull/28), [run 1](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42310362077), [run 2](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42310847797), [run 3](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42311007461) | Debug/Release, blad jednego wariantu i include/exclude; komplet testow niezaleznie od uploadu. | PASS: Dwa warianty po19pass; blad Debug daje summary failure przy Release success. Po poprawce trzy warianty po19pass, upload false skipped, diagnostyka tylko Release/true. |

<!-- LAB-REVIEW-TABLE:END -->

## Obserwacje zaleznosci

Stan poczatkowy po wygenerowaniu lockfiles: npm audit zglosil 23 pozycje (18 moderate, 5 high, 0 critical). Lokalne buildy i 19 + 11 testow przeszly. Zmiany wersji w kolejnych zadaniach beda wykonywane jawnie wraz z lockfiles.

Po zgodnej poprawce CLI 21.2.29 (zadanie 9): lokalny npm ci/build i 11/11 UI pass; audyt zglosil 21 pozycji (19 moderate, 2 high). Pozostale deklaracje Angulara zachowano.

Bezpośredni push na main odrzucono; zmiany przechodzą przez PR z wymaganymi checks.

## Ochrona main - dowody etapu 12

[Aktywna regula](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/rules/26724075) oraz [historyczny zapis konfiguracji i stanow PR](docs/evidence/main-protection-stage12.json). Bezposredni push zostal odrzucony przez GH013 i nie zmienil main. PR #11 byl CLEAN po zielonych checks, a po kontrolowanej regresji BLOCKED przy Backend FAILURE i Frontend SUCCESS.

## Cache zależności

Pomiary manualne na tej samej gałęzi. Brak restore-keys; miss oznacza brak przywrócenia. Pełne klucze i logi: [cache-stage13.json](docs/evidence/cache-stage13.json).

| Próba | npm ci | NuGet restore | Exact hit | Run |
| --- | --- | --- | --- | --- |
| cold | 9 s | 6 s | oba false | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290040632) |
| warm | 7 s | 4 s | oba true | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290187724) |
| newlock | 9 s | 6 s | oba false | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290532119) |
| namespace | 7 s | 6 s | oba false | [run](https://github.com/shihan-project/tdo-2026-lab-02-s30576-pj/actions/runs/42290717724) |

Każdy run wykonał instalację, build oraz 19 testów API i 11 UI. Wersje do próby nowego lockfile: OpenAPI 10.0.14 i @angular/build 21.2.29. Po aktualizacji npm audit: 18 moderate, 0 high, 0 critical.
