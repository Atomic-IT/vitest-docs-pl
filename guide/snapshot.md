---
title: Snapshot | Przewodnik
---

# Snapshot

<CourseLink href="https://vueschool.io/lessons/snapshots-in-vitest?friend=vueuse">Naucz się Snapshot przez wideo od Vue School</CourseLink>

Testy snapshotowe są bardzo przydatnym narzędziem, gdy trzeba upewnić się, że wyjście funkcji nie zmienia się niespodziewanie.

Podczas używania snapshotów, Vitest wykonuje snapshot podanej wartości, a następnie porównuje go z referencyjnym plikiem snapshot przechowywanym obok testu. Test nie powiedzie się, jeśli dwa snapshoty się nie zgadzają: albo zmiana jest niespodziewana, albo referencyjny snapshot musi zostać zaktualizowany do nowej wersji wyniku.

## Używanie snapshotów

Aby wykonać snapshot wartości, możesz użyć [`toMatchSnapshot()`](/api/expect#tomatchsnapshot) z API `expect()`:

```ts
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchSnapshot()
})
```

Przy pierwszym uruchomieniu tego testu, Vitest tworzy plik snapshot, który wygląda tak:

```js
// Vitest Snapshot v1, https://vitest.dev/guide/snapshot.html

exports['toUpperCase 1'] = '"FOOBAR"'
```

Artefakt snapshot powinien być commitowany wraz ze zmianami w kodzie i przeglądany jako część procesu code review. Przy kolejnych uruchomieniach testów Vitest porówna wyrenderowane wyjście z poprzednim snapshotem. Jeśli się zgadzają, test przejdzie. Jeśli się nie zgadzają, albo runner testów znalazł błąd w kodzie, który powinien zostać naprawiony, albo implementacja się zmieniła i snapshot musi zostać zaktualizowany.

::: warning
Podczas używania snapshotów z asynchronicznymi testami współbieżnymi, `expect` z lokalnego [kontekstu testu](/guide/test-context) musi być użyty, aby zapewnić wykrycie właściwego testu.
:::

## Snapshoty inline

Podobnie możesz użyć [`toMatchInlineSnapshot()`](/api/expect#tomatchinlinesnapshot), aby przechowywać snapshot inline w pliku testowym.

```ts
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchInlineSnapshot()
})
```

Zamiast tworzyć plik snapshot, Vitest zmodyfikuje plik testowy bezpośrednio, aby zaktualizować snapshot jako string:

```ts
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchInlineSnapshot('"FOOBAR"')
})
```

To pozwala zobaczyć oczekiwane wyjście bezpośrednio bez przeskakiwania między różnymi plikami.

::: warning
Podczas używania snapshotów z asynchronicznymi testami współbieżnymi, `expect` z lokalnego [kontekstu testu](/guide/test-context) musi być użyty, aby zapewnić wykrycie właściwego testu.
:::

## Aktualizowanie snapshotów

Gdy otrzymana wartość nie zgadza się ze snapshotem, test nie przechodzi i pokazuje różnicę między nimi. Gdy zmiana snapshotu jest oczekiwana, możesz chcieć zaktualizować snapshot z bieżącego stanu.

W trybie watch możesz nacisnąć klawisz `u` w terminalu, aby bezpośrednio zaktualizować nieudany snapshot.

Lub możesz użyć flagi `--update` lub `-u` w CLI, aby Vitest zaktualizował snapshoty.

```bash
vitest -u
```

## Snapshoty plikowe

Podczas wywoływania `toMatchSnapshot()`, przechowujemy wszystkie snapshoty w sformatowanym pliku snap. To oznacza, że musimy escapować niektóre znaki (mianowicie podwójny cudzysłów `"` i backtick `` ` ``) w stringu snapshotu. Jednocześnie możesz stracić podświetlanie składni dla zawartości snapshotu (jeśli są w jakimś języku).

W świetle tego wprowadziliśmy [`toMatchFileSnapshot()`](/api/expect#tomatchfilesnapshot), aby jawnie dopasowywać do pliku. To pozwala przypisać dowolne rozszerzenie pliku do pliku snapshot i sprawia, że są bardziej czytelne.

```ts
import { expect, it } from 'vitest'

it('render basic', async () => {
  const result = renderHTML(h('div', { class: 'foo' }))
  await expect(result).toMatchFileSnapshot('./test/basic.output.html')
})
```

Porówna to z zawartością `./test/basic.output.html`. I może być zapisane z powrotem z flagą `--update`.

## Snapshoty wizualne

Do testów regresji wizualnej komponentów UI i stron, Vitest zapewnia wbudowane wsparcie przez [tryb przeglądarki](/guide/browser/) z asercją [`toMatchScreenshot()`](/api/browser/assertions#tomatchscreenshot-experimental):

```ts
import { expect, test } from 'vitest'
import { page } from 'vitest/browser'

test('przycisk wygląda poprawnie', async () => {
  const button = page.getByRole('button')
  await expect(button).toMatchScreenshot('primary-button')
})
```

To przechwytuje zrzuty ekranu i porównuje je z referencyjnymi obrazami, aby wykryć niezamierzone zmiany wizualne. Dowiedz się więcej w [przewodniku Testowanie regresji wizualnej](/guide/browser/visual-regression-testing).

## Niestandardowy serializer

Możesz dodać własną logikę do zmiany sposobu serializacji snapshotów. Jak Jest, Vitest ma domyślne serializery dla wbudowanych typów JavaScript, elementów HTML, ImmutableJS i dla elementów React.

Możesz jawnie dodać niestandardowy serializer używając API [`expect.addSnapshotSerializer`](/api/expect#expect-addsnapshotserializer).

```ts
expect.addSnapshotSerializer({
  serialize(val, config, indentation, depth, refs, printer) {
    // `printer` to funkcja, która serializuje wartość używając istniejących pluginów.
    return `Pretty foo: ${printer(
      val.foo,
      config,
      indentation,
      depth,
      refs,
    )}`
  },
  test(val) {
    return val && Object.prototype.hasOwnProperty.call(val, 'foo')
  },
})
```

Wspieramy również opcję [snapshotSerializers](/config/#snapshotserializers), aby niejawnie dodawać niestandardowe serializery.

```ts [path/to/custom-serializer.ts]
import { SnapshotSerializer } from 'vitest'

export default {
  serialize(val, config, indentation, depth, refs, printer) {
    // `printer` to funkcja, która serializuje wartość używając istniejących pluginów.
    return `Pretty foo: ${printer(
      val.foo,
      config,
      indentation,
      depth,
      refs,
    )}`
  },
  test(val) {
    return val && Object.prototype.hasOwnProperty.call(val, 'foo')
  },
} satisfies SnapshotSerializer
```

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    snapshotSerializers: ['path/to/custom-serializer.ts'],
  },
})
```

Po dodaniu testu takiego jak ten:

```ts
test('foo snapshot test', () => {
  const bar = {
    foo: {
      x: 1,
      y: 2,
    },
  }

  expect(bar).toMatchSnapshot()
})
```

Otrzymasz następujący snapshot:

```
Pretty foo: Object {
  "x": 1,
  "y": 2,
}
```

Używamy `pretty-format` od Jest do serializacji snapshotów. Możesz przeczytać więcej tutaj: [pretty-format](https://github.com/facebook/jest/blob/main/packages/pretty-format/README.md#serialize).

## Różnice od Jest

Vitest zapewnia niemal kompatybilną funkcjonalność snapshotów z [Jest](https://jestjs.io/docs/snapshot-testing) z kilkoma wyjątkami:

#### 1. Nagłówek komentarza w pliku snapshot jest inny

```diff
- // Jest Snapshot v1, https://goo.gl/fbAQLP
+ // Vitest Snapshot v1, https://vitest.dev/guide/snapshot.html
```

To tak naprawdę nie wpływa na funkcjonalność, ale może wpłynąć na diff przy commitach podczas migracji z Jest.

#### 2. `printBasicPrototype` jest domyślnie `false`

Zarówno snapshoty Jest jak i Vitest są napędzane przez [`pretty-format`](https://github.com/facebook/jest/blob/main/packages/pretty-format). W Vitest ustawiamy `printBasicPrototype` domyślnie na `false`, aby zapewnić czystsze wyjście snapshotu, podczas gdy w Jest <29.0.0 jest to domyślnie `true`.

```ts
import { expect, test } from 'vitest'

test('snapshot', () => {
  const bar = [
    {
      foo: 'bar',
    },
  ]

  // w Jest
  expect(bar).toMatchInlineSnapshot(`
    Array [
      Object {
        "foo": "bar",
      },
    ]
  `)

  // w Vitest
  expect(bar).toMatchInlineSnapshot(`
    [
      {
        "foo": "bar",
      },
    ]
  `)
})
```

Uważamy, że to bardziej rozsądne domyślne ustawienie dla czytelności i ogólnego DX. Jeśli nadal wolisz zachowanie Jest, możesz zmienić swoją konfigurację:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    snapshotFormat: {
      printBasicPrototype: true,
    },
  },
})
```

#### 3. Znak `>` jest używany jako separator zamiast dwukropka `:` dla niestandardowych wiadomości

Vitest używa znaku `>` jako separatora zamiast dwukropka `:` dla czytelności, gdy niestandardowa wiadomość jest przekazywana podczas tworzenia pliku snapshot.

Dla następującego przykładowego kodu testowego:
```js
test('toThrowErrorMatchingSnapshot', () => {
  expect(() => {
    throw new Error('error')
  }).toThrowErrorMatchingSnapshot('hint')
})
```

W Jest snapshot będzie:
```console
exports[`toThrowErrorMatchingSnapshot: hint 1`] = `"error"`;
```

W Vitest równoważny snapshot będzie:
```console
exports[`toThrowErrorMatchingSnapshot > hint 1`] = `[Error: error]`;
```

#### 4. Domyślny snapshot `Error` jest inny dla `toThrowErrorMatchingSnapshot` i `toThrowErrorMatchingInlineSnapshot`

```js
import { expect, test } from 'vitest'

test('snapshot', () => {
  // w Jest i Vitest
  expect(new Error('error')).toMatchInlineSnapshot(`[Error: error]`)

  // Jest robi snapshot `Error.message` dla instancji `Error`
  // Vitest wypisuje tę samą wartość co toMatchInlineSnapshot
  expect(() => {
    throw new Error('error')
  }).toThrowErrorMatchingInlineSnapshot(`"error"`) // [!code --]
  }).toThrowErrorMatchingInlineSnapshot(`[Error: error]`) // [!code ++]
})
```
