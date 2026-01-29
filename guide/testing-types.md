---
title: Testowanie typów | Przewodnik
---

# Testowanie typów

::: tip Przykładowy projekt

[GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/typecheck) - [Wypróbuj online](https://stackblitz.com/fork/github/vitest-dev/vitest/tree/main/examples/typecheck?initialPath=__vitest__/)

:::

Vitest pozwala pisać testy dla twoich typów, używając składni `expectTypeOf` lub `assertType`. Domyślnie wszystkie testy wewnątrz plików `*.test-d.ts` są uważane za testy typów, ale możesz to zmienić za pomocą opcji konfiguracji [`typecheck.include`](/config/#typecheck-include).

Pod spodem Vitest wywołuje `tsc` lub `vue-tsc`, w zależności od twojej konfiguracji, i parsuje wyniki. Vitest wydrukuje również błędy typów w twoim kodzie źródłowym, jeśli jakieś znajdzie. Możesz to wyłączyć za pomocą opcji konfiguracji [`typecheck.ignoreSourceErrors`](/config/#typecheck-ignoresourceerrors).

Pamiętaj, że Vitest nie uruchamia tych plików, są one tylko statycznie analizowane przez kompilator. Oznacza to, że jeśli użyjesz dynamicznej nazwy lub `test.each` lub `test.for`, nazwa testu nie zostanie ewaluowana - zostanie wyświetlona tak jak jest.

::: warning
Przed Vitest 2.1 twój `typecheck.include` nadpisywał wzorzec `include`, więc twoje testy runtime nie były faktycznie uruchamiane; były tylko sprawdzane pod względem typów.

Od Vitest 2.1, jeśli twoje `include` i `typecheck.include` się nakładają, Vitest zgłosi testy typów i testy runtime jako oddzielne wpisy.
:::

Używanie flag CLI, takich jak `--allowOnly` i `-t`, jest również wspierane dla sprawdzania typów.

```ts [mount.test-d.ts]
import { assertType, expectTypeOf } from 'vitest'
import { mount } from './mount.js'

test('moje typy działają poprawnie', () => {
  expectTypeOf(mount).toBeFunction()
  expectTypeOf(mount).parameter(0).toExtend<{ name: string }>()

  // @ts-expect-error name jest stringiem
  assertType(mount({ name: 42 }))
})
```

Każdy błąd typu wywołany wewnątrz pliku testowego będzie traktowany jako błąd testu, więc możesz użyć dowolnej sztuczki typów, aby testować typy swojego projektu.

Możesz zobaczyć listę możliwych matcherów w [sekcji API](/api/expect-typeof).

## Czytanie błędów

Jeśli używasz API `expectTypeOf`, odwołaj się do [dokumentacji expect-type o komunikatach błędów](https://github.com/mmkal/expect-type#error-messages).

Gdy typy się nie zgadzają, `.toEqualTypeOf` i `.toExtend` używają specjalnego typu pomocniczego do tworzenia komunikatów błędów, które są jak najbardziej użyteczne. Ale jest pewien niuans w ich zrozumieniu. Ponieważ asercje są pisane "płynnie", niepowodzenie powinno być na typie "oczekiwanym", nie na typie "faktycznym" (`expect<Actual>().toEqualTypeOf<Expected>()`). Oznacza to, że błędy typów mogą być trochę mylące - więc ta biblioteka produkuje typ `MismatchInfo`, aby spróbować jawnie określić, jakie jest oczekiwanie. Na przykład:

```ts
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: string }>()
```

Jest asercją, która nie powiedzie się, ponieważ `{a: 1}` ma typ `{a: number}`, a nie `{a: string}`. Komunikat błędu w tym przypadku będzie wyglądał mniej więcej tak:

```
test/test.ts:999:999 - error TS2344: Type '{ a: string; }' does not satisfy the constraint '{ a: \\"Expected: string, Actual: number\\"; }'.
  Types of property 'a' are incompatible.
    Type 'string' is not assignable to type '\\"Expected: string, Actual: number\\"'.

999 expectTypeOf({a: 1}).toEqualTypeOf<{a: string}>()
```

Zauważ, że zgłoszone ograniczenie typu to czytelna dla człowieka wiadomość określająca zarówno typy "oczekiwany" jak i "faktyczny". Zamiast brać dosłownie zdanie `Types of property 'a' are incompatible // Type 'string' is not assignable to type "Expected: string, Actual: number"` - po prostu spójrz na nazwę właściwości (`'a'`) i wiadomość: `Expected: string, Actual: number`. To powie ci, co jest nie tak, w większości przypadków. Ekstremalnie złożone typy będą oczywiście wymagały więcej wysiłku do debugowania i mogą wymagać eksperymentowania. Proszę [zgłoś issue](https://github.com/mmkal/expect-type), jeśli komunikaty błędów są faktycznie mylące.

Metody `toBe...` (takie jak `toBeString`, `toBeNumber`, `toBeVoid` itp.) kończą się niepowodzeniem przez rozwiązanie do typu, który nie jest wywoływalny, gdy typ `Actual` pod testem nie pasuje. Na przykład niepowodzenie dla asercji takiej jak `expectTypeOf(1).toBeString()` będzie wyglądać mniej więcej tak:

```
test/test.ts:999:999 - error TS2349: This expression is not callable.
  Type 'ExpectString<number>' has no call signatures.

999 expectTypeOf(1).toBeString()
                    ~~~~~~~~~~
```

Część `This expression is not callable` nie jest zbyt pomocna - znaczący błąd jest w następnej linii, `Type 'ExpectString<number> has no call signatures`. To w zasadzie oznacza, że przekazałeś number, ale stwierdziłeś, że powinien być stringiem.

Jeśli TypeScript dodałby wsparcie dla [typów "throw"](https://github.com/microsoft/TypeScript/pull/40468), te komunikaty błędów mogłyby być znacząco ulepszone. Do tego czasu będą wymagały pewnego zmrużenia oczu.

#### Konkretne obiekty "oczekiwane" vs typeargs

Komunikaty błędów dla asercji takiej jak ta:

```ts
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: '' })
```

Będą mniej pomocne niż dla asercji takiej jak ta:

```ts
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: string }>()
```

Dzieje się tak, ponieważ kompilator TypeScript musi wywnioskować typearg dla stylu `.toEqualTypeOf({a: ''})`, a ta biblioteka może oznaczyć to jako niepowodzenie tylko przez porównanie z generycznym typem `Mismatch`. Więc, gdzie to możliwe, użyj typearg zamiast konkretnego typu dla `.toEqualTypeOf` i `.toExtend`. Jeśli jest znacznie wygodniej porównywać dwa konkretne typy, możesz użyć `typeof`:

```ts
const one = valueFromFunctionOne({ some: { complex: inputs } })
const two = valueFromFunctionTwo({ some: { other: inputs } })

expectTypeOf(one).toEqualTypeOf<typeof two>()
```

Jeśli masz trudności z pracą z API `expectTypeOf` i rozumieniem błędów, zawsze możesz użyć prostszego API `assertType`:

```ts
const answer = 42

assertType<number>(answer)
// @ts-expect-error answer nie jest stringiem
assertType<string>(answer)
```

::: tip
Podczas używania składni `@ts-expect-error` możesz chcieć upewnić się, że nie zrobiłeś literówki. Możesz to zrobić, włączając swoje pliki typów w opcji konfiguracji [`test.include`](/config/#include), więc Vitest faktycznie *uruchomi* te testy i zakończy się niepowodzeniem z `ReferenceError`.

To przejdzie, ponieważ oczekuje błędu, ale słowo "answer" ma literówkę, więc to fałszywy pozytywny błąd:

```ts
// @ts-expect-error answer nie jest stringiem
assertType<string>(answr)
```
:::

## Uruchamianie sprawdzania typów

Aby włączyć sprawdzanie typów, po prostu dodaj flagę [`--typecheck`](/config/#typecheck) do swojego polecenia Vitest w `package.json`:

```json [package.json]
{
  "scripts": {
    "test": "vitest --typecheck"
  }
}
```

Teraz możesz uruchomić sprawdzanie typów:

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
bun test
```
:::

Vitest używa `tsc --noEmit` lub `vue-tsc --noEmit`, w zależności od twojej konfiguracji, więc możesz usunąć te skrypty ze swojego pipeline.
