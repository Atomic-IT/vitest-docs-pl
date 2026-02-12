---
title: Interfejs wiersza poleceń | Przewodnik
outline: deep
---

# Interfejs wiersza poleceń

## Komendy

### `vitest`

Uruchom Vitest w bieżącym katalogu. Automatycznie wejdzie w tryb watch w środowisku deweloperskim i tryb uruchomienia w CI (lub nieinteraktywnym terminalu).

Możesz przekazać dodatkowy argument jako filtr plików testowych do uruchomienia. Na przykład:

```bash
vitest foobar
```

Uruchomi tylko plik testowy, który zawiera `foobar` w ścieżce. Ten filtr sprawdza tylko zawieranie i nie obsługuje wzorców regexp ani glob (chyba że terminal przetworzy je przed przekazaniem filtru do Vitest).

Od Vitest 3 możesz również określić test według nazwy pliku i numeru linii:

```bash
$ vitest basic/foo.test.ts:10
```

::: warning
Zauważ, że Vitest wymaga pełnej nazwy pliku, aby ta funkcja działała. Może być względna do bieżącego katalogu roboczego lub absolutną ścieżką do pliku.

```bash
$ vitest basic/foo.js:10 # ✅
$ vitest ./basic/foo.js:10 # ✅
$ vitest /users/project/basic/foo.js:10 # ✅
$ vitest foo:10 # ❌
$ vitest ./basic/foo:10 # ❌
```

W tej chwili Vitest również nie obsługuje zakresów:

```bash
$ vitest basic/foo.test.ts:10, basic/foo.test.ts:25 # ✅
$ vitest basic/foo.test.ts:10-25 # ❌
```
:::

### `vitest run`

Wykonaj pojedyncze uruchomienie bez trybu obserwacji.

### `vitest watch`

Uruchom wszystkie zestawy testów, ale obserwuj zmiany i ponownie uruchamiaj testy, gdy się zmienią. To samo co wywołanie `vitest` bez argumentu. Przejdzie do `vitest run` w CI lub gdy stdin nie jest TTY (nieinteraktywne środowisko).

### `vitest dev`

Alias do `vitest watch`.

### `vitest related`

Uruchom tylko testy, które pokrywają listę plików źródłowych. Działa z importami statycznymi (np. `import('./index.js')` lub `import index from './index.js`), ale nie z dynamicznymi (np. `import(filepath)`). Wszystkie pliki powinny być względne do głównego folderu.

Przydatne do uruchamiania z [`lint-staged`](https://github.com/okonet/lint-staged) lub z Twoją konfiguracją CI.

```bash
vitest related /src/index.ts /src/hello-world.js
```

::: tip
Nie zapominaj, że Vitest domyślnie działa z włączonym trybem obserwacji. Jeśli używasz narzędzi takich jak `lint-staged`, powinieneś również przekazać opcję `--run`, aby komenda mogła normalnie zakończyć działanie.

```js [.lintstagedrc.js]
export default {
  '*.{js,ts}': 'vitest related --run',
}
```
:::

### `vitest bench`

Uruchom tylko testy [benchmark](/guide/features.html#benchmarking), które porównują wyniki wydajności.

### `vitest init`

`vitest init <name>` może być użyte do konfiguracji projektu. W tej chwili obsługuje tylko wartość [`browser`](/guide/browser/):

```bash
vitest init browser
```

### `vitest list`

Komenda `vitest list` dziedziczy wszystkie opcje `vitest`, aby wyświetlić listę wszystkich pasujących testów. Ta komenda ignoruje opcję `reporters`. Domyślnie wyświetli nazwy wszystkich testów, które pasowały do filtra plików i wzorca nazwy:

```shell
vitest list filename.spec.ts -t="some-test"
```

```txt
describe > some-test
describe > some-test > test 1
describe > some-test > test 2
```

Możesz przekazać flagę `--json`, aby wyświetlić testy w formacie JSON lub zapisać je w oddzielnym pliku:

```bash
vitest list filename.spec.ts -t="some-test" --json=./file.json
```

Jeśli flaga `--json` nie otrzyma wartości, wyświetli JSON na stdout.

Możesz również przekazać flagę `--filesOnly`, aby wyświetlić tylko pliki testowe:

```bash
vitest list --filesOnly
```

```txt
tests/test1.test.ts
tests/test2.test.ts
```

## Opcje

::: tip
Vitest obsługuje zarówno camelCase, jak i kebab-case dla argumentów CLI. Na przykład zarówno `--passWithNoTests`, jak i `--pass-with-no-tests` będą działać (`--no-color` i `--inspect-brk` są wyjątkami).

Vitest obsługuje również różne sposoby określania wartości: zarówno `--reporter dot`, jak i `--reporter=dot` są prawidłowe.

Jeśli opcja obsługuje tablicę wartości, musisz przekazać opcję wielokrotnie:

```
vitest --reporter=dot --reporter=default
```

Opcje logiczne mogą być zanegowane za pomocą przedrostka `no-`. Określenie wartości jako `false` również działa:

```
vitest --no-api
vitest --api=false
```
:::

<!--@include: ./cli-generated.md-->

### changed

- **Typ**: `boolean | string`
- **Domyślnie**: false

Uruchom testy tylko dla zmienionych plików. Jeśli nie podano wartości, uruchomi testy dla niezatwierdzonych zmian (włącznie ze staged i unstaged).

Aby uruchomić testy dla zmian dokonanych w ostatnim commicie, możesz użyć `--changed HEAD~1`. Możesz również przekazać hash commita (np. `--changed 09a9920`) lub nazwę brancha (np. `--changed origin/develop`).

Gdy używany z pokryciem kodu, raport będzie zawierał tylko pliki związane ze zmianami.

W połączeniu z opcją konfiguracji [`forceRerunTriggers`](/config/#forcereruntriggers) uruchomi cały zestaw testów, jeśli zmieni się przynajmniej jeden z plików wymienionych na liście `forceRerunTriggers`. Domyślnie zmiany w pliku konfiguracyjnym Vitest i `package.json` zawsze ponownie uruchomią cały zestaw.

### shard

- **Typ**: `string`
- **Domyślnie**: wyłączone

Fragment zestawu testów do wykonania w formacie `<index>`/`<count>`, gdzie

- `count` to dodatnia liczba całkowita, liczba podzielonych części
- `index` to dodatnia liczba całkowita, indeks podzielonej części

Ta komenda podzieli wszystkie testy na `count` równych części i uruchomi tylko te, które znajdują się w części o indeksie `index`. Na przykład, aby podzielić zestaw testów na trzy części, użyj:

```sh
vitest run --shard=1/3
vitest run --shard=2/3
vitest run --shard=3/3
```

:::warning
Nie możesz użyć tej opcji z włączonym `--watch` (włączony domyślnie w trybie dev).
:::

::: tip
Jeśli `--reporter=blob` jest używany bez pliku wyjściowego, domyślna ścieżka będzie zawierać bieżącą konfigurację shard, aby uniknąć kolizji z innymi procesami Vitest.
:::

### merge-reports

- **Typ:** `boolean | string`

Łączy każdy raport blob znajdujący się w określonym folderze (domyślnie `.vitest-reports`). Możesz użyć dowolnych reporterów z tą komendą (z wyjątkiem [`blob`](/guide/reporters#blob-reporter)):

```sh
vitest --merge-reports --reporter=junit
```

[cac's dot notation]: https://github.com/cacjs/cac#dot-nested-options
