---
title: Funkcje | Przewodnik
outline: deep
---

# Funkcje

<script setup>
import FeaturesList from '../.vitepress/components/FeaturesList.vue'
</script>

<FeaturesList class="!gap-1 text-lg" />

<div h-2 />
<CourseLink href="https://vueschool.io/lessons/your-first-test?friend=vueuse">Naucz się pisać swój pierwszy test z wideo</CourseLink>

## Wspólna konfiguracja między testami, developmentem i budowaniem

Konfiguracja Vite, transformery, resolvery i wtyczki. Używaj tej samej konfiguracji z aplikacji do uruchamiania testów.

Dowiedz się więcej w [Konfiguracja Vitest](/guide/#configuring-vitest).

## Tryb obserwacji

```bash
$ vitest
```

Gdy modyfikujesz kod źródłowy lub pliki testów, Vitest inteligentnie przeszukuje graf modułów i uruchamia ponownie tylko powiązane testy, tak jak działa HMR w Vite!

`vitest` uruchamia się w `trybie obserwacji` **domyślnie w środowisku deweloperskim** i w `trybie uruchomienia` w środowisku CI (gdy `process.env.CI` jest obecne) inteligentnie. Możesz użyć `vitest watch` lub `vitest run`, aby jawnie określić pożądany tryb.

Uruchom Vitest z flagą `--standalone`, aby utrzymać go działającego w tle. Nie uruchomi żadnych testów, dopóki się nie zmienią. Vitest nie uruchomi testów, jeśli kod źródłowy zostanie zmieniony, dopóki test importujący źródło nie zostanie uruchomiony.

## Popularne wzorce webowe od razu

Wbudowane wsparcie dla ES Module / TypeScript / JSX / PostCSS

## Wątki

Domyślnie Vitest uruchamia pliki testowe w [wielu procesach](/guide/parallelism) używając [`node:child_process`](https://nodejs.org/api/child_process.html), pozwalając testom działać jednocześnie. Jeśli chcesz jeszcze bardziej przyspieszyć swój zestaw testów, rozważ włączenie `--pool=threads`, aby uruchamiać testy używając [`node:worker_threads`](https://nodejs.org/api/worker_threads.html) (uwaga: niektóre pakiety mogą nie działać z tą konfiguracją).
Aby uruchomić testy w pojedynczym wątku lub procesie, zobacz [`fileParallelism`](/config/#fileParallelism).

Vitest izoluje również środowisko każdego pliku, więc mutacje env w jednym pliku nie wpływają na inne. Izolację można wyłączyć, przekazując `--no-isolate` do CLI (poświęcając poprawność na rzecz wydajności).

## Filtrowanie testów

Vitest zapewnia wiele sposobów na zawężenie testów do uruchomienia, aby przyspieszyć testowanie i skupić się na developmencie.

Dowiedz się więcej o [Filtrowanie testów](/guide/filtering).

## Uruchamianie testów współbieżnie

Użyj `.concurrent` w kolejnych testach, aby uruchomić je równolegle.

```ts
import { describe, it } from 'vitest'

// Dwa testy oznaczone jako concurrent zostaną uruchomione równolegle
describe('suite', () => {
  it('serial test', async () => { /* ... */ })
  it.concurrent('concurrent test 1', async ({ expect }) => { /* ... */ })
  it.concurrent('concurrent test 2', async ({ expect }) => { /* ... */ })
})
```

Jeśli użyjesz `.concurrent` na zestawie, każdy test w nim zostanie uruchomiony równolegle.

```ts
import { describe, it } from 'vitest'

// Wszystkie testy w tym zestawie zostaną uruchomione równolegle
describe.concurrent('suite', () => {
  it('concurrent test 1', async ({ expect }) => { /* ... */ })
  it('concurrent test 2', async ({ expect }) => { /* ... */ })
  it.concurrent('concurrent test 3', async ({ expect }) => { /* ... */ })
})
```

Możesz również używać `.skip`, `.only` i `.todo` z współbieżnymi zestawami i testami. Przeczytaj więcej w [Referencja API](/api/#test-concurrent).

::: warning
Podczas uruchamiania współbieżnych testów, Snapshoty i Asercje muszą używać `expect` z lokalnego [Kontekstu testu](/guide/test-context), aby zapewnić wykrycie właściwego testu.
:::

## Snapshot

Wsparcie dla snapshotów [kompatybilne z Jest](https://jestjs.io/docs/snapshot-testing).

```ts
import { expect, it } from 'vitest'

it('renders correctly', () => {
  const result = render()
  expect(result).toMatchSnapshot()
})
```

Dowiedz się więcej w [Snapshot](/guide/snapshot).

## Kompatybilność Chai i Jest `expect`

[Chai](https://www.chaijs.com/) jest wbudowany dla asercji z API [kompatybilnym z Jest `expect`](https://jestjs.io/docs/expect).

Zauważ, że jeśli używasz bibliotek zewnętrznych, które dodają matchery, ustawienie [`test.globals`](/config/#globals) na `true` zapewni lepszą kompatybilność.

## Mockowanie

[Tinyspy](https://github.com/tinylibs/tinyspy) jest wbudowany do mockowania z API kompatybilnym z `jest` na obiekcie `vi`.

```ts
import { expect, vi } from 'vitest'

const fn = vi.fn()

fn('hello', 1)

expect(vi.isMockFunction(fn)).toBe(true)
expect(fn.mock.calls[0]).toEqual(['hello', 1])

fn.mockImplementation((arg: string) => arg)

fn('world', 2)

expect(fn.mock.results[1].value).toBe('world')
```

Vitest obsługuje zarówno [happy-dom](https://github.com/capricorn86/happy-dom), jak i [jsdom](https://github.com/jsdom/jsdom) do mockowania DOM i API przeglądarki. Nie są dostarczane z Vitest, musisz je zainstalować osobno:

::: code-group
```bash [happy-dom]
$ npm i -D happy-dom
```
```bash [jsdom]
$ npm i -D jsdom
```
:::

Następnie zmień opcję `environment` w pliku konfiguracyjnym:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    environment: 'happy-dom', // lub 'jsdom', 'node'
  },
})
```

Dowiedz się więcej w [Mockowanie](/guide/mocking).

## Pokrycie kodu

Vitest obsługuje natywne pokrycie kodu przez [`v8`](https://v8.dev/blog/javascript-code-coverage) oraz instrumentowane pokrycie kodu przez [`istanbul`](https://istanbul.js.org/).

```json [package.json]
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

Dowiedz się więcej w [Pokrycie kodu](/guide/coverage).

## Testowanie w kodzie źródłowym

Vitest zapewnia również sposób na uruchamianie testów w kodzie źródłowym wraz z implementacją, podobnie jak [testy modułów w Rust](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-module-and-cfgtest).

Dzięki temu testy współdzielą to samo zamknięcie (closure) co implementacje i mogą testować prywatne stany bez eksportowania. Jednocześnie przybliża to pętlę zwrotną dla developmentu.

```ts [src/index.ts]
// implementacja
export function add(...args: number[]): number {
  return args.reduce((a, b) => a + b, 0)
}

// zestawy testów w kodzie źródłowym
if (import.meta.vitest) {
  const { it, expect } = import.meta.vitest
  it('add', () => {
    expect(add()).toBe(0)
    expect(add(1)).toBe(1)
    expect(add(1, 2, 3)).toBe(6)
  })
}
```

Dowiedz się więcej w [Testowanie w kodzie źródłowym](/guide/in-source).

## Benchmarking <Badge type="warning">Eksperymentalne</Badge> {#benchmarking}

Możesz uruchamiać testy wydajnościowe z funkcją [`bench`](/api/#bench) przez [Tinybench](https://github.com/tinylibs/tinybench), aby porównać wyniki wydajności.

```ts [sort.bench.ts]
import { bench, describe } from 'vitest'

describe('sort', () => {
  bench('normal', () => {
    const x = [1, 5, 4, 2, 3]
    x.sort((a, b) => {
      return a - b
    })
  })

  bench('reverse', () => {
    const x = [1, 5, 4, 2, 3]
    x.reverse().sort((a, b) => {
      return a - b
    })
  })
})
```

<img alt="Raport benchmarku" img-dark src="https://github.com/vitest-dev/vitest/assets/4232207/6f0383ea-38ba-4f14-8a05-ab243afea01d">
<img alt="Raport benchmarku" img-light src="https://github.com/vitest-dev/vitest/assets/4232207/efbcb427-ecf1-4882-88de-210cd73415f6">

## Testowanie typów <Badge type="warning">Eksperymentalne</Badge> {#type-testing}

Możesz [pisać testy](/guide/testing-types), aby wychwycić regresje typów. Vitest zawiera pakiet [`expect-type`](https://github.com/mmkal/expect-type), aby zapewnić podobne i łatwe do zrozumienia API.

```ts [types.test-d.ts]
import { assertType, expectTypeOf, test } from 'vitest'
import { mount } from './mount.js'

test('my types work properly', () => {
  expectTypeOf(mount).toBeFunction()
  expectTypeOf(mount).parameter(0).toExtend<{ name: string }>()

  // @ts-expect-error name is a string
  assertType(mount({ name: 42 }))
})
```

## Sharding

Uruchamiaj testy na różnych maszynach używając flag [`--shard`](/guide/cli#shard) i [`--reporter=blob`](/guide/reporters#blob-reporter).
Wszystkie wyniki testów i pokrycia mogą być scalone na końcu pipeline'u CI używając komendy `--merge-reports`:

```bash
vitest --shard=1/2 --reporter=blob --coverage
vitest --shard=2/2 --reporter=blob --coverage
vitest --merge-reports --reporter=junit --coverage
```

Zobacz [`Poprawa wydajności | Sharding`](/guide/improving-performance#sharding), aby uzyskać więcej informacji.

## Zmienne środowiskowe

Vitest automatycznie ładuje wyłącznie zmienne środowiskowe z prefiksem `VITE_` z plików `.env`, aby zachować kompatybilność z testami związanymi z frontendem, zgodnie z [ustaloną konwencją Vite](https://vitejs.dev/guide/env-and-mode.html#env-files). Aby załadować każdą zmienną środowiskową z plików `.env`, możesz użyć metody `loadEnv` importowanej z `vite`:

```ts [vitest.config.ts]
import { loadEnv } from 'vite'
import { defineConfig } from 'vitest/config'

export default defineConfig(({ mode }) => ({
  test: {
    // mode określa, który plik ".env.{mode}" wybrać, jeśli istnieje
    env: loadEnv(mode, process.cwd(), ''),
  },
}))
```

## Nieobsłużone błędy

Domyślnie Vitest przechwytuje i raportuje wszystkie [nieobsłużone odrzucenia](https://developer.mozilla.org/en-US/docs/Web/API/Window/unhandledrejection_event), [nieprzechwycone wyjątki](https://nodejs.org/api/process.html#event-uncaughtexception) (w Node.js) i zdarzenia [error](https://developer.mozilla.org/en-US/docs/Web/API/Window/error_event) (w [przeglądarce](/guide/browser/)).

Możesz wyłączyć to zachowanie, przechwytując je ręcznie. Vitest zakłada, że callback jest obsługiwany przez Ciebie i nie zgłosi błędu.

::: code-group
```ts [setup.node.js]
// w Node.js
process.on('unhandledRejection', () => {
  // Twój własny handler
})

process.on('uncaughtException', () => {
  // Twój własny handler
})
```
```ts [setup.browser.js]
// w przeglądarce
window.addEventListener('error', () => {
  // Twój własny handler
})

window.addEventListener('unhandledrejection', () => {
  // Twój własny handler
})
```
:::

Alternatywnie możesz również ignorować zgłoszone błędy za pomocą opcji [`dangerouslyIgnoreUnhandledErrors`](/config/#dangerouslyignoreunhandlederrors). Vitest nadal będzie je raportować, ale nie wpłyną na wynik testu (kod wyjścia nie zostanie zmieniony).

Jeśli musisz przetestować, że błąd nie został przechwycony, możesz utworzyć test, który wygląda tak:

```ts
test('my function throws uncaught error', async ({ onTestFinished }) => {
  const unhandledRejectionListener = vi.fn()
  process.on('unhandledRejection', unhandledRejectionListener)
  onTestFinished(() => {
    process.off('unhandledRejection', unhandledRejectionListener)
  })

  callMyFunctionThatRejectsError()

  await expect.poll(unhandledRejectionListener).toHaveBeenCalled()
})
```
