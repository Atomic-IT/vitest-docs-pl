---
title: Testowanie w źródle | Przewodnik
---

# Testowanie w źródle

Vitest zapewnia sposób na uruchamianie testów wewnątrz twojego kodu źródłowego obok implementacji, podobnie do [testów modułowych Rust](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-module-and-cfgtest).

To sprawia, że testy współdzielą ten sam closure co implementacje i mogą testować prywatne stany bez eksportowania. Jednocześnie przynosi to bliższą pętlę feedbacku dla rozwoju.

::: warning
Ten przewodnik wyjaśnia, jak pisać testy wewnątrz twojego kodu źródłowego. Jeśli musisz pisać testy w oddzielnych plikach testowych, postępuj zgodnie z [przewodnikiem "Pisanie testów"](/guide/#writing-tests).
:::

## Konfiguracja

Aby zacząć, umieść blok `if (import.meta.vitest)` na końcu twojego pliku źródłowego i napisz wewnątrz kilka testów. Na przykład:

```ts [src/index.ts]
// implementacja
export function add(...args: number[]) {
  return args.reduce((a, b) => a + b, 0)
}

// zestawy testów w źródle
if (import.meta.vitest) {
  const { it, expect } = import.meta.vitest
  it('add', () => {
    expect(add()).toBe(0)
    expect(add(1)).toBe(1)
    expect(add(1, 2, 3)).toBe(6)
  })
}
```

Zaktualizuj konfigurację `includeSource` dla Vitest, aby objęła pliki w `src/`:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    includeSource: ['src/**/*.{js,ts}'], // [!code ++]
  },
})
```

Następnie możesz zacząć testować!

```bash
$ npx vitest
```

## Build produkcyjny

Dla buildu produkcyjnego musisz ustawić opcje `define` w swoim pliku konfiguracyjnym, pozwalając bundlerowi na eliminację martwego kodu. Na przykład w Vite

```ts [vite.config.ts]
/// <reference types="vitest/config" />

import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    includeSource: ['src/**/*.{js,ts}'],
  },
  define: { // [!code ++]
    'import.meta.vitest': 'undefined', // [!code ++]
  }, // [!code ++]
})
```

### Inne bundlery

::: details Rolldown
```js [rolldown.config.js]
import { defineConfig } from 'rolldown/config'

export default defineConfig({
  transform: {
    define: { // [!code ++]
      'import.meta.vitest': 'undefined', // [!code ++]
    }, // [!code ++]
  },
})
```

Dowiedz się więcej: [Rolldown](https://rolldown.rs/)
:::

::: details Rollup
```js [rollup.config.js]
import replace from '@rollup/plugin-replace' // [!code ++]

export default {
  plugins: [
    replace({ // [!code ++]
      'import.meta.vitest': 'undefined', // [!code ++]
    }) // [!code ++]
  ],
  // inne opcje
}
```

Dowiedz się więcej: [Rollup](https://rollupjs.org/)
:::

::: details unbuild
```js [build.config.js]
import { defineBuildConfig } from 'unbuild'

export default defineBuildConfig({
  replace: { // [!code ++]
    'import.meta.vitest': 'undefined', // [!code ++]
  }, // [!code ++]
  // inne opcje
})
```

Dowiedz się więcej: [unbuild](https://github.com/unjs/unbuild)
:::

::: details webpack
```js [webpack.config.js]
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.DefinePlugin({ // [!code ++]
      'import.meta.vitest': 'undefined', // [!code ++]
    })// [!code ++]
  ],
}
```

Dowiedz się więcej: [webpack](https://webpack.js.org/plugins/define-plugin/)
:::

## TypeScript

Aby uzyskać wsparcie TypeScript dla `import.meta.vitest`, dodaj `vitest/importMeta` do twojego `tsconfig.json`:

```json [tsconfig.json]
{
  "compilerOptions": {
    "types": [
      "vitest/importMeta" // [!code ++]
    ]
  }
}
```

Odwołaj się do [`examples/in-source-test`](https://github.com/vitest-dev/vitest/tree/main/examples/in-source-test) po pełny przykład.

## Uwagi

Ta funkcja może być przydatna dla:

- Testów jednostkowych dla małych funkcji lub narzędzi
- Prototypowania
- Asercji inline

Zaleca się **używanie oddzielnych plików testowych** dla bardziej złożonych testów, takich jak testy komponentów lub E2E.
