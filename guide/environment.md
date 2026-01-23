---
title: Środowisko testowe | Przewodnik
---

# Środowisko testowe

Vitest zapewnia opcję [`environment`](/config/#environment) do uruchamiania kodu w określonym środowisku. Możesz modyfikować zachowanie środowiska za pomocą opcji [`environmentOptions`](/config/#environmentoptions).

Domyślnie możesz używać tych środowisk:

- `node` to domyślne środowisko
- `jsdom` emuluje środowisko przeglądarki, dostarczając Browser API, używa pakietu [`jsdom`](https://github.com/jsdom/jsdom)
- `happy-dom` emuluje środowisko przeglądarki, dostarczając Browser API, uważane za szybsze niż jsdom, ale brakuje mu niektórych API, używa pakietu [`happy-dom`](https://github.com/capricorn86/happy-dom)
- `edge-runtime` emuluje [edge-runtime](https://edge-runtime.vercel.app/) Vercel, używa pakietu [`@edge-runtime/vm`](https://www.npmjs.com/package/@edge-runtime/vm)

::: info
Podczas używania środowisk `jsdom` lub `happy-dom`, Vitest stosuje te same zasady co Vite przy importowaniu [CSS](https://vitejs.dev/guide/features.html#css) i [zasobów](https://vitejs.dev/guide/features.html#static-assets). Jeśli importowanie zewnętrznej zależności kończy się błędem `unknown extension .css`, musisz ręcznie zinlineować cały łańcuch importów, dodając wszystkie pakiety do [`server.deps.inline`](/config/#server-deps-inline). Na przykład, jeśli błąd występuje w `package-3` w tym łańcuchu importów: `kod źródłowy -> package-1 -> package-2 -> package-3`, musisz dodać wszystkie trzy pakiety do `server.deps.inline`.

`require` CSS i zasobów wewnątrz zewnętrznych zależności jest rozwiązywane automatycznie.
:::

::: warning
"Środowiska" istnieją tylko podczas uruchamiania testów w Node.js.

`browser` nie jest uważany za środowisko w Vitest. Jeśli chcesz uruchomić część swoich testów używając [Trybu Przeglądarki](/guide/browser/), możesz utworzyć [projekt testowy](/guide/browser/#projects-config).
:::

## Środowiska dla konkretnych plików

Ustawiając opcję `environment` w swojej konfiguracji, będzie ona stosowana do wszystkich plików testowych w projekcie. Aby mieć bardziej szczegółową kontrolę, możesz użyć komentarzy kontrolnych do określenia środowiska dla konkretnych plików. Komentarze kontrolne to komentarze zaczynające się od `@vitest-environment`, po których następuje nazwa środowiska:

```ts
// @vitest-environment jsdom

import { expect, test } from 'vitest'

test('test', () => {
  expect(typeof window).not.toBe('undefined')
})
```

## Niestandardowe środowisko

Możesz utworzyć własny pakiet, aby rozszerzyć środowisko Vitest. Aby to zrobić, utwórz pakiet o nazwie `vitest-environment-${name}` lub określ ścieżkę do prawidłowego pliku JS/TS. Ten pakiet powinien eksportować obiekt o kształcie `Environment`:

```ts
import type { Environment } from 'vitest/environments'

export default <Environment>{
  name: 'custom',
  viteEnvironment: 'ssr',
  // opcjonalne - tylko jeśli obsługujesz pulę "experimental-vm"
  async setupVM() {
    const vm = await import('node:vm')
    const context = vm.createContext()
    return {
      getVmContext() {
        return context
      },
      teardown() {
        // wywoływane po uruchomieniu wszystkich testów z tym środowiskiem
      }
    }
  },
  setup() {
    // niestandardowa konfiguracja
    return {
      teardown() {
        // wywoływane po uruchomieniu wszystkich testów z tym środowiskiem
      }
    }
  }
}
```

::: warning
Vitest wymaga opcji `viteEnvironment` na obiekcie środowiska (domyślnie przechodzi na nazwę środowiska Vitest). Powinna być równa `ssr`, `client` lub dowolnej niestandardowej nazwie [środowiska Vite](https://vite.dev/guide/api-environment). Ta wartość określa, które środowisko jest używane do przetwarzania pliku.
:::

Masz również dostęp do domyślnych środowisk Vitest przez wpis `vitest/environments`:

```ts
import { builtinEnvironments, populateGlobal } from 'vitest/environments'

console.log(builtinEnvironments) // { jsdom, happy-dom, node, edge-runtime }
```

Vitest zapewnia również funkcję narzędziową `populateGlobal`, która może być używana do przenoszenia właściwości z obiektu do globalnej przestrzeni nazw:

```ts
interface PopulateOptions {
  // czy funkcje nieklasowe powinny być powiązane z globalną przestrzenią nazw
  bindFunctions?: boolean
}

interface PopulateResult {
  // lista wszystkich kluczy, które zostały skopiowane, nawet jeśli wartość nie istnieje na oryginalnym obiekcie
  keys: Set<string>
  // mapa oryginalnego obiektu, który mógł zostać nadpisany kluczami
  // możesz zwrócić te wartości wewnątrz funkcji `teardown`
  originals: Map<string | symbol, any>
}

export function populateGlobal(global: any, original: any, options: PopulateOptions): PopulateResult
```
