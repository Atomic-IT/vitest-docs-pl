# Timery

Gdy testujemy kod, który używa timeoutów lub interwałów, zamiast zmuszać nasze testy do czekania lub przekroczenia limitu czasu, możemy przyspieszyć nasze testy używając "fałszywych" timerów, które mockują wywołania `setTimeout` i `setInterval`.

Zobacz [sekcję API `vi.useFakeTimers`](/api/vi#vi-usefaketimers), aby uzyskać bardziej szczegółowy opis API.

## Przykład

```js
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest'

function executeAfterTwoHours(func) {
  setTimeout(func, 1000 * 60 * 60 * 2) // 2 godziny
}

function executeEveryMinute(func) {
  setInterval(func, 1000 * 60) // 1 minuta
}

const mock = vi.fn(() => console.log('wykonano'))

describe('opóźnione wykonanie', () => {
  beforeEach(() => {
    vi.useFakeTimers()
  })
  afterEach(() => {
    vi.restoreAllMocks()
  })
  it('powinien wykonać funkcję', () => {
    executeAfterTwoHours(mock)
    vi.runAllTimers()
    expect(mock).toHaveBeenCalledTimes(1)
  })
  it('nie powinien wykonać funkcji', () => {
    executeAfterTwoHours(mock)
    // przesunięcie o 2ms nie wywoła funkcji
    vi.advanceTimersByTime(2)
    expect(mock).not.toHaveBeenCalled()
  })
  it('powinien wykonywać co minutę', () => {
    executeEveryMinute(mock)
    vi.advanceTimersToNextTimer()
    expect(mock).toHaveBeenCalledTimes(1)
    vi.advanceTimersToNextTimer()
    expect(mock).toHaveBeenCalledTimes(2)
  })
})
```
