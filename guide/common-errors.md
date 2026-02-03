---
title: Częste błędy | Przewodnik
---

# Częste błędy

## Cannot find module './relative-path'

Jeśli otrzymujesz błąd, że moduł nie może być znaleziony, może to oznaczać kilka różnych rzeczy:

- 1. Źle napisałeś ścieżkę. Upewnij się, że ścieżka jest poprawna.

- 2. Możliwe, że polegasz na `baseUrl` w swoim `tsconfig.json`. Vite domyślnie nie bierze pod uwagę `tsconfig.json`, więc możesz potrzebować zainstalować [`vite-tsconfig-paths`](https://www.npmjs.com/package/vite-tsconfig-paths) samodzielnie, jeśli polegasz na tym zachowaniu.

```ts
import { defineConfig } from 'vitest/config'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths()]
})
```

Lub przepisz swoją ścieżkę, aby nie była względna do roota:

```diff
- import helpers from 'src/helpers'
+ import helpers from '../src/helpers'
```

- 3. Upewnij się, że nie masz względnych [aliasów](/config/#alias). Vite traktuje je jako względne do pliku, w którym znajduje się import, zamiast do roota.

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    alias: {
      '@/': './src/', // [!code --]
      '@/': new URL('./src/', import.meta.url).pathname, // [!code ++]
    }
  }
})
```

## Failed to Terminate Worker

Ten błąd może wystąpić, gdy `fetch` NodeJS jest używany z domyślnym [`pool: 'threads'`](/config/#threads). Ten problem jest śledzony w issue [Timeout abort can leave process(es) running in the background #3077](https://github.com/vitest-dev/vitest/issues/3077).

Jako obejście możesz przełączyć się na [`pool: 'forks'`](/config/#forks) lub [`pool: 'vmForks'`](/config/#vmforks).

::: code-group
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    pool: 'forks',
  },
})
```
```bash [CLI]
vitest --pool=forks
```
:::

## Niestandardowe warunki pakietów nie są rozwiązywane

Jeśli używasz niestandardowych warunków w swoim `package.json` [exports](https://nodejs.org/api/packages.html#package-entry-points) lub [subpath imports](https://nodejs.org/api/packages.html#subpath-imports), możesz zauważyć, że Vitest domyślnie nie respektuje tych warunków.

Na przykład, jeśli masz następujące w swoim `package.json`:

```json
{
  "exports": {
    ".": {
      "custom": "./lib/custom.js",
      "import": "./lib/index.js"
    }
  },
  "imports": {
    "#internal": {
      "custom": "./src/internal.js",
      "default": "./lib/internal.js"
    }
  }
}
```

Domyślnie Vitest użyje tylko warunków `import` i `default`. Aby Vitest respektował niestandardowe warunki, musisz skonfigurować [`ssr.resolve.conditions`](https://vite.dev/config/ssr-options#ssr-resolve-conditions) w swojej konfiguracji Vitest:

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  ssr: {
    resolve: {
      conditions: ['custom', 'import', 'default'],
    },
  },
})
```

::: tip Dlaczego `ssr.resolve.conditions` a nie `resolve.conditions`?
Vitest podąża za konwencją konfiguracji Vite:
- [`resolve.conditions`](https://vite.dev/config/shared-options#resolve-conditions) stosuje się do środowiska `client` Vite, które odpowiada trybowi przeglądarki Vitest, jsdom, happy-dom lub niestandardowym środowiskom z `viteEnvironment: 'client'`.
- [`ssr.resolve.conditions`](https://vite.dev/config/ssr-options#ssr-resolve-conditions) stosuje się do środowiska `ssr` Vite, które odpowiada środowisku node Vitest lub niestandardowym środowiskom z `viteEnvironment: 'ssr'`.

Ponieważ Vitest domyślnie używa środowiska `node` (które używa `viteEnvironment: 'ssr'`), rozwiązywanie modułów używa `ssr.resolve.conditions`. Dotyczy to zarówno eksportów pakietów, jak i subpath imports.

Możesz dowiedzieć się więcej o środowiskach Vite i środowiskach Vitest w [`environment`](/config/environment).
:::

## Segfaults i błędy natywnego kodu

Uruchamianie [natywnych modułów NodeJS](https://nodejs.org/api/addons.html) w `pool: 'threads'` może powodować zagadkowe błędy pochodzące z natywnego kodu.

- `Segmentation fault (core dumped)`
- `thread '<unnamed>' panicked at 'assertion failed`
- `Abort trap: 6`
- `internal error: entered unreachable code`

W tych przypadkach natywny moduł prawdopodobnie nie jest zbudowany jako bezpieczny wielowątkowo. Jako obejście możesz przełączyć się na `pool: 'forks'`, który uruchamia przypadki testowe w wielu `node:child_process` zamiast wielu `node:worker_threads`.

::: code-group
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    pool: 'forks',
  },
})
```
```bash [CLI]
vitest --pool=forks
```
:::
