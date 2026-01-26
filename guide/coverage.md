---
title: Pokrycie | Przewodnik
---

# Pokrycie

Vitest wspiera natywne pokrycie kodu przez [`v8`](https://v8.dev/blog/javascript-code-coverage) oraz instrumentowane pokrycie kodu przez [`istanbul`](https://istanbul.js.org/).

## Dostawcy pokrycia

Zarówno wsparcie `v8`, jak i `istanbul` jest opcjonalne. Domyślnie używany będzie `v8`.

Możesz wybrać narzędzie do pokrycia, ustawiając `test.coverage.provider` na `v8` lub `istanbul`:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8' // lub 'istanbul'
    },
  },
})
```

Gdy uruchomisz proces Vitest, poprosi cię o automatyczną instalację odpowiedniego pakietu wsparcia.

Lub jeśli wolisz zainstalować je ręcznie:

::: code-group
```bash [v8]
npm i -D @vitest/coverage-v8
```
```bash [istanbul]
npm i -D @vitest/coverage-istanbul
```
:::

## Dostawca V8

::: info
Poniższy opis pokrycia V8 jest specyficzny dla Vitest i nie dotyczy innych runnerów testów.
Od wersji `v3.2.0` Vitest używa [remapowania pokrycia opartego na AST](/blog/vitest-3-2#coverage-v8-ast-aware-remapping) dla pokrycia V8, co produkuje identyczne raporty pokrycia jak Istanbul.

To pozwala użytkownikom mieć szybkość pokrycia V8 z dokładnością pokrycia Istanbul.
:::

Domyślnie Vitest używa dostawcy pokrycia `'v8'`.
Ten dostawca wymaga środowiska uruchomieniowego Javascript zaimplementowanego na [silniku V8](https://v8.dev/), takiego jak NodeJS, Deno lub jakiekolwiek przeglądarki oparte na Chromium, takie jak Google Chrome.

Zbieranie pokrycia jest wykonywane podczas runtime przez instruowanie V8 za pomocą [`node:inspector`](https://nodejs.org/api/inspector.html) i [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/tot/Profiler/) w przeglądarkach. Pliki źródłowe użytkownika mogą być wykonywane bez żadnych kroków pre-instrumentacji.

- ✅ Zalecana opcja do użycia
- ✅ Brak kroku pre-transpilacji. Pliki testowe mogą być wykonywane bez zmian.
- ✅ Szybszy czas wykonania niż Istanbul.
- ✅ Niższe zużycie pamięci niż Istanbul.
- ✅ Dokładność raportu pokrycia jest tak dobra jak z Istanbul ([od Vitest `v3.2.0`](/blog/vitest-3-2#coverage-v8-ast-aware-remapping)).
- ⚠️ W niektórych przypadkach może być wolniejszy niż Istanbul, np. przy ładowaniu wielu różnych modułów. V8 nie wspiera ograniczania zbierania pokrycia do konkretnych modułów.
- ⚠️ Istnieją drobne ograniczenia ustawione przez silnik V8. Zobacz [`ast-v8-to-istanbul` | Ograniczenia](https://github.com/AriPerkkio/ast-v8-to-istanbul?tab=readme-ov-file#limitations).
- ❌ Nie działa w środowiskach, które nie używają V8, takich jak Firefox lub Bun. Lub w środowiskach, które nie eksponują pokrycia V8 przez profiler, takich jak Cloudflare Workers.

<script setup>
import ArrowDown from '../.vitepress/components/ArrowDown.vue'
import Box from '../.vitepress/components/Box.vue'
</script>

<div style="display: flex; flex-direction: column; align-items: center; padding: 2rem 0; max-width: 20rem;">
  <Box>Plik testowy</Box>
  <ArrowDown />
  <Box>Włącz zbieranie pokrycia runtime V8</Box>
  <ArrowDown />
  <Box>Uruchom plik</Box>
  <ArrowDown />
  <Box>Zbierz wyniki pokrycia z V8</Box>
  <ArrowDown />
  <Box>Remapuj wyniki pokrycia do plików źródłowych</Box>
  <ArrowDown />
  <Box>Raport pokrycia</Box>
</div>

## Dostawca Istanbul

[Narzędzia pokrycia kodu Istanbul](https://istanbul.js.org/) istnieją od 2012 roku i są bardzo dobrze przetestowane w boju.
Ten dostawca działa na dowolnym środowisku uruchomieniowym Javascript, ponieważ śledzenie pokrycia odbywa się przez instrumentację plików źródłowych użytkownika.

W praktyce instrumentacja plików źródłowych oznacza dodawanie dodatkowego Javascriptu w plikach użytkownika:

```js
// Uproszczony przykład liczników pokrycia gałęzi i funkcji
const coverage = { // [!code ++]
  branches: { 1: [0, 0] }, // [!code ++]
  functions: { 1: 0 }, // [!code ++]
} // [!code ++]

export function getUsername(id) {
  // Pokrycie funkcji zwiększone, gdy to jest wywoływane  // [!code ++]
  coverage.functions['1']++ // [!code ++]

  if (id == null) {
    // Pokrycie gałęzi zwiększone, gdy to jest wywoływane  // [!code ++]
    coverage.branches['1'][0]++ // [!code ++]

    throw new Error('User ID is required')
  }
  // Niejawne pokrycie else zwiększone, gdy warunek if nie jest spełniony  // [!code ++]
  coverage.branches['1'][1]++ // [!code ++]

  return database.getUser(id)
}

globalThis.__VITEST_COVERAGE__ ||= {} // [!code ++]
globalThis.__VITEST_COVERAGE__[filename] = coverage // [!code ++]
```

- ✅ Działa na dowolnym środowisku uruchomieniowym Javascript
- ✅ Szeroko używany i przetestowany w boju przez ponad 13 lat.
- ✅ W niektórych przypadkach szybszy niż V8. Instrumentacja pokrycia może być ograniczona do konkretnych plików, w przeciwieństwie do V8, gdzie wszystkie moduły są instrumentowane.
- ❌ Wymaga kroku pre-instrumentacji
- ❌ Prędkość wykonania jest wolniejsza niż V8 z powodu narzutu instrumentacji
- ❌ Instrumentacja zwiększa rozmiary plików
- ❌ Zużycie pamięci jest wyższe niż V8

<div style="display: flex; flex-direction: column; align-items: center; padding: 2rem 0; max-width: 20rem;">
  <Box>Plik testowy</Box>
  <ArrowDown />
  <Box>Pre-instrumentacja z Babel</Box>
  <ArrowDown />
  <Box>Uruchom plik</Box>
  <ArrowDown />
  <Box>Zbierz wyniki pokrycia z zakresu Javascript</Box>
  <ArrowDown />
  <Box>Remapuj wyniki pokrycia do plików źródłowych</Box>
  <ArrowDown />
  <Box>Raport pokrycia</Box>
</div>

## Konfiguracja pokrycia

::: tip
Wszystkie opcje pokrycia są wymienione w [Referencji konfiguracji pokrycia](/config/#coverage).
:::

Aby testować z włączonym pokryciem, możesz przekazać flagę `--coverage` w CLI lub ustawić `coverage.enabled` w `vitest.config.ts`:

::: code-group
```json [package.json]
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```
```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      enabled: true
    },
  },
})
```
:::

## Włączanie i wykluczanie plików z raportu pokrycia

Możesz zdefiniować, jakie pliki są pokazywane w raporcie pokrycia, konfigurując [`coverage.include`](/config/#coverage-include) i [`coverage.exclude`](/config/#coverage-exclude).

Domyślnie Vitest pokaże tylko pliki, które zostały zaimportowane podczas uruchomienia testu.
Aby uwzględnić niepokryte pliki w raporcie, musisz skonfigurować [`coverage.include`](/config/#coverage-include) z wzorcem, który wybierze twoje pliki źródłowe:

::: code-group
```ts [vitest.config.ts] {6}
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      include: ['src/**/*.{ts,tsx}']
    },
  },
})
```
```sh [Pokryte pliki]
├── src
│   ├── components
│   │   └── counter.tsx   # [!code ++]
│   ├── mock-data
│   │   ├── products.json # [!code error]
│   │   └── users.json    # [!code error]
│   └── utils
│       ├── formatters.ts # [!code ++]
│       ├── time.ts       # [!code ++]
│       └── users.ts      # [!code ++]
├── test
│   └── utils.test.ts     # [!code error]
│
├── package.json          # [!code error]
├── tsup.config.ts        # [!code error]
└── vitest.config.ts      # [!code error]
```
:::

Aby wykluczyć pliki pasujące do `coverage.include`, możesz zdefiniować dodatkowe [`coverage.exclude`](/config/#coverage-exclude):

::: code-group
```ts [vitest.config.ts] {7}
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      include: ['src/**/*.{ts,tsx}'],
      exclude: ['**/utils/users.ts']
    },
  },
})
```
```sh [Pokryte pliki]
├── src
│   ├── components
│   │   └── counter.tsx   # [!code ++]
│   ├── mock-data
│   │   ├── products.json # [!code error]
│   │   └── users.json    # [!code error]
│   └── utils
│       ├── formatters.ts # [!code ++]
│       ├── time.ts       # [!code ++]
│       └── users.ts      # [!code error]
├── test
│   └── utils.test.ts     # [!code error]
│
├── package.json          # [!code error]
├── tsup.config.ts        # [!code error]
└── vitest.config.ts      # [!code error]
```
:::

## Niestandardowy reporter pokrycia

Możesz używać niestandardowych reporterów pokrycia, przekazując nazwę pakietu lub ścieżkę bezwzględną w `test.coverage.reporter`:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      reporter: [
        // Określ reporter używając nazwy pakietu NPM
        ['@vitest/custom-coverage-reporter', { someOption: true }],

        // Określ reporter używając lokalnej ścieżki
        '/absolute/path/to/custom-reporter.cjs',
      ],
    },
  },
})
```

Niestandardowe reportery są ładowane przez Istanbul i muszą odpowiadać jego interfejsowi reportera. Zobacz [implementację wbudowanych reporterów](https://github.com/istanbuljs/istanbuljs/tree/master/packages/istanbul-reports/lib) jako referencję.

```js [custom-reporter.cjs]
const { ReportBase } = require('istanbul-lib-report')

module.exports = class CustomReporter extends ReportBase {
  constructor(opts) {
    super()

    // Opcje przekazane z konfiguracji są dostępne tutaj
    this.file = opts.file
  }

  onStart(root, context) {
    this.contentWriter = context.writer.writeFile(this.file)
    this.contentWriter.println('Początek niestandardowego raportu pokrycia')
  }

  onEnd() {
    this.contentWriter.println('Koniec niestandardowego raportu pokrycia')
    this.contentWriter.close()
  }
}
```

## Niestandardowy dostawca pokrycia

Możliwe jest również dostarczenie własnego niestandardowego dostawcy pokrycia przez przekazanie `'custom'` w `test.coverage.provider`:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'custom',
      customProviderModule: 'my-custom-coverage-provider'
    },
  },
})
```

Niestandardowi dostawcy wymagają opcji `customProviderModule`, która jest nazwą modułu lub ścieżką, skąd załadować `CoverageProviderModule`. Musi eksportować obiekt implementujący `CoverageProviderModule` jako domyślny eksport:

```ts [my-custom-coverage-provider.ts]
import type {
  CoverageProvider,
  CoverageProviderModule,
  ResolvedCoverageOptions,
  Vitest
} from 'vitest'

const CustomCoverageProviderModule: CoverageProviderModule = {
  getProvider(): CoverageProvider {
    return new CustomCoverageProvider()
  },

  // Implementuje resztę CoverageProviderModule ...
}

class CustomCoverageProvider implements CoverageProvider {
  name = 'custom-coverage-provider'
  options!: ResolvedCoverageOptions

  initialize(ctx: Vitest) {
    this.options = ctx.config.coverage
  }

  // Implementuje resztę CoverageProvider ...
}

export default CustomCoverageProviderModule
```

Proszę odnieść się do definicji typów po więcej szczegółów.

## Ignorowanie kodu

Obaj dostawcy pokrycia mają swoje własne sposoby ignorowania kodu z raportów pokrycia:

- [`v8`](https://github.com/AriPerkkio/ast-v8-to-istanbul?tab=readme-ov-file#ignoring-code)
- [`istanbul`](https://github.com/istanbuljs/nyc#parsing-hints-ignoring-lines)

Podczas używania TypeScript kody źródłowe są transpilowane za pomocą `esbuild`, który usuwa wszystkie komentarze z kodów źródłowych ([esbuild#516](https://github.com/evanw/esbuild/issues/516)).
Komentarze, które są uważane za [legalne komentarze](https://esbuild.github.io/api/#legal-comments), są zachowane.

Możesz dołączyć słowo kluczowe `@preserve` w podpowiedzi ignorowania.
Uważaj, że te podpowiedzi ignorowania mogą teraz być również dołączone w końcowym buildzie produkcyjnym.

```diff
-/* istanbul ignore if */
+/* istanbul ignore if -- @preserve */
if (condition) {

-/* v8 ignore if */
+/* v8 ignore if -- @preserve */
if (condition) {
```

### Przykłady

::: code-group

```ts [if else]
/* v8 ignore if -- @preserve */
if (parameter) { // [!code error]
  console.log('Ignorowane') // [!code error]
} // [!code error]
else {
  console.log('Włączone')
}

/* v8 ignore else -- @preserve */
if (parameter) {
  console.log('Włączone')
}
else { // [!code error]
  console.log('Ignorowane') // [!code error]
} // [!code error]
```

```ts [next node]
/* v8 ignore next -- @preserve */
console.log('Ignorowane') // [!code error]
console.log('Włączone')

/* v8 ignore next -- @preserve */
function ignored() { // [!code error]
  console.log('wszystkie') // [!code error]
  // [!code error]
  console.log('linie') // [!code error]
  // [!code error]
  console.log('są') // [!code error]
  // [!code error]
  console.log('ignorowane') // [!code error]
} // [!code error]

/* v8 ignore next -- @preserve */
class Ignored { // [!code error]
  ignored() {} // [!code error]
  alsoIgnored() {} // [!code error]
} // [!code error]

/* v8 ignore next -- @preserve */
condition // [!code error]
  ? console.log('ignorowane') // [!code error]
  : console.log('też ignorowane') // [!code error]
```

```ts [try catch]
/* v8 ignore next -- @preserve */
try { // [!code error]
  console.log('Ignorowane') // [!code error]
} // [!code error]
catch (error) { // [!code error]
  console.log('Ignorowane') // [!code error]
} // [!code error]

try {
  console.log('Włączone')
}
catch (error) {
  /* v8 ignore next -- @preserve */
  console.log('Ignorowane') // [!code error]
  /* v8 ignore next -- @preserve */
  console.log('Ignorowane') // [!code error]
}

// Wymaga rolldown-vite z powodu braku wsparcia esbuild.
// Zobacz https://vite.dev/guide/rolldown.html#how-to-try-rolldown
try {
  console.log('Włączone')
}
catch (error) /* v8 ignore next */ { // [!code error]
  console.log('Ignorowane') // [!code error]
} // [!code error]
```

```ts [switch case]
switch (type) {
  case 1:
    return 'Włączone'

  /* v8 ignore next -- @preserve */
  case 2: // [!code error]
    return 'Ignorowane' // [!code error]

  case 3:
    return 'Włączone'

  /* v8 ignore next -- @preserve */
  default: // [!code error]
    return 'Ignorowane' // [!code error]
}
```

```ts [whole file]
/* v8 ignore file -- @preserve */
export function ignored() { // [!code error]
  return 'Cały plik jest ignorowany'// [!code error]
}// [!code error]
```
:::

## Wydajność pokrycia

Jeśli generowanie pokrycia kodu jest wolne w twoim projekcie, zobacz [Profilowanie wydajności testów | Pokrycie kodu](/guide/profiling-test-performance.html#code-coverage).

## Vitest UI

Możesz sprawdzić swój raport pokrycia w [Vitest UI](/guide/ui).

Vitest UI włączy raport pokrycia, gdy jest jawnie włączony i obecny jest reporter html coverage, w przeciwnym razie nie będzie dostępny:
- włącz `coverage.enabled=true` w swoim pliku konfiguracyjnym lub uruchom Vitest z flagą `--coverage.enabled=true`
- dodaj `html` do listy `coverage.reporter`: możesz również włączyć opcję `subdir`, aby umieścić raport pokrycia w podkatalogu

<img alt="aktywacja html coverage w Vitest UI" img-light src="/vitest-ui-show-coverage-light.png">
<img alt="aktywacja html coverage w Vitest UI" img-dark src="/vitest-ui-show-coverage-dark.png">

<img alt="html coverage w Vitest UI" img-light src="/ui-coverage-1-light.png">
<img alt="html coverage w Vitest UI" img-dark src="/ui-coverage-1-dark.png">
