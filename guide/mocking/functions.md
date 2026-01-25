# Mockowanie funkcji

Mockowanie funkcji można podzielić na dwie różne kategorie: szpiegowanie i mockowanie.

Jeśli musisz obserwować zachowanie metody na obiekcie, możesz użyć [`vi.spyOn()`](/api/vi#vi-spyon), aby utworzyć szpiega, który śledzi wywołania tej metody.

Jeśli musisz przekazać niestandardową implementację funkcji jako argument lub utworzyć nową mockowaną jednostkę, możesz użyć [`vi.fn()`](/api/vi#vi-fn), aby utworzyć funkcję mockującą.

Zarówno `vi.spyOn`, jak i `vi.fn` współdzielą te same metody.

## Przykład

```js
import { afterEach, describe, expect, it, vi } from 'vitest'

const messages = {
  items: [
    { message: 'Prosta wiadomość testowa', from: 'Testman' },
    // ...
  ],
  addItem(item) {
    messages.items.push(item)
    messages.callbacks.forEach(callback => callback(item))
  },
  onItem(callback) {
    messages.callbacks.push(callback)
  },
  getLatest, // może być również `getter lub setter jeśli wspierane`
}

function getLatest(index = messages.items.length - 1) {
  return messages.items[index]
}

it('powinien pobrać najnowszą wiadomość ze szpiegiem', () => {
  const spy = vi.spyOn(messages, 'getLatest')
  expect(spy.getMockName()).toEqual('getLatest')

  expect(messages.getLatest()).toEqual(
    messages.items[messages.items.length - 1],
  )

  expect(spy).toHaveBeenCalledTimes(1)

  spy.mockImplementationOnce(() => 'access-restricted')
  expect(messages.getLatest()).toEqual('access-restricted')

  expect(spy).toHaveBeenCalledTimes(2)
})

it('przekazywanie mocka', () => {
  const callback = vi.fn()
  messages.onItem(callback)

  messages.addItem({ message: 'Kolejna wiadomość testowa', from: 'Testman' })
  expect(callback).toHaveBeenCalledWith({
    message: 'Kolejna wiadomość testowa',
    from: 'Testman',
  })
})
```
