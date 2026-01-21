# Wiele konfiguracji

Możesz określić kilka różnych konfiguracji przeglądarki używając opcji [`browser.instances`](/config/browser/instances).

Główną zaletą używania `browser.instances` zamiast [projektów testowych](/guide/projects) jest ulepszone buforowanie. Każdy projekt będzie używał tego samego serwera Vite, co oznacza, że transformacja plików i [wstępne pakowanie zależności](https://vite.dev/guide/dep-pre-bundling.html) musi nastąpić tylko raz.

## Kilka przeglądarek

Możesz użyć pola `browser.instances` do określenia opcji dla różnych przeglądarek. Na przykład, jeśli chcesz uruchomić te same testy w różnych przeglądarkach, minimalna konfiguracja będzie wyglądać tak:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      headless: true,
      instances: [
        { browser: 'chromium' },
        { browser: 'firefox' },
        { browser: 'webkit' },
      ],
    },
  },
})
```

## Różne konfiguracje

Możesz również określić różne opcje konfiguracji niezależnie od przeglądarki (chociaż instancje _mogą_ również mieć pola `browser`):

::: code-group
```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      headless: true,
      instances: [
        {
          browser: 'chromium',
          name: 'chromium-1',
          setupFiles: ['./ratio-setup.ts'],
          provide: {
            ratio: 1,
          },
        },
        {
          browser: 'chromium',
          name: 'chromium-2',
          provide: {
            ratio: 2,
          },
        },
      ],
    },
  },
})
```
```ts [example.test.ts]
import { expect, inject, test } from 'vitest'
import { globalSetupModifier } from './example.js'

test('ratio works', () => {
  expect(inject('ratio') * globalSetupModifier).toBe(14)
})
```
:::

W tym przykładzie Vitest uruchomi wszystkie testy w przeglądarce `chromium`, ale wykona plik `'./ratio-setup.ts'` tylko w pierwszej konfiguracji i wstrzyknie inną wartość `ratio` w zależności od [pola `provide`](/config/#provide).

::: warning
Zauważ, że musisz zdefiniować niestandardową wartość `name`, jeśli używasz tej samej nazwy przeglądarki, ponieważ w przeciwnym razie Vitest przypisze `browser` jako nazwę projektu.
:::

## Filtrowanie

Możesz filtrować, które projekty uruchomić, za pomocą [flagi `--project`](/guide/cli#project). Vitest automatycznie przypisze nazwę przeglądarki jako nazwę projektu, jeśli nie została przypisana ręcznie. Jeśli główna konfiguracja ma już nazwę, Vitest je połączy: `custom` -> `custom (browser)`.

```shell
$ vitest --project=chromium
```

::: code-group
```ts{6,8} [default]
export default defineConfig({
  test: {
    browser: {
      instances: [
        // name: chromium
        { browser: 'chromium' },
        // name: custom
        { browser: 'firefox', name: 'custom' },
      ]
    }
  }
})
```
```ts{3,7,9} [custom]
export default defineConfig({
  test: {
    name: 'custom',
    browser: {
      instances: [
        // name: custom (chromium)
        { browser: 'chromium' },
        // name: manual
        { browser: 'firefox', name: 'manual' },
      ]
    }
  }
})
```
:::
