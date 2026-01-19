---
title: Tryb Przeglądarki | Przewodnik
outline: deep
---

# Tryb Przeglądarki {#browser-mode}

Ta strona zawiera informacje o funkcji trybu przeglądarki w API Vitest, która pozwala uruchamiać testy natywnie w przeglądarce, zapewniając dostęp do globalnych obiektów przeglądarki, takich jak window i document.

::: tip
Jeśli szukasz dokumentacji dla `expect`, `vi` lub jakiegokolwiek ogólnego API, takiego jak projekty testowe lub testowanie typów, zapoznaj się z przewodnikiem ["Rozpoczęcie pracy"](/guide/).
:::

<img alt="Vitest UI" img-light src="/ui-browser-1-light.png">
<img alt="Vitest UI" img-dark src="/ui-browser-1-dark.png">

## Instalacja

Dla łatwiejszej konfiguracji możesz użyć komendy `vitest init browser`, aby zainstalować wymagane zależności i utworzyć konfigurację przeglądarki.

::: code-group
```bash [npm]
npx vitest init browser
```
```bash [yarn]
yarn exec vitest init browser
```
```bash [pnpm]
pnpx vitest init browser
```
```bash [bun]
bunx vitest init browser
```
:::

### Ręczna instalacja

Możesz również zainstalować pakiety ręcznie. Vitest zawsze wymaga zdefiniowania dostawcy. Możesz wybrać [`preview`](/config/browser/preview), [`playwright`](/config/browser/playwright) lub [`webdriverio`](/config/browser/webdriverio).

Jeśli chcesz tylko podejrzeć, jak wyglądają Twoje testy, możesz użyć dostawcy `preview`:

::: code-group
```bash [npm]
npm install -D vitest @vitest/browser-preview
```
```bash [yarn]
yarn add -D vitest @vitest/browser-preview
```
```bash [pnpm]
pnpm add -D vitest @vitest/browser-preview
```
```bash [bun]
bun add -D vitest @vitest/browser-preview
```
:::

::: warning
Jednak do uruchamiania testów w CI musisz zainstalować [`playwright`](https://npmjs.com/package/playwright) lub [`webdriverio`](https://www.npmjs.com/package/webdriverio). Zalecamy również przejście na jeden z nich do testowania lokalnego zamiast używania domyślnego dostawcy `preview`, ponieważ opiera się on na symulowaniu zdarzeń zamiast używania Chrome DevTools Protocol.

Jeśli jeszcze nie używasz żadnego z tych narzędzi, zalecamy rozpoczęcie od Playwright, ponieważ obsługuje równoległe wykonywanie, co sprawia, że Twoje testy działają szybciej.

::: tabs key:provider
== Playwright
[Playwright](https://npmjs.com/package/playwright) to framework do testowania i automatyzacji stron internetowych.

::: code-group
```bash [npm]
npm install -D vitest @vitest/browser-playwright
```
```bash [yarn]
yarn add -D vitest @vitest/browser-playwright
```
```bash [pnpm]
pnpm add -D vitest @vitest/browser-playwright
```
```bash [bun]
bun add -D vitest @vitest/browser-playwright
```
== WebdriverIO

[WebdriverIO](https://www.npmjs.com/package/webdriverio) pozwala uruchamiać testy lokalnie używając protokołu WebDriver.

::: code-group
```bash [npm]
npm install -D vitest @vitest/browser-webdriverio
```
```bash [yarn]
yarn add -D vitest @vitest/browser-webdriverio
```
```bash [pnpm]
pnpm add -D vitest @vitest/browser-webdriverio
```
```bash [bun]
bun add -D vitest @vitest/browser-webdriverio
```
:::

## Konfiguracja

Aby aktywować tryb przeglądarki w konfiguracji Vitest, ustaw pole `browser.enabled` na `true` w pliku konfiguracyjnym Vitest. Oto przykładowa konfiguracja używająca pola browser:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      provider: playwright(),
      enabled: true,
      // wymagana jest co najmniej jedna instancja
      instances: [
        { browser: 'chromium' },
      ],
    },
  }
})
```

::: info
Vitest przypisuje port `63315`, aby uniknąć konfliktów z serwerem deweloperskim, pozwalając na uruchamianie obu równolegle. Możesz to zmienić za pomocą opcji [`browser.api`](/config/#browser-api).

CLI nie wyświetla automatycznie URL serwera Vite. Możesz nacisnąć "b", aby wyświetlić URL podczas działania w trybie obserwacji.
:::

Jeśli wcześniej nie używałeś Vite, upewnij się, że masz zainstalowaną i określoną w konfiguracji wtyczkę swojego frameworka. Niektóre frameworki mogą wymagać dodatkowej konfiguracji - sprawdź ich dokumentację związaną z Vite, aby się upewnić.

::: code-group
```ts [react]
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  plugins: [react()],
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [
        { browser: 'chromium' },
      ],
    }
  }
})
```
```ts [vue]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [
        { browser: 'chromium' },
      ],
    }
  }
})
```
```ts [svelte]
import { defineConfig } from 'vitest/config'
import { svelte } from '@sveltejs/vite-plugin-svelte'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  plugins: [svelte()],
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [
        { browser: 'chromium' },
      ],
    }
  }
})
```
```ts [solid]
import { defineConfig } from 'vitest/config'
import solidPlugin from 'vite-plugin-solid'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  plugins: [solidPlugin()],
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [
        { browser: 'chromium' },
      ],
    }
  }
})
```
```ts [marko]
import { defineConfig } from 'vitest/config'
import marko from '@marko/vite'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  plugins: [marko()],
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [
        { browser: 'chromium' },
      ],
    }
  }
})
```
```ts [qwik]
import { defineConfig } from 'vitest/config'
import { qwikVite } from '@builder.io/qwik/optimizer'
import { playwright } from '@vitest/browser-playwright'

// optional, run the tests in SSR mode
import { testSSR } from 'vitest-browser-qwik/ssr-plugin'

export default defineConfig({
  plugins: [testSSR(), qwikVite()],
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [{ browser: 'chromium' }]
    },
  },
})
```
:::

Jeśli musisz uruchomić niektóre testy używając runnera opartego na Node, możesz zdefiniować opcję [`projects`](/guide/projects) z oddzielnymi konfiguracjami dla różnych strategii testowania:

{#projects-config}

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    projects: [
      {
        test: {
          // przykład konwencji opartej na plikach,
          // nie musisz jej przestrzegać
          include: [
            'tests/unit/**/*.{test,spec}.ts',
            'tests/**/*.unit.{test,spec}.ts',
          ],
          name: 'unit',
          environment: 'node',
        },
      },
      {
        test: {
          // przykład konwencji opartej na plikach,
          // nie musisz jej przestrzegać
          include: [
            'tests/browser/**/*.{test,spec}.ts',
            'tests/**/*.browser.{test,spec}.ts',
          ],
          name: 'browser',
          browser: {
            enabled: true,
            provider: playwright(),
            instances: [
              { browser: 'chromium' },
            ],
          },
        },
      },
    ],
  },
})
```

## Typy opcji przeglądarki

Opcja browser w Vitest zależy od dostawcy. Vitest zakończy się niepowodzeniem, jeśli przekażesz `--browser` i nie określisz jej nazwy w pliku konfiguracyjnym. Dostępne opcje:

- `webdriverio` obsługuje te przeglądarki:
  - `firefox`
  - `chrome`
  - `edge`
  - `safari`
- `playwright` obsługuje te przeglądarki:
  - `firefox`
  - `webkit`
  - `chromium`

## Kompatybilność przeglądarek

Vitest używa [serwera deweloperskiego Vite](https://vitejs.dev/guide/#browser-support) do uruchamiania testów, więc obsługujemy tylko funkcje określone w opcji [`esbuild.target`](https://vitejs.dev/config/shared-options.html#esbuild) (domyślnie `esnext`).

Domyślnie Vite celuje w przeglądarki obsługujące natywne [ES Modules](https://caniuse.com/es6-module), natywny [dynamiczny import ESM](https://caniuse.com/es6-module-dynamic-import) i [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta). Dodatkowo wykorzystujemy [`BroadcastChannel`](https://caniuse.com/?search=BroadcastChannel) do komunikacji między iframe'ami:

- Chrome >=87
- Firefox >=78
- Safari >=15.4
- Edge >=88

## Uruchamianie testów

Gdy określisz nazwę przeglądarki w opcji browser, Vitest spróbuje uruchomić określoną przeglądarkę domyślnie używając `preview`, a następnie uruchomi tam testy. Jeśli nie chcesz używać `preview`, możesz skonfigurować niestandardowego dostawcę przeglądarki używając opcji `browser.provider`.

Aby określić przeglądarkę za pomocą CLI, użyj flagi `--browser` z nazwą przeglądarki, na przykład:

```sh
npx vitest --browser=chromium
```

Możesz również przekazać opcje przeglądarki do CLI za pomocą notacji kropkowej:

```sh
npx vitest --browser.headless
```

::: warning
Od Vitest 3.2, jeśli nie masz opcji `browser` w konfiguracji, ale określisz flagę `--browser`, Vitest zakończy się niepowodzeniem, ponieważ nie może założyć, że konfiguracja jest przeznaczona dla przeglądarki, a nie testów Node.js.
:::

Domyślnie Vitest automatycznie otworzy UI przeglądarki do developmentu. Twoje testy będą uruchamiane wewnątrz iframe'a na środku. Możesz skonfigurować viewport, wybierając preferowane wymiary, wywołując `page.viewport` wewnątrz testu lub ustawiając domyślne wartości w [konfiguracji](/config/#browser-viewport).

## Tryb headless

Tryb headless to kolejna opcja dostępna w trybie przeglądarki. W trybie headless przeglądarka działa w tle bez interfejsu użytkownika, co czyni go użytecznym do uruchamiania zautomatyzowanych testów. Opcję headless w Vitest można ustawić na wartość boolean, aby włączyć lub wyłączyć tryb headless.

Podczas używania trybu headless Vitest nie otworzy automatycznie UI. Jeśli chcesz kontynuować używanie UI, ale testy mają działać w trybie headless, możesz zainstalować pakiet [`@vitest/ui`](/guide/ui) i przekazać flagę `--ui` podczas uruchamiania Vitest.

Oto przykładowa konfiguracja włączająca tryb headless:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      provider: playwright(),
      enabled: true,
      headless: true,
    },
  }
})
```

Możesz również ustawić tryb headless używając flagi `--browser.headless` w CLI, na przykład:

```sh
npx vitest --browser.headless
```

W tym przypadku Vitest uruchomi się w trybie headless używając przeglądarki Chrome.

::: warning
Tryb headless nie jest domyślnie dostępny. Musisz użyć dostawcy [`playwright`](https://npmjs.com/package/playwright) lub [`webdriverio`](https://www.npmjs.com/package/webdriverio), aby włączyć tę funkcję.
:::

## Przykłady

Domyślnie nie potrzebujesz żadnych zewnętrznych pakietów do pracy z Trybem Przeglądarki:

```js [example.test.js]
import { expect, test } from 'vitest'
import { page } from 'vitest/browser'
import { render } from './my-render-function.js'

test('properly handles form inputs', async () => {
  render() // montuje elementy DOM

  // Sprawdza stan początkowy.
  await expect.element(page.getByText('Hi, my name is Alice')).toBeInTheDocument()

  // Pobiera węzeł DOM inputa przez zapytanie o powiązaną etykietę.
  const usernameInput = page.getByLabelText(/username/i)

  // Wpisuje imię do inputa. To już waliduje, że input
  // jest wypełniony poprawnie, nie trzeba ręcznie sprawdzać wartości.
  await usernameInput.fill('Bob')

  await expect.element(page.getByText('Hi, my name is Bob')).toBeInTheDocument()
})
```

Jednak Vitest zapewnia również pakiety do renderowania komponentów dla kilku popularnych frameworków od razu:

- [`vitest-browser-vue`](https://github.com/vitest-dev/vitest-browser-vue) do renderowania komponentów [vue](https://vuejs.org)
- [`vitest-browser-svelte`](https://github.com/vitest-dev/vitest-browser-svelte) do renderowania komponentów [svelte](https://svelte.dev)
- [`vitest-browser-react`](https://github.com/vitest-dev/vitest-browser-react) do renderowania komponentów [react](https://react.dev)

Pakiety społeczności są dostępne dla innych frameworków:

- [`vitest-browser-lit`](https://github.com/EskiMojo14/vitest-browser-lit) do renderowania komponentów [lit](https://lit.dev)
- [`vitest-browser-preact`](https://github.com/JoviDeCroock/vitest-browser-preact) do renderowania komponentów [preact](https://preactjs.com)
- [`vitest-browser-qwik`](https://github.com/kunai-consulting/vitest-browser-qwik) do renderowania komponentów [qwik](https://qwik.dev)

Jeśli Twój framework nie jest reprezentowany, możesz stworzyć własny pakiet - to prosta nakładka na renderer frameworka i API `page.elementLocator`. Dodamy link do niego na tej stronie. Upewnij się, że ma nazwę zaczynającą się od `vitest-browser-`.

Oprócz renderowania komponentów i lokalizowania elementów, będziesz również musiał wykonywać asercje. Vitest forkuje bibliotekę [`@testing-library/jest-dom`](https://github.com/testing-library/jest-dom), aby zapewnić szeroki zakres asercji DOM od razu. Przeczytaj więcej w [API Asercji](/api/browser/assertions).

```ts
import { expect } from 'vitest'
import { page } from 'vitest/browser'
// element jest poprawnie wyrenderowany
await expect.element(page.getByText('Hello World')).toBeInTheDocument()
```

Vitest udostępnia [Context API](/api/browser/context) z małym zestawem narzędzi, które mogą być przydatne w testach. Na przykład, jeśli musisz wykonać interakcję, taką jak kliknięcie elementu lub wpisanie tekstu do inputa, możesz użyć `userEvent` z `vitest/browser`. Przeczytaj więcej w [API Interaktywności](/api/browser/interactivity).

```ts
import { page, userEvent } from 'vitest/browser'
await userEvent.fill(page.getByLabelText(/username/i), 'Alice')
// lub po prostu locator.fill
await page.getByLabelText(/username/i).fill('Alice')
```

::: code-group
```ts [vue]
import { render } from 'vitest-browser-vue'
import Component from './Component.vue'

test('properly handles v-model', async () => {
  const screen = render(Component)

  // Sprawdza stan początkowy.
  await expect.element(screen.getByText('Hi, my name is Alice')).toBeInTheDocument()

  // Pobiera węzeł DOM inputa przez zapytanie o powiązaną etykietę.
  const usernameInput = screen.getByLabelText(/username/i)

  // Wpisuje imię do inputa. To już waliduje, że input
  // jest wypełniony poprawnie, nie trzeba ręcznie sprawdzać wartości.
  await usernameInput.fill('Bob')

  await expect.element(screen.getByText('Hi, my name is Bob')).toBeInTheDocument()
})
```
```ts [svelte]
import { render } from 'vitest-browser-svelte'
import { expect, test } from 'vitest'

import Greeter from './greeter.svelte'

test('greeting appears on click', async () => {
  const screen = render(Greeter, { name: 'World' })

  const button = screen.getByRole('button')
  await button.click()
  const greeting = screen.getByText(/hello world/iu)

  await expect.element(greeting).toBeInTheDocument()
})
```
```tsx [react]
import { render } from 'vitest-browser-react'
import Fetch from './fetch'

test('loads and displays greeting', async () => {
  // Renderuje element React do DOM
  const screen = render(<Fetch url="/greeting" />)

  await screen.getByText('Load Greeting').click()
  // czeka przed rzuceniem błędu, jeśli nie może znaleźć elementu
  const heading = screen.getByRole('heading')

  // sprawdza, czy wiadomość alertu jest poprawna
  await expect.element(heading).toHaveTextContent('hello there')
  await expect.element(screen.getByRole('button')).toBeDisabled()
})
```
```ts [lit]
import { render } from 'vitest-browser-lit'
import { html } from 'lit'
import './greeter-button'

test('greeting appears on click', async () => {
  const screen = render(html`<greeter-button name="World"></greeter-button>`)

  const button = screen.getByRole('button')
  await button.click()
  const greeting = screen.getByText(/hello world/iu)

  await expect.element(greeting).toBeInTheDocument()
})
```
```tsx [preact]
import { render } from 'vitest-browser-preact'
import { createElement } from 'preact'
import Greeting from '.Greeting'

test('greeting appears on click', async () => {
  const screen = render(<Greeting />)

  const button = screen.getByRole('button')
  await button.click()
  const greeting = screen.getByText(/hello world/iu)

  await expect.element(greeting).toBeInTheDocument()
})
```
```tsx [qwik]
import { render } from 'vitest-browser-qwik'
import Greeting from './greeting'

test('greeting appears on click', async () => {
  // renderSSR i renderHook są również dostępne
  const screen = render(<Greeting />)

  const button = screen.getByRole('button')
  await button.click()
  const greeting = screen.getByText(/hello world/iu)

  await expect.element(greeting).toBeInTheDocument()
})
```
:::

Vitest nie obsługuje wszystkich frameworków od razu, ale możesz użyć zewnętrznych narzędzi do uruchamiania testów z tymi frameworkami. Zachęcamy również społeczność do tworzenia własnych wrapperów `vitest-browser` - jeśli masz taki, możesz dodać go do powyższych przykładów.

Dla nieobsługiwanych frameworków zalecamy używanie pakietów `testing-library`:

- [`@solidjs/testing-library`](https://testing-library.com/docs/solid-testing-library/intro) do renderowania komponentów [solid](https://www.solidjs.com)
- [`@marko/testing-library`](https://testing-library.com/docs/marko-testing-library/intro) do renderowania komponentów [marko](https://markojs.com)

Możesz również zobaczyć więcej przykładów w repozytorium [`browser-examples`](https://github.com/vitest-tests/browser-examples).

::: warning
`testing-library` dostarcza pakiet `@testing-library/user-event`. Nie zalecamy używania go bezpośrednio, ponieważ symuluje zdarzenia zamiast faktycznie je wyzwalać - zamiast tego użyj [`userEvent`](/api/browser/interactivity) importowanego z `vitest/browser`, który używa Chrome DevTools Protocol lub Webdriver (w zależności od dostawcy) pod maską.
:::

::: code-group
```tsx [solid]
// oparte na API @testing-library/solid
// https://testing-library.com/docs/solid-testing-library/api

import { render } from '@testing-library/solid'

it('uses params', async () => {
  const App = () => (
    <>
      <Route
        path="/ids/:id"
        component={() => (
          <p>
            Id:
            {useParams()?.id}
          </p>
        )}
      />
      <Route path="/" component={() => <p>Start</p>} />
    </>
  )
  const { baseElement } = render(() => <App />, { location: 'ids/1234' })
  const screen = page.elementLocator(baseElement)

  await expect.screen(screen.getByText('Id: 1234')).toBeInTheDocument()
})
```
```ts [marko]
// oparte na API @testing-library/marko
// https://testing-library.com/docs/marko-testing-library/api

import { render, screen } from '@marko/testing-library'
import Greeting from './greeting.marko'

test('renders a message', async () => {
  const { baseElement } = await render(Greeting, { name: 'Marko' })
  const screen = page.elementLocator(baseElement)
  await expect.element(screen.getByText(/Marko/)).toBeInTheDocument()
  expect(container.firstChild).toMatchInlineSnapshot(`
    <h1>Hello, Marko!</h1>
  `)
})
```
:::

## Ograniczenia

### Dialogi blokujące wątek

Podczas używania Vitest Browser ważne jest, aby pamiętać, że dialogi blokujące wątek, takie jak `alert` lub `confirm`, nie mogą być używane natywnie. Dzieje się tak, ponieważ blokują stronę internetową, co oznacza, że Vitest nie może kontynuować komunikacji ze stroną, powodując zawieszenie wykonania.

W takich sytuacjach Vitest zapewnia domyślne mocki z domyślnymi zwracanymi wartościami dla tych API. Zapewnia to, że jeśli użytkownik przypadkowo użyje synchronicznych popup web API, wykonanie się nie zawiesi. Jednak nadal zaleca się, aby użytkownik mockował te web API dla lepszego doświadczenia. Przeczytaj więcej w [Mockowanie](/guide/mocking).

### Szpiegowanie eksportów modułów

Tryb Przeglądarki używa natywnego wsparcia ESM przeglądarki do serwowania modułów. Obiekt przestrzeni nazw modułu jest zapieczętowany i nie może być rekonfigurowany, w przeciwieństwie do testów Node.js, gdzie Vitest może łatać Module Runner. Oznacza to, że nie możesz wywołać `vi.spyOn` na zaimportowanym obiekcie:

```ts
import { vi } from 'vitest'
import * as module from './module.js'

vi.spyOn(module, 'method') // ❌ rzuca błąd
```

Aby obejść to ograniczenie, Vitest obsługuje opcję `{ spy: true }` w `vi.mock('./module.js')`. Spowoduje to automatyczne szpiegowanie każdego eksportu w module bez zastępowania ich fałszywymi.

```ts
import { vi } from 'vitest'
import * as module from './module.js'

vi.mock('./module.js', { spy: true })

vi.mocked(module.method).mockImplementation(() => {
  // ...
})
```

Jednak jedynym sposobem na mockowanie eksportowanych _zmiennych_ jest wyeksportowanie metody, która zmieni wewnętrzną wartość:

::: code-group
```js [module.js]
export let MODE = 'test'
export function changeMode(newMode) {
  MODE = newMode
}
```
```js [module.test.ts]
import { expect } from 'vitest'
import { changeMode, MODE } from './module.js'

changeMode('production')
expect(MODE).toBe('production')
```
:::
