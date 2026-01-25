# Mockowanie zmiennych globalnych

Możesz mockować zmienne globalne, które nie są obecne w `jsdom` lub `node`, używając helpera [`vi.stubGlobal`](/api/vi#vi-stubglobal). Umieści on wartość zmiennej globalnej w obiekcie `globalThis`.

Domyślnie Vitest nie resetuje tych zmiennych globalnych, ale możesz włączyć opcję [`unstubGlobals`](/config/#unstubglobals) w swojej konfiguracji, aby przywrócić oryginalne wartości po każdym teście, lub ręcznie wywołać [`vi.unstubAllGlobals()`](/api/vi#vi-unstuballglobals).

```ts
import { vi } from 'vitest'

const IntersectionObserverMock = vi.fn(class {
  disconnect = vi.fn()
  observe = vi.fn()
  takeRecords = vi.fn()
  unobserve = vi.fn()
})

vi.stubGlobal('IntersectionObserver', IntersectionObserverMock)

// teraz możesz uzyskać do niego dostęp jako `IntersectionObserver` lub `window.IntersectionObserver`
```
