# Mockowanie dat

Czasami musisz mieć kontrolę nad datą, aby zapewnić spójność podczas testowania. Vitest używa pakietu [`@sinonjs/fake-timers`](https://github.com/sinonjs/fake-timers) do manipulowania timerami, a także datą systemową. Więcej szczegółów o konkretnym API znajdziesz [tutaj](/api/vi#vi-setsystemtime).

## Przykład

```js
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest'

const businessHours = [9, 17]

function purchase() {
  const currentHour = new Date().getHours()
  const [open, close] = businessHours

  if (currentHour > open && currentHour < close) {
    return { message: 'Success' }
  }

  return { message: 'Error' }
}

describe('proces zakupowy', () => {
  beforeEach(() => {
    // informujemy vitest, że używamy mockowanego czasu
    vi.useFakeTimers()
  })

  afterEach(() => {
    // przywracamy datę po każdym uruchomieniu testu
    vi.useRealTimers()
  })

  it('pozwala na zakupy w godzinach pracy', () => {
    // ustawiamy godzinę w ramach godzin pracy
    const date = new Date(2000, 1, 1, 13)
    vi.setSystemTime(date)

    // dostęp do Date.now() zwróci datę ustawioną powyżej
    expect(purchase()).toEqual({ message: 'Success' })
  })

  it('nie pozwala na zakupy poza godzinami pracy', () => {
    // ustawiamy godzinę poza godzinami pracy
    const date = new Date(2000, 1, 1, 19)
    vi.setSystemTime(date)

    // dostęp do Date.now() zwróci datę ustawioną powyżej
    expect(purchase()).toEqual({ message: 'Error' })
  })
})
```
