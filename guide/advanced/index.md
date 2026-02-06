---
title: Zaawansowane API
---

# Rozpoczęcie pracy <Badge type="danger">zaawansowane</Badge> {#getting-started}

::: warning
Ten przewodnik zawiera listę zaawansowanych API do uruchamiania testów za pomocą skryptu Node.js. Jeśli chcesz tylko [uruchamiać testy](/guide/), prawdopodobnie tego nie potrzebujesz. Jest przeznaczony głównie dla autorów bibliotek.
:::

Możesz importować dowolną metodę z punktu wejścia `vitest/node`.

## startVitest

```ts
function startVitest(
  mode: VitestRunMode,
  cliFilters: string[] = [],
  options: CliOptions = {},
  viteOverrides?: ViteUserConfig,
  vitestOptions?: VitestOptions,
): Promise<Vitest>
```

Możesz zacząć uruchamiać testy Vitest używając jego Node API:

```js
import { startVitest } from 'vitest/node'

const vitest = await startVitest('test')

await vitest.close()
```

Funkcja `startVitest` zwraca instancję [`Vitest`](/api/advanced/vitest), jeśli testy mogą być uruchomione.

Jeśli tryb watch nie jest włączony, Vitest automatycznie wywoła metodę `close`.

Jeśli tryb watch jest włączony i terminal wspiera TTY, Vitest zarejestruje skróty konsolowe.

Możesz przekazać listę filtrów jako drugi argument. Vitest uruchomi tylko testy, które zawierają co najmniej jeden z przekazanych ciągów znaków w ścieżce pliku.

Dodatkowo możesz użyć trzeciego argumentu do przekazania argumentów CLI, które nadpiszą wszelkie opcje konfiguracji testu. Alternatywnie możesz przekazać pełną konfigurację Vite jako czwarty argument, która będzie miała pierwszeństwo przed innymi opcjami zdefiniowanymi przez użytkownika.

Po uruchomieniu testów możesz uzyskać wyniki z API [`state.getTestModules`](/api/advanced/test-module):

```ts
import type { TestModule } from 'vitest/node'

const vitest = await startVitest('test')

console.log(vitest.state.getTestModules()) // [TestModule]
```

::: tip
Przewodnik ["Uruchamianie testów"](/guide/advanced/tests#startvitest) zawiera przykład użycia.
:::

## createVitest

```ts
function createVitest(
  mode: VitestRunMode,
  options: CliOptions,
  viteOverrides: ViteUserConfig = {},
  vitestOptions: VitestOptions = {},
): Promise<Vitest>
```

Możesz utworzyć instancję Vitest używając funkcji `createVitest`. Zwraca tę samą instancję [`Vitest`](/api/advanced/vitest) co `startVitest`, ale nie uruchamia testów i nie waliduje zainstalowanych pakietów.

```js
import { createVitest } from 'vitest/node'

const vitest = await createVitest('test', {
  watch: false,
})
```

::: tip
Przewodnik ["Uruchamianie testów"](/guide/advanced/tests#createvitest) zawiera przykład użycia.
:::

## resolveConfig

```ts
function resolveConfig(
  options: UserConfig = {},
  viteOverrides: ViteUserConfig = {},
): Promise<{
  vitestConfig: ResolvedConfig
  viteConfig: ResolvedViteConfig
}>
```

Ta metoda rozwiązuje konfigurację z niestandardowymi parametrami. Jeśli nie podano parametrów, `root` będzie `process.cwd()`.

```ts
import { resolveConfig } from 'vitest/node'

// vitestConfig zawiera tylko rozwiązane właściwości "test"
const { vitestConfig, viteConfig } = await resolveConfig({
  mode: 'custom',
  configFile: false,
  resolve: {
    conditions: ['custom']
  },
  test: {
    setupFiles: ['/my-setup-file.js'],
    pool: 'threads',
  },
})
```

::: info
Ze względu na sposób działania `createServer` w Vite, Vitest musi rozwiązać konfigurację podczas hooka `configResolve` pluginu. Dlatego ta metoda nie jest faktycznie używana wewnętrznie i jest udostępniona wyłącznie jako publiczne API.

Jeśli przekażesz konfigurację do API `startVitest` lub `createVitest`, Vitest nadal rozwiąże konfigurację ponownie.
:::

::: warning
`resolveConfig` nie rozwiązuje `projects`. Aby rozwiązać konfiguracje projektów, Vitest potrzebuje ustanowionego serwera Vite.

Zauważ również, że `viteConfig.test` nie będzie w pełni rozwiązane. Jeśli potrzebujesz konfiguracji Vitest, użyj zamiast tego `vitestConfig`.
:::

## parseCLI

```ts
function parseCLI(argv: string | string[], config: CliParseOptions = {}): {
  filter: string[]
  options: CliOptions
}
```

Możesz użyć tej metody do parsowania argumentów CLI. Akceptuje ciąg znaków (gdzie argumenty są rozdzielone pojedynczą spacją) lub tablicę ciągów znaków argumentów CLI w tym samym formacie, którego używa CLI Vitest. Zwraca filtr i `options`, które możesz później przekazać do metod `createVitest` lub `startVitest`.

```ts
import { parseCLI } from 'vitest/node'

const result = parseCLI('vitest ./files.ts --coverage --browser=chrome')

result.options
// {
//   coverage: { enabled: true },
//   browser: { name: 'chrome', enabled: true }
// }

result.filter
// ['./files.ts']
```
