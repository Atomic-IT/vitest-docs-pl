---
title: Rozpoczęcie pracy | Przewodnik
---

# Rozpoczęcie pracy

## Przegląd

Vitest (wymawiane jako _"viitest"_) to framework testowy nowej generacji
napędzany przez
Vite.

Więcej o uzasadnieniu projektu dowiesz się w sekcji [Dlaczego Vitest](/guide/why).

## Wypróbuj Vitest Online

Możesz wypróbować Vitest online na [StackBlitz](https://vitest.new). Uruchamia Vitest bezpośrednio w przeglądarce i jest niemal identyczny z lokalną konfiguracją, ale nie wymaga instalowania czegokolwiek na Twoim komputerze.

## Dodawanie Vitest do Twojego Projektu

<CourseLink href="https://vueschool.io/lessons/how-to-install-vitest?friend=vueuse">Naucz się instalować z wideo</CourseLink>

::: code-group
```bash [npm]
npm install -D vitest
```
```bash [yarn]
yarn add -D vitest
```
```bash [pnpm]
pnpm add -D vitest
```
```bash [bun]
bun add -D vitest
```
:::

:::tip
Vitest wymaga Vite >=v6.0.0 oraz Node >=v20.0.0
:::

Zaleca się zainstalowanie kopii `vitest` w Twoim `package.json`, używając jednej z powyższych metod. Jednak jeśli wolisz uruchamiać `vitest` bezpośrednio, możesz użyć `npx vitest` (narzędzie `npx` jest dostarczane z npm i Node.js).

Narzędzie `npx` wykonuje określoną komendę. Domyślnie `npx` najpierw sprawdza, czy komenda istnieje w lokalnych binariach projektu. Jeśli nie zostanie tam znaleziona, `npx` szuka w systemowym `$PATH` i wykonuje ją, jeśli zostanie znaleziona. Jeśli komenda nie zostanie znaleziona w żadnej z tych lokalizacji, `npx` zainstaluje ją w tymczasowej lokalizacji przed wykonaniem.

## Pisanie Testów

Jako przykład napiszemy prosty test weryfikujący wynik funkcji dodającej dwie liczby.

``` js [sum.js]
export function sum(a, b) {
  return a + b
}
```

``` js [sum.test.js]
import { expect, test } from 'vitest'
import { sum } from './sum.js'

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3)
})
```

::: tip
Domyślnie testy muszą zawierać `.test.` lub `.spec.` w nazwie pliku.
:::

Następnie, aby wykonać test, dodaj następującą sekcję do swojego `package.json`:

```json [package.json]
{
  "scripts": {
    "test": "vitest"
  }
}
```

Na koniec uruchom `npm run test`, `yarn test` lub `pnpm test`, w zależności od menedżera pakietów, a Vitest wyświetli następujący komunikat:

```txt
✓ sum.test.js (1)
  ✓ adds 1 + 2 to equal 3

Test Files  1 passed (1)
     Tests  1 passed (1)
  Start at  02:15:44
  Duration  311ms
```

::: warning
Jeśli używasz Bun jako menedżera pakietów, upewnij się, że używasz komendy `bun run test` zamiast `bun test`, w przeciwnym razie Bun uruchomi swój własny runner testowy.
:::

Więcej o użyciu Vitest znajdziesz w sekcji [API](/api/).

## Konfiguracja Vitest

Jedną z głównych zalet Vitest jest jego zunifikowana konfiguracja z Vite. Jeśli istnieje, `vitest` odczyta Twój główny `vite.config.ts`, aby dopasować się do wtyczek i konfiguracji Twojej aplikacji Vite. Na przykład, konfiguracja Vite [resolve.alias](https://vitejs.dev/config/shared-options.html#resolve-alias) i [plugins](https://vitejs.dev/guide/using-plugins.html) będzie działać od razu. Jeśli chcesz inną konfigurację podczas testowania, możesz:

- Utworzyć `vitest.config.ts`, który będzie miał wyższy priorytet
- Przekazać opcję `--config` do CLI, np. `vitest --config ./path/to/vitest.config.ts`
- Użyć `process.env.VITEST` lub właściwości `mode` w `defineConfig` (będzie ustawione na `test`, jeśli nie zostanie nadpisane), aby warunkowo zastosować inną konfigurację w `vite.config.ts`. Zauważ, że jak każda inna zmienna środowiskowa, `VITEST` jest również dostępne na `import.meta.env` w Twoich testach

Vitest obsługuje te same rozszerzenia pliku konfiguracyjnego co Vite: `.js`, `.mjs`, `.cjs`, `.ts`, `.cts`, `.mts`. Vitest nie obsługuje rozszerzenia `.json`.

Jeśli nie używasz Vite jako narzędzia do budowania, możesz skonfigurować Vitest używając właściwości `test` w pliku konfiguracyjnym:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // ...
  },
})
```

::: tip
Nawet jeśli sam nie używasz Vite, Vitest w dużym stopniu polega na nim w procesie transformacji. Z tego powodu możesz również skonfigurować dowolną właściwość opisaną w [dokumentacji Vite](https://vitejs.dev/config/).
:::

Jeśli już używasz Vite, dodaj właściwość `test` w swojej konfiguracji Vite. Będziesz również musiał dodać referencję do typów Vitest używając [dyrektywy triple slash](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html#-reference-types-) na początku pliku konfiguracyjnego.

```ts [vite.config.ts]
/// <reference types="vitest/config" />
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    // ...
  },
})
```

Zobacz listę opcji konfiguracyjnych w [Referencji Konfiguracji](../config/)

::: warning
Jeśli zdecydujesz się mieć dwa osobne pliki konfiguracyjne dla Vite i Vitest, upewnij się, że definiujesz te same opcje Vite w pliku konfiguracyjnym Vitest, ponieważ nadpisze on plik Vite, a nie go rozszerzy. Możesz również użyć metody `mergeConfig` z `vite` lub `vitest/config`, aby scalić konfigurację Vite z konfiguracją Vitest:

:::code-group
```ts [vitest.config.mjs]
import { defineConfig, mergeConfig } from 'vitest/config'
import viteConfig from './vite.config.mjs'

export default mergeConfig(viteConfig, defineConfig({
  test: {
    // ...
  },
}))
```

```ts [vite.config.mjs]
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [Vue()],
})
```

Jednak zalecamy używanie tego samego pliku zarówno dla Vite, jak i Vitest, zamiast tworzenia dwóch osobnych plików.
:::

## Wsparcie dla Projektów

Uruchamiaj różne konfiguracje projektów w ramach tego samego projektu z [Projektami Testowymi](/guide/projects). Możesz zdefiniować listę plików i folderów, które definiują Twoje projekty w pliku `vitest.config`.

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      // możesz użyć listy wzorców glob do zdefiniowania swoich projektów
      // Vitest oczekuje listy plików konfiguracyjnych
      // lub katalogów, w których znajduje się plik konfiguracyjny
      'packages/*',
      'tests/*/vitest.config.{e2e,unit}.ts',
      // możesz nawet uruchomić te same testy,
      // ale z różnymi konfiguracjami w tym samym procesie "vitest"
      {
        test: {
          name: 'happy-dom',
          root: './shared_tests',
          environment: 'happy-dom',
          setupFiles: ['./setup.happy-dom.ts'],
        },
      },
      {
        test: {
          name: 'node',
          root: './shared_tests',
          environment: 'node',
          setupFiles: ['./setup.node.ts'],
        },
      },
    ],
  },
})
```

## Interfejs Wiersza Poleceń

W projekcie, w którym zainstalowany jest Vitest, możesz użyć binarki `vitest` w swoich skryptach npm lub uruchomić ją bezpośrednio za pomocą `npx vitest`. Oto domyślne skrypty npm w projekcie Vitest:

<!-- prettier-ignore -->
```json [package.json]
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

Aby uruchomić testy raz bez obserwowania zmian w plikach, użyj `vitest run`.
Możesz określić dodatkowe opcje CLI, takie jak `--port` lub `--https`. Aby uzyskać pełną listę opcji CLI, uruchom `npx vitest --help` w swoim projekcie.

Dowiedz się więcej o [Interfejsie Wiersza Poleceń](/guide/cli)

## Automatyczna Instalacja Zależności

Vitest poprosi Cię o zainstalowanie niektórych zależności, jeśli nie są jeszcze zainstalowane. Możesz wyłączyć to zachowanie, ustawiając zmienną środowiskową `VITEST_SKIP_INSTALL_CHECKS=1`.

## Integracje z IDE

Udostępniamy również oficjalne rozszerzenie dla Visual Studio Code, aby ulepszyć Twoje doświadczenie testowania z Vitest.

[Zainstaluj z VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=vitest.explorer)

Dowiedz się więcej o [Integracjach z IDE](/guide/ide)

## Przykłady

| Przykład | Źródło | Playground |
|---|---|---|
| `basic` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/basic) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/basic?initialPath=__vitest__/) |
| `fastify` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/fastify) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/fastify?initialPath=__vitest__/) |
| `in-source-test` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/in-source-test) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/in-source-test?initialPath=__vitest__/) |
| `lit` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/lit) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/lit?initialPath=__vitest__/) |
| `vue` | [GitHub](https://github.com/vitest-tests/browser-examples/tree/main/examples/vue) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/vue?initialPath=__vitest__/) |
| `marko` | [GitHub](https://github.com/vitest-tests/browser-examples/tree/main/examples/marko) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/marko?initialPath=__vitest__/) |
| `preact` | [GitHub](https://github.com/vitest-tests/browser-examples/tree/main/examples/preact) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/preact?initialPath=__vitest__/) |
| `qwik`| [Github](https://github.com/vitest-tests/browser-examples/tree/main/examples/qwik) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/qwik?initialPath=__vitest__/) |
| `react` | [GitHub](https://github.com/vitest-tests/browser-examples/tree/main/examples/react) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/react?initialPath=__vitest__/) |
| `solid` | [GitHub](https://github.com/vitest-tests/browser-examples/tree/main/examples/solid) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/solid?initialPath=__vitest__/) |
| `svelte` | [GitHub](https://github.com/vitest-tests/browser-examples/tree/main/examples/svelte) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-tests/browser-examples/tree/main/examples/svelte?initialPath=__vitest__/) |
| `profiling` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/profiling) | Niedostępne |
| `typecheck` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/typecheck) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/typecheck?initialPath=__vitest__/) |
| `projects` | [GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/projects) | [Wypróbuj Online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/projects?initialPath=__vitest__/) |

## Projekty używające Vitest

- [unocss](https://github.com/unocss/unocss)
- [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import)
- [unplugin-vue-components](https://github.com/antfu/unplugin-vue-components)
- [vue](https://github.com/vuejs/core)
- [vite](https://github.com/vitejs/vite)
- [vitesse](https://github.com/antfu/vitesse)
- [vitesse-lite](https://github.com/antfu/vitesse-lite)
- [fluent-vue](https://github.com/demivan/fluent-vue)
- [vueuse](https://github.com/vueuse/vueuse)
- [milkdown](https://github.com/Saul-Mirone/milkdown)
- [gridjs-svelte](https://github.com/iamyuu/gridjs-svelte)
- [spring-easing](https://github.com/okikio/spring-easing)
- [bytemd](https://github.com/bytedance/bytemd)
- [faker](https://github.com/faker-js/faker)
- [million](https://github.com/aidenybai/million)
- [Vitamin](https://github.com/wtchnm/Vitamin)
- [neodrag](https://github.com/PuruVJ/neodrag)
- [svelte-multiselect](https://github.com/janosh/svelte-multiselect)
- [iconify](https://github.com/iconify/iconify)
- [tdesign-vue-next](https://github.com/Tencent/tdesign-vue-next)
- [cz-git](https://github.com/Zhengqbbb/cz-git)

<!--
Dla kontrybutorów:
Obecnie nie przyjmujemy nowych wpisów do tej listy.
Dziękujemy za wybór Vitest!
-->

## Używanie Niewydanych Commitów

Każdy commit na gałęzi main i PR z etykietą `cr-tracked` są publikowane na [pkg.pr.new](https://github.com/stackblitz-labs/pkg.pr.new). Możesz to zainstalować przez `npm i https://pkg.pr.new/vitest@{commit}`.

Jeśli chcesz przetestować własną modyfikację lokalnie, możesz zbudować i połączyć ją samodzielnie (wymagany jest [pnpm](https://pnpm.io/)):

```bash
git clone https://github.com/vitest-dev/vitest.git
cd vitest
pnpm install
cd packages/vitest
pnpm run build
pnpm link --global # możesz użyć preferowanego menedżera pakietów do tego kroku
```

Następnie przejdź do projektu, w którym używasz Vitest i uruchom `pnpm link --global vitest` (lub menedżer pakietów, którego użyłeś do globalnego połączenia `vitest`).

## Społeczność

Jeśli masz pytania lub potrzebujesz pomocy, skontaktuj się ze społecznością na [Discord](https://chat.vitest.dev) i [GitHub Discussions](https://github.com/vitest-dev/vitest/discussions).
