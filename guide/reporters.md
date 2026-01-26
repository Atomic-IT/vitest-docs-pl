---
title: Reportery | Przewodnik
outline: deep
---

# Reportery

Vitest dostarcza kilka wbudowanych reporterów do wyświetlania wyników testów w różnych formatach, a także możliwość używania niestandardowych reporterów. Możesz wybrać różne reportery używając opcji wiersza poleceń `--reporter` lub dołączając właściwość `reporters` w swoim [pliku konfiguracyjnym](/config/#reporters). Jeśli reporter nie jest określony, Vitest użyje reportera `default`, jak opisano poniżej.

Używanie reporterów przez wiersz poleceń:

```bash
npx vitest --reporter=verbose
```

Używanie reporterów przez [`vitest.config.ts`](/config/):

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    reporters: ['verbose']
  },
})
```

Niektóre reportery można dostosować, przekazując im dodatkowe opcje. Opcje specyficzne dla reporterów są opisane w sekcjach poniżej.

```ts
export default defineConfig({
  test: {
    reporters: [
      'default',
      ['junit', { suiteName: 'Testy UI' }]
    ],
  },
})
```

## Wyjście reportera

Domyślnie reportery Vitest drukują swoje wyjście do terminala. Podczas używania reporterów `json`, `html` lub `junit` możesz zamiast tego zapisać wyjście testów do pliku, dołączając opcję konfiguracji [`outputFile`](/config/#outputfile) w pliku konfiguracyjnym Vite lub przez CLI.

:::code-group
```bash [CLI]
npx vitest --reporter=json --outputFile=./test-output.json
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['json'],
    outputFile: './test-output.json'
  },
})
```
:::

## Łączenie reporterów

Możesz używać wielu reporterów jednocześnie, aby drukować wyniki testów w różnych formatach. Na przykład:

```bash
npx vitest --reporter=json --reporter=default
```

```ts
export default defineConfig({
  test: {
    reporters: ['json', 'default'],
    outputFile: './test-output.json'
  },
})
```

Powyższy przykład wydrukuje wyniki testów do terminala w domyślnym stylu i zapisze je jako JSON do wskazanego pliku wyjściowego.

Przy używaniu wielu reporterów możliwe jest również wskazanie wielu plików wyjściowych, w następujący sposób:

```ts
export default defineConfig({
  test: {
    reporters: ['junit', 'json', 'verbose'],
    outputFile: {
      junit: './junit-report.xml',
      json: './json-report.json',
    },
  },
})
```

Ten przykład zapisze oddzielne raporty JSON i XML, a także wydrukuje szczegółowy raport do terminala.

## Wbudowane reportery

### Reporter Default

Domyślnie (tzn. jeśli reporter nie jest określony), Vitest wyświetla podsumowanie uruchomionych testów i ich status na dole. Gdy suite przejdzie, jego status zostanie zgłoszony na górze podsumowania.

Możesz wyłączyć podsumowanie, konfigurując reporter:

:::code-group
```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: [
      ['default', { summary: false }]
    ]
  },
})
```
:::

Przykładowe wyjście dla testów w trakcie:

```bash
 ✓ test/example-1.test.ts (5 tests | 1 skipped) 306ms
 ✓ test/example-2.test.ts (5 tests | 1 skipped) 307ms

 ❯ test/example-3.test.ts 3/5
 ❯ test/example-4.test.ts 1/5

 Test Files 2 passed (4)
      Tests 10 passed | 3 skipped (65)
   Start at 11:01:36
   Duration 2.00s
```

Końcowe wyjście po zakończeniu testów:

```bash
 ✓ test/example-1.test.ts (5 tests | 1 skipped) 306ms
 ✓ test/example-2.test.ts (5 tests | 1 skipped) 307ms
 ✓ test/example-3.test.ts (5 tests | 1 skipped) 307ms
 ✓ test/example-4.test.ts (5 tests | 1 skipped) 307ms

 Test Files  4 passed (4)
      Tests  16 passed | 4 skipped (20)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

Jeśli uruchamiany jest tylko jeden plik testowy, Vitest wyświetli pełne drzewo testów tego pliku, podobnie do reportera [`tree`](#tree-reporter). Reporter default również wydrukuje drzewo testów, jeśli w pliku jest co najmniej jeden nieudany test.

```bash
✓ __tests__/file1.test.ts (2) 725ms
   ✓ first test file (2) 725ms
     ✓ 2 + 2 should equal 4
     ✓ 4 - 2 should equal 2

 Test Files  1 passed (1)
      Tests  2 passed (2)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### Reporter Verbose

Reporter verbose drukuje każdy przypadek testowy po jego zakończeniu. Nie raportuje suite'ów ani plików oddzielnie. Jeśli włączono `--includeTaskLocation`, uwzględni również lokalizację każdego testu w wyjściu. Podobnie do reportera `default`, możesz wyłączyć podsumowanie, konfigurując reporter.

Dodatkowo reporter `verbose` drukuje komunikaty błędów testów od razu. Pełny błąd testu jest raportowany po zakończeniu uruchomienia testu.

Jest to jedyny reporter terminalowy, który raportuje [adnotacje](/guide/test-annotations), gdy test nie kończy się niepowodzeniem.

:::code-group
```bash [CLI]
npx vitest --reporter=verbose
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: [
      ['verbose', { summary: false }]
    ]
  },
})
```
:::

Przykładowe wyjście:

```bash
✓ __tests__/file1.test.ts > first test file > 2 + 2 should equal 4 1ms
✓ __tests__/file1.test.ts > first test file > 4 - 2 should equal 2 1ms
✓ __tests__/file2.test.ts > second test file > 1 + 1 should equal 2 1ms
✓ __tests__/file2.test.ts > second test file > 2 - 1 should equal 1 1ms

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

Przykład z `--includeTaskLocation`:

```bash
✓ __tests__/file1.test.ts:2:1 > first test file > 2 + 2 should equal 4 1ms
✓ __tests__/file1.test.ts:3:1 > first test file > 4 - 2 should equal 2 1ms
✓ __tests__/file2.test.ts:2:1 > second test file > 1 + 1 should equal 2 1ms
✓ __tests__/file2.test.ts:3:1 > second test file > 2 - 1 should equal 1 1ms

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### Reporter Tree

Reporter tree jest taki sam jak reporter `default`, ale również wyświetla każdy indywidualny test po zakończeniu suite. Podobnie do reportera `default`, możesz wyłączyć podsumowanie, konfigurując reporter.

:::code-group
```bash [CLI]
npx vitest --reporter=tree
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: [
      ['tree', { summary: false }]
    ]
  },
})
```
:::

Przykładowe wyjście dla testów w trakcie z domyślnym `slowTestThreshold: 300`:

```bash
 ✓ __tests__/example-1.test.ts (2) 725ms
   ✓ first test file (2) 725ms
     ✓ 2 + 2 should equal 4
     ✓ 4 - 2 should equal 2

 ❯ test/example-2.test.ts 3/5
   ↳ should run longer than three seconds 1.57s
 ❯ test/example-3.test.ts 1/5

 Test Files 2 passed (4)
      Tests 10 passed | 3 skipped (65)
   Start at 11:01:36
   Duration 2.00s
```

Przykład końcowego wyjścia terminala dla udanego zestawu testów:

```bash
✓ __tests__/file1.test.ts (2) 725ms
   ✓ first test file (2) 725ms
     ✓ 2 + 2 should equal 4
     ✓ 4 - 2 should equal 2
✓ __tests__/file2.test.ts (2) 746ms
  ✓ second test file (2) 746ms
    ✓ 1 + 1 should equal 2
    ✓ 2 - 1 should equal 1

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### Reporter Dot

Drukuje pojedynczą kropkę dla każdego ukończonego testu, zapewniając minimalne wyjście, jednocześnie pokazując wszystkie testy, które zostały uruchomione. Szczegóły są podawane tylko dla nieudanych testów, wraz z podsumowaniem dla suite.

:::code-group
```bash [CLI]
npx vitest --reporter=dot
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['dot']
  },
})
```
:::

Przykładowe wyjście terminala dla udanego zestawu testów:

```bash
....

 Test Files  2 passed (2)
      Tests  4 passed (4)
   Start at  12:34:32
   Duration  1.26s (transform 35ms, setup 1ms, collect 90ms, tests 1.47s, environment 0ms, prepare 267ms)
```

### Reporter JUnit

Generuje raport wyników testów w formacie JUnit XML. Może być drukowany do terminala lub zapisany do pliku XML za pomocą opcji konfiguracji [`outputFile`](/config/#outputfile).

:::code-group
```bash [CLI]
npx vitest --reporter=junit
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['junit']
  },
})
```
:::

Przykład raportu JUnit XML:
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<testsuites name="vitest tests" tests="2" failures="1" errors="0" time="0.503">
    <testsuite name="__tests__/test-file-1.test.ts" timestamp="2023-10-19T17:41:58.580Z" hostname="My-Computer.local" tests="2" failures="1" errors="0" skipped="0" time="0.013">
        <testcase classname="__tests__/test-file-1.test.ts" name="first test file &gt; 2 + 2 should equal 4" time="0.01">
            <failure message="expected 5 to be 4 // Object.is equality" type="AssertionError">
AssertionError: expected 5 to be 4 // Object.is equality
 ❯ __tests__/test-file-1.test.ts:20:28
            </failure>
        </testcase>
        <testcase classname="__tests__/test-file-1.test.ts" name="first test file &gt; 4 - 2 should equal 2" time="0">
        </testcase>
    </testsuite>
</testsuites>
```

Wygenerowany XML zawiera zagnieżdżone tagi `testsuites` i `testcase`. Mogą być również dostosowane za pomocą opcji reportera `suiteName` i `classnameTemplate`. `classnameTemplate` może być albo stringiem szablonu, albo funkcją.

Wspierane placeholdery dla opcji `classnameTemplate` to:
- filename
- filepath

```ts
export default defineConfig({
  test: {
    reporters: [
      ['junit', { suiteName: 'niestandardowa nazwa suite', classnameTemplate: 'filename:{filename} - filepath:{filepath}' }]
    ]
  },
})
```

### Reporter JSON

Generuje raport wyników testów w formacie JSON kompatybilnym z opcją `--json` Jesta. Może być drukowany do terminala lub zapisany do pliku za pomocą opcji konfiguracji [`outputFile`](/config/#outputfile).

:::code-group
```bash [CLI]
npx vitest --reporter=json
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['json']
  },
})
```
:::

Przykład raportu JSON:

```json
{
  "numTotalTestSuites": 4,
  "numPassedTestSuites": 2,
  "numFailedTestSuites": 1,
  "numPendingTestSuites": 1,
  "numTotalTests": 4,
  "numPassedTests": 1,
  "numFailedTests": 1,
  "numPendingTests": 1,
  "numTodoTests": 1,
  "startTime": 1697737019307,
  "success": false,
  "testResults": [
    {
      "assertionResults": [
        {
          "ancestorTitles": [
            "",
            "first test file"
          ],
          "fullName": " first test file 2 + 2 should equal 4",
          "status": "failed",
          "title": "2 + 2 should equal 4",
          "duration": 9,
          "failureMessages": [
            "expected 5 to be 4 // Object.is equality"
          ],
          "location": {
            "line": 20,
            "column": 28
          },
          "meta": {}
        }
      ],
      "startTime": 1697737019787,
      "endTime": 1697737019797,
      "status": "failed",
      "message": "",
      "name": "/root-directory/__tests__/test-file-1.test.ts"
    }
  ],
  "coverageMap": {}
}
```

::: info
Od Vitest 3, reporter JSON zawiera informacje o pokryciu w `coverageMap`, jeśli pokrycie jest włączone.
:::

### Reporter HTML

Generuje plik HTML do przeglądania wyników testów przez interaktywne [GUI](/guide/ui). Po wygenerowaniu pliku Vitest utrzymuje działający lokalny serwer deweloperski i dostarcza link do przeglądania raportu w przeglądarce.

Plik wyjściowy można określić za pomocą opcji konfiguracji [`outputFile`](/config/#outputfile). Jeśli opcja `outputFile` nie jest podana, zostanie utworzony nowy plik HTML.

:::code-group
```bash [CLI]
npx vitest --reporter=html
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['html']
  },
})
```
:::

::: tip
Ten reporter wymaga zainstalowanego pakietu [`@vitest/ui`](/guide/ui).
:::

### Reporter TAP

Generuje raport zgodny z [Test Anything Protocol](https://testanything.org/) (TAP).

:::code-group
```bash [CLI]
npx vitest --reporter=tap
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['tap']
  },
})
```
:::

Przykład raportu TAP:
```bash
TAP version 13
1..1
not ok 1 - __tests__/test-file-1.test.ts # time=14.00ms {
    1..1
    not ok 1 - first test file # time=13.00ms {
        1..2
        not ok 1 - 2 + 2 should equal 4 # time=11.00ms
            ---
            error:
                name: "AssertionError"
                message: "expected 5 to be 4 // Object.is equality"
            at: "/root-directory/__tests__/test-file-1.test.ts:20:28"
            actual: "5"
            expected: "4"
            ...
        ok 2 - 4 - 2 should equal 2 # time=1.00ms
    }
}
```

### Reporter TAP Flat

Generuje płaski raport TAP. Podobnie jak reporter `tap`, wyniki testów są formatowane zgodnie ze standardami TAP, ale zestawy testów są formatowane jako płaska lista zamiast zagnieżdżonej hierarchii.

:::code-group
```bash [CLI]
npx vitest --reporter=tap-flat
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['tap-flat']
  },
})
```
:::

Przykład płaskiego raportu TAP:
```bash
TAP version 13
1..2
not ok 1 - __tests__/test-file-1.test.ts > first test file > 2 + 2 should equal 4 # time=11.00ms
    ---
    error:
        name: "AssertionError"
        message: "expected 5 to be 4 // Object.is equality"
    at: "/root-directory/__tests__/test-file-1.test.ts:20:28"
    actual: "5"
    expected: "4"
    ...
ok 2 - __tests__/test-file-1.test.ts > first test file > 4 - 2 should equal 2 # time=0.00ms
```

### Reporter Hanging Process

Wyświetla listę zawieszonych procesów, jeśli jakiekolwiek uniemożliwiają Vitest bezpieczne zakończenie. Reporter `hanging-process` sam nie wyświetla wyników testów, ale może być używany w połączeniu z innym reporterem do monitorowania procesów podczas uruchamiania testów. Używanie tego reportera może być zasobożerne, więc powinien być ogólnie zarezerwowany do celów debugowania w sytuacjach, gdy Vitest konsekwentnie nie może zakończyć procesu.

:::code-group
```bash [CLI]
npx vitest --reporter=hanging-process
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['hanging-process']
  },
})
```
:::

### Reporter GitHub Actions {#github-actions-reporter}

Generuje [polecenia workflow](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-error-message)
do dostarczania adnotacji dla niepowodzeń testów. Ten reporter jest automatycznie włączany z reporterem [`default`](#default-reporter), gdy `process.env.GITHUB_ACTIONS === 'true'`.

<img alt="GitHub Actions" img-dark src="https://github.com/vitest-dev/vitest/assets/4232207/336cddc2-df6b-4b8a-8e72-4d00010e37f5">
<img alt="GitHub Actions" img-light src="https://github.com/vitest-dev/vitest/assets/4232207/ce8447c1-0eab-4fe1-abef-d0d322290dca">

Jeśli konfigurujesz niestandardowe reportery, musisz jawnie dodać `github-actions`.

```ts
export default defineConfig({
  test: {
    reporters: process.env.GITHUB_ACTIONS ? ['dot', 'github-actions'] : ['dot'],
  },
})
```

Możesz dostosować ścieżki plików, które są drukowane w [formacie polecenia adnotacji GitHub](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions), używając opcji `onWritePath`. Jest to przydatne przy uruchamianiu Vitest w środowisku kontenerowym, takim jak Docker, gdzie ścieżki plików mogą nie odpowiadać ścieżkom w środowisku GitHub Actions.

```ts
export default defineConfig({
  test: {
    reporters: process.env.GITHUB_ACTIONS
      ? [
          'default',
          ['github-actions', { onWritePath(path) {
            return path.replace(/^\/app\//, `${process.env.GITHUB_WORKSPACE}/`)
          } }],
        ]
      : ['default'],
  },
})
```

Jeśli używasz [API adnotacji](/guide/test-annotations), reporter automatycznie wyświetli je w interfejsie GitHub. Możesz to wyłączyć, ustawiając opcję `displayAnnotations` na `false`:

```ts
export default defineConfig({
  test: {
    reporters: [
      ['github-actions', { displayAnnotations: false }],
    ],
  },
})
```

### Reporter Blob

Przechowuje wyniki testów na maszynie, aby mogły być później scalone za pomocą polecenia [`--merge-reports`](/guide/cli#merge-reports).
Domyślnie przechowuje wszystkie wyniki w folderze `.vitest-reports`, ale można to nadpisać flagami `--outputFile` lub `--outputFile.blob`.

```bash
npx vitest --reporter=blob --outputFile=reports/blob-1.json
```

Zalecamy używanie tego reportera, jeśli uruchamiasz Vitest na różnych maszynach z flagą [`--shard`](/guide/cli#shard).
Wszystkie raporty blob można scalić w dowolny raport za pomocą polecenia `--merge-reports` na końcu pipeline CI:

```bash
npx vitest --merge-reports=reports --reporter=json --reporter=default
```

::: tip
Zarówno `--reporter=blob`, jak i `--merge-reports` nie działają w trybie watch.
:::

## Niestandardowe reportery

Możesz używać zewnętrznych niestandardowych reporterów zainstalowanych z NPM, określając nazwę ich pakietu w opcji reporters:

:::code-group
```bash [CLI]
npx vitest --reporter=some-published-vitest-reporter
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    reporters: ['some-published-vitest-reporter']
  },
})
```
:::

Dodatkowo możesz zdefiniować własne [niestandardowe reportery](/guide/advanced/reporters) i używać ich, określając ścieżkę do pliku:

```bash
npx vitest --reporter=./path/to/reporter.ts
```

Niestandardowe reportery powinny implementować [interfejs Reporter](https://github.com/vitest-dev/vitest/blob/main/packages/vitest/src/node/types/reporter.ts).
