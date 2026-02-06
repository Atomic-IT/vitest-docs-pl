# Uruchamianie testów <Badge type="danger">zaawansowane</Badge> {#running-tests}

::: warning
Ten przewodnik wyjaśnia, jak używać zaawansowanego API do uruchamiania testów za pomocą skryptu Node.js. Jeśli chcesz tylko [uruchamiać testy](/guide/), prawdopodobnie tego nie potrzebujesz. Jest przeznaczony głównie dla autorów bibliotek.
:::

Vitest udostępnia dwie metody do inicjowania Vitest:

- `startVitest` inicjuje Vitest, waliduje czy pakiety są zainstalowane i natychmiast uruchamia testy
- `createVitest` tylko inicjuje Vitest i nie uruchamia żadnych testów

## `startVitest`

```ts
import { startVitest } from 'vitest/node'

const vitest = await startVitest(
  'test',
  [], // filtry CLI
  {}, // nadpisanie konfiguracji testu
  {}, // nadpisanie konfiguracji Vite
  {}, // niestandardowe opcje Vitest
)
const testModules = vitest.state.getTestModules()
for (const testModule of testModules) {
  console.log(testModule.moduleId, testModule.ok() ? 'passed' : 'failed')
}
```

::: tip
API [`TestModule`](/api/advanced/test-module), [`TestSuite`](/api/advanced/test-suite) i [`TestCase`](/api/advanced/test-case) nie są eksperymentalne i przestrzegają SemVer od Vitest 2.1.
:::

## `createVitest`

Tworzy instancję [Vitest](/api/advanced/vitest) bez uruchamiania testów.

Metoda `createVitest` nie waliduje czy wymagane pakiety są zainstalowane. Nie respektuje również `config.standalone` ani `config.mergeReports`. Vitest nie zostanie zamknięty automatycznie nawet jeśli `watch` jest wyłączony.

```ts
import { createVitest } from 'vitest/node'

const vitest = await createVitest(
  'test',
  {}, // nadpisanie konfiguracji testu
  {}, // nadpisanie konfiguracji Vite
  {}, // niestandardowe opcje Vitest
)

// wywoływane gdy `vitest.cancelCurrentRun()` jest wywołane
vitest.onCancel(() => {})
// wywoływane podczas wywołania `vitest.close()`
vitest.onClose(() => {})
// wywoływane gdy Vitest ponownie uruchamia pliki testowe
vitest.onTestsRerun((files) => {})

try {
  // to ustawi process.exitCode na 1 jeśli testy nie powiodą się,
  // i nie zamknie procesu automatycznie
  await vitest.start(['my-filter'])
}
catch (err) {
  // to może rzucić
  // "FilesNotFoundError" jeśli nie znaleziono plików
  // "GitNotFoundError" z `--changed` i repozytorium nie jest zainicjalizowane
}
finally {
  await vitest.close()
}
```

Jeśli zamierzasz zachować instancję `Vitest`, upewnij się, że przynajmniej wywołasz [`init`](/api/advanced/vitest#init). To zainicjuje reportery i dostawcę pokrycia, ale nie uruchomi żadnych testów. Zaleca się również włączenie trybu `watch` nawet jeśli nie zamierzasz używać watchera Vitest, ale chcesz zachować instancję działającą. Vitest polega na tej fladze dla prawidłowego działania niektórych funkcji w ciągłym procesie.

Po zainicjowaniu reporterów użyj [`runTestSpecifications`](/api/advanced/vitest#runtestspecifications) lub [`rerunTestSpecifications`](/api/advanced/vitest#reruntestspecifications), aby uruchomić testy, jeśli wymagane jest ręczne uruchomienie:

```ts
watcher.on('change', async (file) => {
  const specifications = vitest.getModuleSpecifications(file)
  if (specifications.length) {
    vitest.invalidateFile(file)
    // możesz użyć runTestSpecifications jeśli hooki "reporter.onWatcher*"
    // nie powinny być wywoływane
    await vitest.rerunTestSpecifications(specifications)
  }
})
```

::: warning
Powyższy przykład pokazuje potencjalny przypadek użycia jeśli wyłączysz domyślne zachowanie watchera. Domyślnie Vitest już ponownie uruchamia testy jeśli pliki się zmienią.

Zauważ również, że `getModuleSpecifications` nie rozwiąże plików testowych, chyba że zostały już przetworzone przez `globTestSpecifications`. Jeśli plik został właśnie utworzony, użyj zamiast tego `project.matchesGlobPattern`:

```ts
watcher.on('add', async (file) => {
  const specifications = []
  for (const project of vitest.projects) {
    if (project.matchesGlobPattern(file)) {
      specifications.push(project.createSpecification(file))
    }
  }

  if (specifications.length) {
    await vitest.rerunTestSpecifications(specifications)
  }
})
```
:::

W przypadkach gdy potrzebujesz wyłączyć watcher, możesz przekazać `server.watch: null` od Vite 5.3 lub `server.watch: { ignored: ['*/*'] }` do konfiguracji Vite:

```ts
await createVitest(
  'test',
  {},
  {
    plugins: [
      {
        name: 'stop-watcher',
        async configureServer(server) {
          await server.watcher.close()
        }
      }
    ],
    server: {
      watch: null,
    },
  }
)
```
