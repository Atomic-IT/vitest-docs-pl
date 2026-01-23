---
title: Kontekst testu | Przewodnik
outline: deep
---

# Kontekst testu

Zainspirowany [Playwright Fixtures](https://playwright.dev/docs/test-fixtures), kontekst testu Vitest pozwala definiować narzędzia, stany i fixtures, które mogą być używane w Twoich testach.

## Użycie

Pierwszym argumentem dla każdego callbacku testu jest kontekst testu.

```ts
import { it } from 'vitest'

it('should work', ({ task }) => {
  // wyświetla nazwę testu
  console.log(task.name)
})
```

## Wbudowany kontekst testu

#### `task`

Obiekt tylko do odczytu zawierający metadane o teście.

#### `expect`

API `expect` powiązane z bieżącym testem:

```ts
import { it } from 'vitest'

it('math is easy', ({ expect }) => {
  expect(2 + 2).toBe(4)
})
```

To API jest przydatne do uruchamiania testów snapshot współbieżnie, ponieważ globalny expect nie może ich śledzić:

```ts
import { it } from 'vitest'

it.concurrent('math is easy', ({ expect }) => {
  expect(2 + 2).toMatchInlineSnapshot()
})

it.concurrent('math is hard', ({ expect }) => {
  expect(2 * 2).toMatchInlineSnapshot()
})
```

#### `skip`

```ts
function skip(note?: string): never
function skip(condition: boolean, note?: string): void
```

Pomija dalsze wykonywanie testu i oznacza test jako pominięty:

```ts
import { expect, it } from 'vitest'

it('math is hard', ({ skip }) => {
  skip()
  expect(2 + 2).toBe(5)
})
```

Od Vitest 3.1 akceptuje parametr logiczny do warunkowego pomijania testu:

```ts
it('math is hard', ({ skip, mind }) => {
  skip(mind === 'foggy')
  expect(2 + 2).toBe(5)
})
```

#### `annotate` <Version>3.2.0</Version> {#annotate}

```ts
function annotate(
  message: string,
  attachment?: TestAttachment,
): Promise<TestAnnotation>

function annotate(
  message: string,
  type?: string,
  attachment?: TestAttachment,
): Promise<TestAnnotation>
```

Dodaje [adnotację testu](/guide/test-annotations), która będzie wyświetlana przez Twój [reporter](/config/#reporters).

```ts
test('annotations API', async ({ annotate }) => {
  await annotate('https://github.com/vitest-dev/vitest/pull/7953', 'issues')
})
```

#### `signal` <Version>3.2.0</Version> {#signal}

[`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal), który może być przerwany przez Vitest. Sygnał jest przerywany w następujących sytuacjach:

- Test przekroczy limit czasu
- Użytkownik ręcznie anulował uruchomienie testu za pomocą Ctrl+C
- [`vitest.cancelCurrentRun`](/api/advanced/vitest#cancelcurrentrun) został wywołany programowo
- Inny test równoległy zakończył się niepowodzeniem i ustawiona jest flaga [`bail`](/config/#bail)

```ts
it('stop request when test times out', async ({ signal }) => {
  await fetch('/resource', { signal })
}, 2000)
```

#### `onTestFailed`

Hook [`onTestFailed`](/api/#ontestfailed) powiązany z bieżącym testem. To API jest przydatne, jeśli uruchamiasz testy współbieżnie i potrzebujesz specjalnej obsługi tylko dla tego konkretnego testu.

#### `onTestFinished`

Hook [`onTestFinished`](/api/#ontestfailed) powiązany z bieżącym testem. To API jest przydatne, jeśli uruchamiasz testy współbieżnie i potrzebujesz specjalnej obsługi tylko dla tego konkretnego testu.

## Rozszerzanie kontekstu testu

Vitest zapewnia dwa różne sposoby rozszerzania kontekstu testu.

### `test.extend`

Podobnie jak [Playwright](https://playwright.dev/docs/api/class-test#test-extend), możesz użyć tej metody do zdefiniowania własnego API `test` z niestandardowymi fixtures i używać go wszędzie.

Na przykład, najpierw tworzymy kolektor `test` z dwoma fixtures: `todos` i `archive`.

```ts [my-test.ts]
import { test as baseTest } from 'vitest'

const todos = []
const archive = []

export const test = baseTest.extend({
  todos: async ({}, use) => {
    // przygotuj fixture przed każdą funkcją testu
    todos.push(1, 2, 3)

    // użyj wartości fixture
    await use(todos)

    // wyczyść fixture po każdej funkcji testu
    todos.length = 0
  },
  archive
})
```

Następnie możemy go zaimportować i użyć.

```ts [my-test.test.ts]
import { expect } from 'vitest'
import { test } from './my-test.js'

test('add items to todos', ({ todos }) => {
  expect(todos.length).toBe(3)

  todos.push(4)
  expect(todos.length).toBe(4)
})

test('move items from todos to archive', ({ todos, archive }) => {
  expect(todos.length).toBe(3)
  expect(archive.length).toBe(0)

  archive.push(todos.pop())
  expect(todos.length).toBe(2)
  expect(archive.length).toBe(1)
})
```

Możemy również dodać więcej fixtures lub nadpisać istniejące fixtures, rozszerzając nasz `test`.

```ts
import { test as todosTest } from './my-test.js'

export const test = todosTest.extend({
  settings: {
    // ...
  }
})
```

#### Inicjalizacja fixture

Runner Vitest inteligentnie zainicjalizuje Twoje fixtures i wstrzyknie je do kontekstu testu na podstawie użycia.

```ts
import { test as baseTest } from 'vitest'

const test = baseTest.extend<{
  todos: number[]
  archive: number[]
}>({
  todos: async ({ task }, use) => {
    await use([1, 2, 3])
  },
  archive: []
})

// todos nie zostanie uruchomione
test('skip', () => {})
test('skip', ({ archive }) => {})

// todos zostanie uruchomione
test('run', ({ todos }) => {})
```

::: warning
Używając `test.extend()` z fixtures, zawsze powinieneś używać wzorca destrukturyzacji obiektów `{ todos }`, aby uzyskać dostęp do kontekstu zarówno w funkcji fixture, jak i w funkcji testu.

```ts
test('context must be destructured', (context) => { // [!code --]
  expect(context.todos.length).toBe(2)
})

test('context must be destructured', ({ todos }) => { // [!code ++]
  expect(todos.length).toBe(2)
})
```

:::

#### Automatyczne fixture

Vitest obsługuje również składnię krotki dla fixtures, pozwalając przekazać opcje dla każdego fixture. Na przykład możesz użyć tego do jawnej inicjalizacji fixture, nawet jeśli nie jest używane w testach.

```ts
import { test as base } from 'vitest'

const test = base.extend({
  fixture: [
    async ({}, use) => {
      // ta funkcja zostanie uruchomiona
      setup()
      await use()
      teardown()
    },
    { auto: true } // Oznacz jako automatyczne fixture
  ],
})

test('works correctly')
```

#### Domyślne fixture

Od Vitest 3 możesz dostarczyć różne wartości w różnych [projektach](/guide/projects). Aby włączyć tę funkcję, przekaż `{ injected: true }` do opcji. Jeśli klucz nie jest określony w [konfiguracji projektu](/config/#provide), zostanie użyta wartość domyślna.

:::code-group
```ts [fixtures.test.ts]
import { test as base } from 'vitest'

const test = base.extend({
  url: [
    // wartość domyślna, jeśli "url" nie jest zdefiniowany w konfiguracji
    '/default',
    // oznacz fixture jako "injected", aby umożliwić nadpisanie
    { injected: true },
  ],
})

test('works correctly', ({ url }) => {
  // url to "/default" w "project-new"
  // url to "/full" w "project-full"
  // url to "/empty" w "project-empty"
})
```
```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      {
        test: {
          name: 'project-new',
        },
      },
      {
        test: {
          name: 'project-full',
          provide: {
            url: '/full',
          },
        },
      },
      {
        test: {
          name: 'project-empty',
          provide: {
            url: '/empty',
          },
        },
      },
    ],
  },
})
```
:::

#### Zakres wartości dla zestawu <Version>3.1.0</Version> {#scoping-values-to-suite}

Od Vitest 3.1 możesz nadpisać wartości kontekstu dla zestawu i jego dzieci, używając API `test.scoped`:

```ts
import { test as baseTest, describe, expect } from 'vitest'

const test = baseTest.extend({
  dependency: 'default',
  dependant: ({ dependency }, use) => use({ dependency })
})

describe('use scoped values', () => {
  test.scoped({ dependency: 'new' })

  test('uses scoped value', ({ dependant }) => {
    // `dependant` używa nowej nadpisanej wartości, która jest zakresowana
    // do wszystkich testów w tym zestawie
    expect(dependant).toEqual({ dependency: 'new' })
  })

  describe('keeps using scoped value', () => {
    test('uses scoped value', ({ dependant }) => {
      // zagnieżdżony zestaw odziedziczył wartość
      expect(dependant).toEqual({ dependency: 'new' })
    })
  })
})

test('keep using the default values', ({ dependant }) => {
  // `dependency` używa wartości domyślnej
  // poza zestawem z .scoped
  expect(dependant).toEqual({ dependency: 'default' })
})
```

To API jest szczególnie przydatne, jeśli masz wartość kontekstu, która zależy od dynamicznej zmiennej, takiej jak połączenie z bazą danych:

```ts
const test = baseTest.extend<{
  db: Database
  schema: string
}>({
  db: async ({ schema }, use) => {
    const db = await createDb({ schema })
    await use(db)
    await cleanup(db)
  },
  schema: '',
})

describe('one type of schema', () => {
  test.scoped({ schema: 'schema-1' })

  // ... testy
})

describe('another type of schema', () => {
  test.scoped({ schema: 'schema-2' })

  // ... testy
})
```

#### Kontekst na zakres <Version>3.2.0</Version>

Możesz zdefiniować kontekst, który będzie inicjowany raz na plik lub worker. Jest inicjowany w ten sam sposób jak zwykłe fixture z parametrem obiektów:

```ts
import { test as baseTest } from 'vitest'

export const test = baseTest.extend({
  perFile: [
    ({}, use) => use([]),
    { scope: 'file' },
  ],
  perWorker: [
    ({}, use) => use([]),
    { scope: 'worker' },
  ],
})
```

Wartość jest inicjowana przy pierwszym dostępie dowolnego testu, chyba że opcje fixture mają `auto: true` - w takim przypadku wartość jest inicjowana przed uruchomieniem jakiegokolwiek testu.

```ts
const test = baseTest.extend({
  perFile: [
    ({}, use) => use([]),
    {
      scope: 'file',
      // zawsze uruchom ten hook przed jakimkolwiek testem
      auto: true
    },
  ],
})
```

::: warning
Wbudowany kontekst testu [`task`](#task) **nie jest dostępny** w fixtures zakresowanych na plik lub worker. Te fixtures otrzymują inny obiekt kontekstu (kontekst pliku lub workera), który nie zawiera właściwości specyficznych dla testu, takich jak `task`.

Jeśli potrzebujesz dostępu do metadanych na poziomie pliku, takich jak ścieżka pliku, użyj zamiast tego `expect.getState().testPath`.
:::

Zakres `worker` uruchomi fixture raz na worker. Liczba uruchomionych workerów zależy od różnych czynników. Domyślnie każdy plik działa w oddzielnym workerze, więc zakresy `file` i `worker` działają tak samo.

Jednak jeśli wyłączysz [izolację](/config/#isolate), liczba workerów jest ograniczona przez konfigurację [`maxWorkers`](/config/#maxworkers).

Zauważ, że określenie `scope: 'worker'` podczas uruchamiania testów w `vmThreads` lub `vmForks` będzie działać tak samo jak `scope: 'file'`. To ograniczenie istnieje, ponieważ każdy plik testowy ma własny kontekst VM, więc gdyby Vitest inicjował go raz, jeden kontekst mógłby przeciekać do drugiego i tworzyć wiele niespójności referencji (instancje tej samej klasy odwoływałyby się do różnych konstruktorów, na przykład).

#### TypeScript

Aby dostarczyć typy fixture dla wszystkich niestandardowych kontekstów, możesz przekazać typ fixtures jako generic.

```ts
interface MyFixtures {
  todos: number[]
  archive: number[]
}

const test = baseTest.extend<MyFixtures>({
  todos: [],
  archive: []
})

test('types are defined correctly', ({ todos, archive }) => {
  expectTypeOf(todos).toEqualTypeOf<number[]>()
  expectTypeOf(archive).toEqualTypeOf<number[]>()
})
```

::: info Wnioskowanie typów
Zauważ, że Vitest nie obsługuje wnioskowania typów, gdy wywoływana jest funkcja `use`. Zawsze preferowane jest przekazanie całego typu kontekstu jako typu generycznego podczas wywoływania `test.extend`:

```ts
import { test as baseTest } from 'vitest'

const test = baseTest.extend<{
  todos: number[]
  schema: string
}>({
  todos: ({ schema }, use) => use([]),
  schema: 'test'
})

test('types are correct', ({
  todos, // number[]
  schema, // string
}) => {
  // ...
})
```

:::

Używając `test.extend`, rozszerzony obiekt `test` zapewnia bezpieczne typowo hooki `beforeEach` i `afterEach`, które są świadome nowego kontekstu:

```ts
const test = baseTest.extend<{
  todos: number[]
}>({
  todos: async ({}, use) => {
    await use([])
  },
})

// W przeciwieństwie do globalnych hooków, te hooki są świadome rozszerzonego kontekstu
test.beforeEach(({ todos }) => {
  todos.push(1)
})

test.afterEach(({ todos }) => {
  console.log(todos)
})
```
