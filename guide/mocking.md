---
title: Mockowanie | Przewodnik
outline: false
---

# Mockowanie

Podczas pisania testów to tylko kwestia czasu, zanim będziesz potrzebować stworzyć "fałszywą" wersję wewnętrznego — lub zewnętrznego — serwisu. Jest to powszechnie nazywane **mockowaniem**. Vitest dostarcza funkcje pomocnicze przez helper `vi`. Możesz go zaimportować z `vitest` lub uzyskać do niego dostęp globalnie, jeśli włączona jest [konfiguracja `global`](/config/#globals).

::: warning
Zawsze pamiętaj o wyczyszczeniu lub przywróceniu mocków przed lub po każdym uruchomieniu testu, aby cofnąć zmiany stanu mocków między uruchomieniami! Zobacz dokumentację [`mockReset`](/api/mock#mockreset) po więcej informacji.
:::

Jeśli nie znasz metod `vi.fn`, `vi.mock` lub `vi.spyOn`, najpierw sprawdź [sekcję API](/api/vi).

Vitest ma obszerną listę przewodników dotyczących mockowania:

- [Mockowanie klas](/guide/mocking/classes.md)
- [Mockowanie dat](/guide/mocking/dates.md)
- [Mockowanie systemu plików](/guide/mocking/file-system.md)
- [Mockowanie funkcji](/guide/mocking/functions.md)
- [Mockowanie zmiennych globalnych](/guide/mocking/globals.md)
- [Mockowanie modułów](/guide/mocking/modules.md)
- [Mockowanie żądań](/guide/mocking/requests.md)
- [Mockowanie timerów](/guide/mocking/timers.md)

Aby szybciej i prościej zacząć z mockowaniem, możesz sprawdzić poniższą ściągawkę.

## Ściągawka

Chcę…

### Mockować eksportowane zmienne
```js [example.js]
export const getter = 'variable'
```
```ts [example.test.ts]
import * as exports from './example.js'

vi.spyOn(exports, 'getter', 'get').mockReturnValue('mocked')
```

::: warning
To nie będzie działać w trybie przeglądarki. Aby obejść ten problem, zobacz [Ograniczenia](/guide/browser/#spying-on-module-exports).
:::

### Mockować eksportowaną funkcję

1. Przykład z `vi.mock`:

::: warning
Nie zapomnij, że wywołanie `vi.mock` jest przenoszone na początek pliku. Zawsze będzie wykonywane przed wszystkimi importami.
:::

```ts [example.js]
export function method() {}
```
```ts
import { method } from './example.js'

vi.mock('./example.js', () => ({
  method: vi.fn()
}))
```

2. Przykład z `vi.spyOn`:
```ts
import * as exports from './example.js'

vi.spyOn(exports, 'method').mockImplementation(() => {})
```

::: warning
Przykład z `vi.spyOn` nie będzie działać w trybie przeglądarki. Aby obejść ten problem, zobacz [Ograniczenia](/guide/browser/#spying-on-module-exports).
:::

### Mockować implementację eksportowanej klasy

1. Przykład z fałszywą `klasą`:
```ts [example.js]
export class SomeClass {}
```
```ts
import { SomeClass } from './example.js'

vi.mock(import('./example.js'), () => {
  const SomeClass = vi.fn(class FakeClass {
    someMethod = vi.fn()
  })
  return { SomeClass }
})
```

2. Przykład z `vi.spyOn`:

```ts
import * as mod from './example.js'

vi.spyOn(mod, 'SomeClass').mockImplementation(class FakeClass {
  someMethod = vi.fn()
})
```

::: warning
Przykład z `vi.spyOn` nie będzie działać w trybie przeglądarki. Aby obejść ten problem, zobacz [Ograniczenia](/guide/browser/#spying-on-module-exports).
:::

### Szpiegować obiekt zwracany z funkcji

1. Przykład z użyciem cache:

```ts [example.js]
export function useObject() {
  return { method: () => true }
}
```

```ts [useObject.js]
import { useObject } from './example.js'

const obj = useObject()
obj.method()
```

```ts [useObject.test.js]
import { useObject } from './example.js'

vi.mock(import('./example.js'), () => {
  let _cache
  const useObject = () => {
    if (!_cache) {
      _cache = {
        method: vi.fn(),
      }
    }
    // teraz za każdym razem, gdy wywoływane jest useObject(),
    // zwróci tę samą referencję obiektu
    return _cache
  }
  return { useObject }
})

const obj = useObject()
// obj.method zostało wywołane wewnątrz some-path
expect(obj.method).toHaveBeenCalled()
```

### Mockować część modułu

```ts
import { mocked, original } from './some-path.js'

vi.mock(import('./some-path.js'), async (importOriginal) => {
  const mod = await importOriginal()
  return {
    ...mod,
    mocked: vi.fn()
  }
})
original() // ma oryginalne zachowanie
mocked() // jest funkcją szpiegującą
```

::: warning
Nie zapomnij, że to tylko [mockuje dostęp _zewnętrzny_](#mocking-pitfalls). W tym przykładzie, jeśli `original` wywołuje `mocked` wewnętrznie, zawsze wywoła funkcję zdefiniowaną w module, nie w fabryce mocka.
:::

### Mockować bieżącą datę

Aby mockować czas `Date`, możesz użyć funkcji pomocniczej `vi.setSystemTime`. Ta wartość **nie** zostanie automatycznie zresetowana między różnymi testami.

Uważaj, że używanie `vi.useFakeTimers` również zmienia czas `Date`.

```ts
const mockDate = new Date(2022, 0, 1)
vi.setSystemTime(mockDate)
const now = new Date()
expect(now.valueOf()).toBe(mockDate.valueOf())
// resetuj mockowany czas
vi.useRealTimers()
```

### Mockować zmienną globalną

Możesz ustawić zmienną globalną przypisując wartość do `globalThis` lub używając helpera [`vi.stubGlobal`](/api/vi#vi-stubglobal). Podczas używania `vi.stubGlobal`, **nie** zostanie automatycznie zresetowana między różnymi testami, chyba że włączysz opcję konfiguracji [`unstubGlobals`](/config/#unstubglobals) lub wywołasz [`vi.unstubAllGlobals`](/api/vi#vi-unstuballglobals).

```ts
vi.stubGlobal('__VERSION__', '1.0.0')
expect(__VERSION__).toBe('1.0.0')
```

### Mockować `import.meta.env`

1. Aby zmienić zmienną środowiskową, możesz po prostu przypisać jej nową wartość.

::: warning
Wartość zmiennej środowiskowej **_nie_** zostanie automatycznie zresetowana między różnymi testami.
:::

```ts
import { beforeEach, expect, it } from 'vitest'

// możesz ją zresetować w hooku beforeEach ręcznie
const originalViteEnv = import.meta.env.VITE_ENV

beforeEach(() => {
  import.meta.env.VITE_ENV = originalViteEnv
})

it('zmienia wartość', () => {
  import.meta.env.VITE_ENV = 'staging'
  expect(import.meta.env.VITE_ENV).toBe('staging')
})
```

2. Jeśli chcesz automatycznie resetować wartość(ci), możesz użyć helpera `vi.stubEnv` z włączoną opcją konfiguracji [`unstubEnvs`](/config/#unstubenvs) (lub ręcznie wywołać [`vi.unstubAllEnvs`](/api/vi#vi-unstuballenvs) w hooku `beforeEach`):

```ts
import { expect, it, vi } from 'vitest'

// przed uruchomieniem testów "VITE_ENV" wynosi "test"
import.meta.env.VITE_ENV === 'test'

it('zmienia wartość', () => {
  vi.stubEnv('VITE_ENV', 'staging')
  expect(import.meta.env.VITE_ENV).toBe('staging')
})

it('wartość jest przywracana przed uruchomieniem kolejnego testu', () => {
  expect(import.meta.env.VITE_ENV).toBe('test')
})
```

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    unstubEnvs: true,
  },
})
```
