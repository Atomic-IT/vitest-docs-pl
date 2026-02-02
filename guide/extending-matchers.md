---
title: Rozszerzanie matcherów | Przewodnik
---

# Rozszerzanie matcherów

Ponieważ Vitest jest kompatybilny zarówno z Chai, jak i Jest, możesz użyć API `chai.use` lub `expect.extend`, w zależności od tego, co wolisz.

Ten przewodnik omówi rozszerzanie matcherów za pomocą `expect.extend`. Jeśli interesuje cię API Chai, sprawdź [ich przewodnik](https://www.chaijs.com/guide/plugins/).

Aby rozszerzyć domyślne matchery, wywołaj `expect.extend` z obiektem zawierającym twoje matchery.

```ts
expect.extend({
  toBeFoo(received, expected) {
    const { isNot } = this
    return {
      // nie zmieniaj swojego "pass" na podstawie isNot. Vitest robi to za ciebie
      pass: received === 'foo',
      message: () => `${received} ${isNot ? 'nie ' : ''}jest foo`
    }
  }
})
```

Jeśli używasz TypeScript, możesz rozszerzyć domyślny interfejs `Assertion` w pliku deklaracji ambient (np. `vitest.d.ts`) poniższym kodem:

::: code-group
```ts [<Version>3.2.0</Version>]
import 'vitest'

interface CustomMatchers<R = unknown> {
  toBeFoo: () => R
}

declare module 'vitest' {
  interface Matchers<T = any> extends CustomMatchers<T> {}
}
```
```ts [<Version>3.0.0</Version>]
import 'vitest'

interface CustomMatchers<R = unknown> {
  toBeFoo: () => R
}

declare module 'vitest' {
  interface Assertion<T = any> extends CustomMatchers<T> {}
  interface AsymmetricMatchersContaining extends CustomMatchers {}
}
```
:::

::: tip
Od Vitest 3.2 możesz rozszerzyć interfejs `Matchers`, aby mieć asercje bezpieczne typowo w `expect.extend`, `expect().*` i metodach `expect.*` jednocześnie. Wcześniej musiałeś definiować oddzielne interfejsy dla każdego z nich.
:::

::: warning
Nie zapomnij dołączyć pliku deklaracji ambient do twojego `tsconfig.json`.
:::

Wartość zwracana przez matcher powinna być kompatybilna z następującym interfejsem:

```ts
interface ExpectationResult {
  pass: boolean
  message: () => string
  // Jeśli przekażesz te wartości, automatycznie pojawią się w diffie, gdy
  // matcher nie przejdzie, więc nie musisz sam drukować diffa
  actual?: unknown
  expected?: unknown
}
```

::: warning
Jeśli tworzysz asynchroniczny matcher, nie zapomnij `await` wyniku (`await expect('foo').toBeFoo()`) w samym teście:

```ts
expect.extend({
  async toBeAsyncAssertion() {
    // ...
  }
})

await expect().toBeAsyncAssertion()
```
:::

Pierwszym argumentem wewnątrz funkcji matchera jest otrzymana wartość (ta wewnątrz `expect(received)`). Reszta to argumenty przekazane bezpośrednio do matchera.

Funkcja matchera ma dostęp do kontekstu `this` z następującymi właściwościami:

### `isNot`

Zwraca true, jeśli matcher został wywołany na `not` (`expect(received).not.toBeFoo()`).

### `promise`

Jeśli matcher został wywołany na `resolved/rejected`, ta wartość będzie zawierać nazwę modyfikatora. W przeciwnym razie będzie pustym stringiem.

### `equals`

To jest funkcja pomocnicza, która pozwala porównać dwie wartości. Zwróci `true`, jeśli wartości są równe, `false` w przeciwnym razie. Ta funkcja jest używana wewnętrznie dla prawie każdego matchera. Domyślnie wspiera obiekty z asymetrycznymi matcherami.

### `utils`

Zawiera zestaw funkcji pomocniczych, których możesz użyć do wyświetlania wiadomości.

Kontekst `this` zawiera również informacje o bieżącym teście. Możesz również uzyskać je wywołując `expect.getState()`. Najbardziej przydatne właściwości to:

### `currentTestName`

Pełna nazwa bieżącego testu (włączając blok describe).

### `task` <Advanced /> <Version type="experimental">4.0.11</Version> {#task}

Zawiera referencję do [zadania runnera `Test`](/api/advanced/runner#tasks), gdy jest dostępne.

::: warning
Podczas używania globalnego `expect` z testami współbieżnymi, `this.task` jest `undefined`. Użyj zamiast tego `context.expect`, aby upewnić się, że `task` jest dostępne w niestandardowych matcherach.
:::

### `testPath`

Ścieżka do bieżącego testu.
