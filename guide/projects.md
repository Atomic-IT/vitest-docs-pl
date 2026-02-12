---
title: Projekty testowe | Przewodnik
---

# Projekty testowe

::: tip Przykładowy projekt

[GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/projects) - [Wypróbuj online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/projects?initialPath=__vitest__/)

:::

::: warning
Ta funkcjonalność jest również znana jako `workspace`. `workspace` jest przestarzały od wersji 3.2 i został zastąpiony konfiguracją `projects`. Są funkcjonalnie identyczne.
:::

Vitest zapewnia sposób na definiowanie wielu konfiguracji projektów w ramach jednego procesu Vitest. Ta funkcja jest szczególnie przydatna dla konfiguracji monorepo, ale może być również używana do uruchamiania testów z różnymi konfiguracjami, takimi jak `resolve.alias`, `plugins`, `test.browser` i więcej.

## Definiowanie projektów

Możesz zdefiniować projekty w swojej głównej [konfiguracji](/config/):

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: ['packages/*'],
  },
})
```

Konfiguracje projektów to konfiguracje inline, pliki lub wzorce glob odnoszące się do projektów. Na przykład, jeśli istnieje folder o nazwie `packages` zawierający projekty, można zdefiniować tablicę w głównej konfiguracji Vitest:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: ['packages/*'],
  },
})
```

Vitest potraktuje każdy folder w `packages` jako oddzielny projekt, nawet jeśli nie zawiera pliku konfiguracyjnego. Jeśli wzorzec glob pasuje do pliku, zwaliduje, czy nazwa zaczyna się od `vitest.config`/`vite.config` lub pasuje do wzorca `(vite|vitest).*.config.*`, aby upewnić się, że jest to plik konfiguracyjny Vitest. Na przykład te pliki konfiguracyjne są prawidłowe:

- `vitest.config.ts`
- `vite.config.js`
- `vitest.unit.config.ts`
- `vite.e2e.config.js`
- `vitest.config.unit.js`
- `vite.config.e2e.js`

Aby wykluczyć foldery i pliki, możesz użyć wzorca negacji:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // uwzględnij wszystkie foldery wewnątrz "packages" z wyjątkiem "excluded"
    projects: [
      'packages/*',
      '!packages/excluded'
    ],
  },
})
```

Jeśli masz zagnieżdżoną strukturę, gdzie niektóre foldery muszą być projektami, ale inne foldery mają swoje własne podfoldery, musisz użyć nawiasów, aby uniknąć dopasowania folderu nadrzędnego:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

// Na przykład, to utworzy projekty:
// packages/a
// packages/b
// packages/business/c
// packages/business/d
// Zauważ, że "packages/business" sam nie jest projektem

export default defineConfig({
  test: {
    projects: [
      // dopasowuje każdy folder wewnątrz "packages" z wyjątkiem "business"
      'packages/!(business)',
      // dopasowuje każdy folder wewnątrz "packages/business"
      'packages/business/*',
    ],
  },
})
```

::: warning
Vitest nie traktuje głównego pliku `vitest.config` jako projektu, chyba że jest jawnie określony w konfiguracji. W konsekwencji, główna konfiguracja wpłynie tylko na globalne opcje, takie jak `reporters` i `coverage`. Zauważ, że Vitest zawsze uruchomi pewne hooki pluginów, takie jak `apply`, `config`, `configResolved` lub `configureServer`, określone w głównym pliku konfiguracyjnym. Vitest również używa tych samych pluginów do wykonywania globalnych setupów i niestandardowego providera pokrycia.
:::

Możesz również odwoływać się do projektów przez ich pliki konfiguracyjne:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: ['packages/*/vitest.config.{e2e,unit}.ts'],
  },
})
```

Ten wzorzec uwzględni tylko projekty z plikiem `vitest.config`, który zawiera `e2e` lub `unit` przed rozszerzeniem.

Możesz również definiować projekty używając konfiguracji inline. Konfiguracja wspiera obie składnie jednocześnie.

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      // dopasowuje każdy folder i plik wewnątrz folderu `packages`
      'packages/*',
      {
        // dodaj "extends: true", aby dziedziczyć opcje z głównej konfiguracji
        extends: true,
        test: {
          include: ['tests/**/*.{browser}.test.{ts,js}'],
          // zaleca się definiowanie nazwy przy używaniu konfiguracji inline
          name: 'happy-dom',
          environment: 'happy-dom',
        }
      },
      {
        test: {
          include: ['tests/**/*.{node}.test.{ts,js}'],
          // kolor etykiety nazwy może być zmieniony
          name: { label: 'node', color: 'green' },
          environment: 'node',
        }
      }
    ]
  }
})
```

::: warning
Wszystkie projekty muszą mieć unikalne nazwy; w przeciwnym razie Vitest wyrzuci błąd. Jeśli nazwa nie jest podana w konfiguracji inline, Vitest przypisze numer. Dla konfiguracji projektów zdefiniowanych składnią glob, Vitest domyślnie użyje właściwości "name" z najbliższego pliku `package.json` lub, jeśli nie istnieje, nazwy folderu.
:::

Projekty nie wspierają wszystkich właściwości konfiguracyjnych. Dla lepszego bezpieczeństwa typów, użyj metody `defineProject` zamiast `defineConfig` w plikach konfiguracyjnych projektów:

```ts twoslash [packages/a/vitest.config.ts]
// @errors: 2769
import { defineProject } from 'vitest/config'

export default defineProject({
  test: {
    environment: 'jsdom',
    // "reporters" nie jest wspierane w konfiguracji projektu,
    // więc pokaże błąd
    reporters: ['json']
  }
})
```

## Uruchamianie testów

Aby uruchamiać testy, zdefiniuj skrypt w swoim głównym `package.json`:

```json [package.json]
{
  "scripts": {
    "test": "vitest"
  }
}
```

Teraz testy można uruchamiać używając menedżera pakietów:

::: code-group
```bash [npm]
npm run test
```
```bash [yarn]
yarn test
```
```bash [pnpm]
pnpm run test
```
```bash [bun]
bun run test
```
:::

Jeśli musisz uruchomić testy tylko wewnątrz pojedynczego projektu, użyj opcji CLI `--project`:

::: code-group
```bash [npm]
npm run test --project e2e
```
```bash [yarn]
yarn test --project e2e
```
```bash [pnpm]
pnpm run test --project e2e
```
```bash [bun]
bun run test --project e2e
```
:::

::: tip
Opcja CLI `--project` może być użyta wielokrotnie, aby filtrować kilka projektów:

::: code-group
```bash [npm]
npm run test --project e2e --project unit
```
```bash [yarn]
yarn test --project e2e --project unit
```
```bash [pnpm]
pnpm run test --project e2e --project unit
```
```bash [bun]
bun run test --project e2e --project unit
```
:::

## Konfiguracja

Żadne z opcji konfiguracyjnych nie są dziedziczone z pliku konfiguracyjnego głównego poziomu. Możesz utworzyć współdzielony plik konfiguracyjny i samodzielnie scalić go z konfiguracją projektu:

```ts [packages/a/vitest.config.ts]
import { defineProject, mergeConfig } from 'vitest/config'
import configShared from '../vitest.shared.js'

export default mergeConfig(
  configShared,
  defineProject({
    test: {
      environment: 'jsdom',
    }
  })
)
```

Dodatkowo możesz użyć opcji `extends`, aby dziedziczyć z konfiguracji głównego poziomu. Wszystkie opcje zostaną scalone.

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    pool: 'threads',
    projects: [
      {
        // odziedziczy opcje z tej konfiguracji, takie jak plugins i pool
        extends: true,
        test: {
          name: 'unit',
          include: ['**/*.unit.test.ts'],
        },
      },
      {
        // nie odziedziczy żadnych opcji z tej konfiguracji
        // to jest domyślne zachowanie
        extends: false,
        test: {
          name: 'integration',
          include: ['**/*.integration.test.ts'],
        },
      },
    ],
  },
})
```

::: danger Niewspierane opcje
Niektóre opcje konfiguracyjne nie są dozwolone w konfiguracji projektu. W szczególności:

- `coverage`: pokrycie jest wykonywane dla całego procesu
- `reporters`: tylko reportery na poziomie głównym mogą być wspierane
- `resolveSnapshotPath`: tylko resolver na poziomie głównym jest respektowany
- wszystkie inne opcje, które nie wpływają na runnery testów

Wszystkie opcje konfiguracyjne, które nie są wspierane wewnątrz konfiguracji projektu, są oznaczone ikoną <CRoot /> obok ich nazwy. Mogą być zdefiniowane tylko raz w głównym pliku konfiguracyjnym.
:::
