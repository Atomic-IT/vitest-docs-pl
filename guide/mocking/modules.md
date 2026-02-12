# Mockowanie modułów

## Definiowanie modułu

Przed mockowaniem "modułu" powinniśmy zdefiniować, czym on jest. W kontekście Vitest "moduł" to plik, który coś eksportuje. Używając [pluginów](https://vite.dev/guide/api-plugin.html), każdy plik może być przekształcony w moduł JavaScript. "Obiekt modułu" to obiekt przestrzeni nazw, który przechowuje dynamiczne referencje do eksportowanych identyfikatorów. Mówiąc prościej, to obiekt z eksportowanymi metodami i właściwościami. W tym przykładzie `example.js` to moduł, który eksportuje `answer` i `variable`:

```js [example.js]
export function answer() {
  // ...
  return 42
}

export const variable = 'example'
```

`exampleObject` tutaj to obiekt modułu:

```js [example.test.js]
import * as exampleObject from './example.js'
```

`exampleObject` będzie zawsze istnieć, nawet jeśli zaimportowałeś przykład używając nazwanych importów:

```js [example.test.js]
import { answer, variable } from './example.js'
```

Możesz odwołać się do `exampleObject` tylko poza samym modułem przykładowym. Na przykład w teście.

## Mockowanie modułu

Na potrzeby tego przewodnika wprowadźmy kilka definicji.

- **Mockowany moduł** to moduł, który został całkowicie zastąpiony innym.
- **Szpiegowany moduł** to mockowany moduł, ale jego eksportowane metody zachowują oryginalną implementację. Mogą być również śledzone.
- **Mockowany eksport** to eksport modułu, którego wywołania mogą być śledzone.
- **Szpiegowany eksport** to mockowany eksport.

Aby całkowicie mockować moduł, możesz użyć [API `vi.mock`](/api/vi#vi-mock). Możesz zdefiniować nowy moduł dynamicznie, dostarczając fabrykę, która zwraca nowy moduł jako drugi argument:

```ts
import { vi } from 'vitest'

// Moduł ./example.js zostanie zastąpiony
// wynikiem funkcji fabryki, a oryginalny
// moduł ./example.js nigdy nie zostanie wywołany
vi.mock(import('./example.js'), () => {
  return {
    answer() {
      // ...
      return 42
    },
    variable: 'mock',
  }
})
```

::: tip
Pamiętaj, że możesz wywołać `vi.mock` w [pliku setup](/config/setupfiles), aby automatycznie zastosować mock modułu w każdym pliku testowym.
:::

::: tip
Zwróć uwagę na użycie dynamicznego importu: `import('./example.ts')`. Vitest usunie go przed wykonaniem kodu, ale pozwala TypeScriptowi prawidłowo walidować string i typować metodę `importOriginal` w IDE lub CLI.
:::

Jeśli kod próbuje uzyskać dostęp do metody, która nie została zwrócona z tej fabryki, Vitest wyrzuci błąd z pomocnym komunikatem. Zauważ, że `answer` nie jest mockowany, tj. nie może być śledzony. Aby uczynić go śledzonym, użyj `vi.fn()`:

```ts
import { vi } from 'vitest'

vi.mock(import('./example.js'), () => {
  return {
    answer: vi.fn(),
    variable: 'mock',
  }
})
```

Metoda fabryki przyjmuje funkcję `importOriginal`, która wykona oryginalny moduł i zwróci jego obiekt modułu:

```ts
import { expect, vi } from 'vitest'
import { answer } from './example.js'

vi.mock(import('./example.js'), async (importOriginal) => {
  const originalModule = await importOriginal()
  return {
    answer: vi.fn(originalModule.answer),
    variable: 'mock',
  }
})

expect(answer()).toBe(42)

expect(answer).toHaveBeenCalled()
expect(answer).toHaveReturned(42)
```

::: warning
Zauważ, że `importOriginal` jest asynchroniczny i wymaga użycia `await`.
:::

W powyższym przykładzie dostarczyliśmy oryginalne `answer` do wywołania `vi.fn()`, więc może nadal je wywoływać, będąc jednocześnie śledzonym.

Jeśli wymagasz użycia `importOriginal`, rozważ szpiegowanie eksportu bezpośrednio przez inne API: `vi.spyOn`. Zamiast zastępować cały moduł, możesz szpiegować tylko pojedynczą eksportowaną metodę. Aby to zrobić, musisz zaimportować moduł jako obiekt przestrzeni nazw:

```ts
import { expect, vi } from 'vitest'
import * as exampleObject from './example.js'

const spy = vi.spyOn(exampleObject, 'answer').mockReturnValue(0)

expect(exampleObject.answer()).toBe(0)
expect(exampleObject.answer).toHaveBeenCalled()
```

::: danger Wsparcie trybu przeglądarki
To nie będzie działać w [trybie przeglądarki](/guide/browser/), ponieważ używa natywnego wsparcia ESM przeglądarki do serwowania modułów. Obiekt przestrzeni nazw modułu jest zapieczętowany i nie może być rekonfigurowany. Aby obejść to ograniczenie, Vitest wspiera opcję `{ spy: true }` w `vi.mock('./example.js')`. To automatycznie zaszpieguje każdy eksport w module bez zastępowania ich fałszywymi.

```ts
import { vi } from 'vitest'
import * as exampleObject from './example.js'

vi.mock('./example.js', { spy: true })

vi.mocked(exampleObject.answer).mockReturnValue(0)
```
:::

::: warning
Musisz zaimportować moduł jako obiekt przestrzeni nazw tylko w pliku, gdzie używasz narzędzia `vi.spyOn`. Jeśli `answer` jest wywoływane w innym pliku i jest tam importowane jako nazwany eksport, Vitest będzie w stanie prawidłowo je śledzić, o ile funkcja, która je wywołała, jest wywołana po `vi.spyOn`:

```ts [source.js]
import { answer } from './example.js'

export function question() {
  if (answer() === 42) {
    return 'Ostateczne pytanie o życie, wszechświat i wszystko'
  }

  return 'Nieznane pytanie'
}
```
:::

Zauważ, że `vi.spyOn` będzie szpiegować tylko wywołania, które zostały wykonane po tym, jak zaczął szpiegować metodę. Więc jeśli funkcja jest wykonywana na najwyższym poziomie podczas importu lub została wywołana przed szpiegowaniem, `vi.spyOn` nie będzie w stanie tego zgłosić.

Aby automatycznie mockować dowolny moduł przed jego zaimportowaniem, możesz wywołać `vi.mock` ze ścieżką:

```ts
import { vi } from 'vitest'

vi.mock(import('./example.js'))
```

Jeśli plik `./__mocks__/example.js` istnieje, Vitest załaduje go zamiast tego. W przeciwnym razie Vitest załaduje oryginalny moduł i zastąpi wszystko rekurencyjnie:

{#automocking-algorithm}

- Wszystkie tablice będą puste
- Wszystkie prymitywy pozostaną nietknięte
- Wszystkie gettery zwrócą `undefined`
- Wszystkie metody zwrócą `undefined`
- Wszystkie obiekty będą głęboko sklonowane
- Wszystkie instancje klas i ich prototypy będą sklonowane

Aby wyłączyć to zachowanie, możesz przekazać `spy: true` jako drugi argument:

```ts
import { vi } from 'vitest'

vi.mock(import('./example.js'), { spy: true })
```

Zamiast zwracać `undefined`, wszystkie metody wywołają oryginalną implementację, ale nadal możesz śledzić te wywołania:

```ts
import { expect, vi } from 'vitest'
import { answer } from './example.js'

vi.mock(import('./example.js'), { spy: true })

// wywołuje oryginalną implementację
expect(answer()).toBe(42)
// vitest nadal może śledzić wywołania
expect(answer).toHaveBeenCalled()
```

Jedną fajną rzeczą, którą wspierają mockowane moduły, jest współdzielenie stanu między instancją a jej prototypem. Rozważ ten moduł:

```ts [answer.js]
export class Answer {
  constructor(value) {
    this._value = value
  }

  value() {
    return this._value
  }
}
```

Mockując go, możemy śledzić każde wywołanie `.value()` nawet bez dostępu do samej instancji:

```ts [answer.test.js]
import { expect, test, vi } from 'vitest'
import { Answer } from './answer.js'

vi.mock(import('./answer.js'), { spy: true })

test('instancja dziedziczy stan', () => {
  // te wywołania mogą być prywatne wewnątrz innej funkcji,
  // do której nie masz dostępu w swoim teście
  const answer1 = new Answer(42)
  const answer2 = new Answer(0)

  expect(answer1.value()).toBe(42)
  expect(answer1.value).toHaveBeenCalled()
  // zauważ, że różne instancje mają swoje własne stany
  expect(answer2.value).not.toHaveBeenCalled()

  expect(answer2.value()).toBe(0)

  // ale stan prototypu akumuluje wszystkie wywołania
  expect(Answer.prototype.value).toHaveBeenCalledTimes(2)
  expect(Answer.prototype.value).toHaveReturned(42)
  expect(Answer.prototype.value).toHaveReturned(0)
})
```

To może być bardzo przydatne do śledzenia wywołań do instancji, które nigdy nie są eksponowane.

## Mockowanie nieistniejącego modułu

Vitest wspiera mockowanie modułów wirtualnych. Te moduły nie istnieją w systemie plików, ale kod je importuje. Na przykład, może się to zdarzyć, gdy środowisko deweloperskie różni się od produkcyjnego. Częstym przykładem jest mockowanie API `vscode` w testach jednostkowych.

Domyślnie Vitest nie uda się transformować plików, jeśli nie może znaleźć źródła importu. Aby to obejść, musisz określić to w swojej konfiguracji. Możesz albo zawsze przekierować import do pliku, albo po prostu zasygnalizować Vite, aby go zignorował i użył fabryki `vi.mock` do zdefiniowania jego eksportów.

Aby przekierować import, użyj opcji konfiguracji [`test.alias`](/config/#alias):

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { resolve } from 'node:path'

export default defineConfig({
  test: {
    alias: {
      vscode: resolve(import.meta.dirname, './mock/vscode.js'),
    },
  },
})
```

Aby oznaczyć moduł jako zawsze rozwiązany, zwróć ten sam string z hooka `resolveId` pluginu:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import { resolve } from 'node:path'

export default defineConfig({
  plugins: [
    {
      name: 'virtual-vscode',
      resolveId(id) {
        if (id === 'vscode') {
          return 'vscode'
        }
      }
    }
  ]
})
```

Teraz możesz używać `vi.mock` jak zwykle w swoich testach:

```ts
import { vi } from 'vitest'

vi.mock(import('vscode'), () => {
  return {
    window: {
      createOutputChannel: vi.fn(),
    }
  }
})
```

## Jak to działa

Vitest implementuje różne mechanizmy mockowania modułów w zależności od środowiska. Jedyną cechą, którą współdzielą, jest transformer pluginu. Gdy Vitest widzi, że plik ma wewnątrz `vi.mock`, transformuje każdy statyczny import w dynamiczny i przenosi wywołanie `vi.mock` na początek pliku. To pozwala Vitest zarejestrować mock przed importem bez łamania reguły ESM o wyniesionych importach.

::: code-group
```ts [example.js]
import { answer } from './answer.js'

vi.mock(import('./answer.js'))

console.log(answer)
```
```ts [example.transformed.js]
vi.mock('./answer.js')

const __vitest_module_0__ = await __handle_mock__(
  () => import('./answer.js')
)
// aby zachować żywe wiązanie, musimy uzyskać dostęp
// do eksportu na przestrzeni nazw modułu
console.log(__vitest_module_0__.answer())
```
:::

Wrapper `__handle_mock__` tylko upewnia się, że mock jest rozwiązany przed zainicjowaniem importu, nie modyfikuje modułu w żaden sposób.

Pluginy mockowania modułów są dostępne w [pakiecie `@vitest/mocker`](https://github.com/vitest-dev/vitest/tree/main/packages/mocker).

### JSDOM, happy-dom, Node

Gdy testy są uruchamiane w emulowanym środowisku, Vitest tworzy [runner modułu](https://vite.dev/guide/api-environment-runtimes.html#modulerunner), który może konsumować kod Vite. Runner modułu jest zaprojektowany w taki sposób, że Vitest może podłączyć się do ewaluacji modułu i zastąpić go mockiem, jeśli został zarejestrowany. To oznacza, że Vitest uruchamia kod w środowisku podobnym do ESM, ale nie używa bezpośrednio natywnego mechanizmu ESM. To pozwala runnerowi testów naginać zasady dotyczące niezmienności ES Modules, pozwalając użytkownikom wywoływać `vi.spyOn` na pozornie ES Module.

### Tryb przeglądarki

Vitest używa natywnego ESM w trybie przeglądarki. To oznacza, że nie możemy tak łatwo zastąpić modułu. Zamiast tego Vitest przechwytuje żądanie fetch (przez `page.route` Playwright lub API pluginu Vite, jeśli używasz `preview` lub `webdriverio`) i serwuje transformowany kod, jeśli moduł był mockowany.

Na przykład, jeśli moduł jest automockowany, Vitest może sparsować statyczne eksporty i utworzyć moduł zastępczy:

::: code-group
```ts [answer.js]
export function answer() {
  return 42
}
```
```ts [answer.transformed.js]
function answer() {
  return 42
}

const __private_module__ = {
  [Symbol.toStringTag]: 'Module',
  answer: vi.fn(answer),
}

export const answer = __private_module__.answer
```
:::

Przykład jest uproszczony dla zwięzłości, ale koncepcja pozostaje niezmieniona. Możemy wstrzyknąć zmienną `__private_module__` do modułu, aby przechowywać mockowane wartości. Jeśli użytkownik wywołał `vi.mock` z `spy: true`, przekazujemy oryginalną wartość; w przeciwnym razie tworzymy prosty mock `vi.fn()`.

Jeśli użytkownik zdefiniował niestandardową fabrykę, utrudnia to wstrzykiwanie kodu, ale nie jest to niemożliwe. Gdy mockowany plik jest serwowany, najpierw rozwiązujemy fabrykę w przeglądarce, następnie przekazujemy klucze z powrotem do serwera i używamy ich do utworzenia modułu zastępczego:

```ts
const resolvedFactoryKeys = await resolveBrowserFactory(url)
const mockedModule = `
const __private_module__ = getFactoryReturnValue(${url})
${resolvedFactoryKeys.map(key => `export const ${key} = __private_module__["${key}"]`).join('\n')}
`
```

Ten moduł może teraz być serwowany z powrotem do przeglądarki. Możesz sprawdzić kod w devtools, gdy uruchamiasz testy.

## Pułapki mockowania modułów

Uważaj, że nie jest możliwe mockowanie wywołań do metod, które są wywoływane wewnątrz innych metod tego samego pliku. Na przykład w tym kodzie:

```ts [foobar.js]
export function foo() {
  return 'foo'
}

export function foobar() {
  return `${foo()}bar`
}
```

Nie jest możliwe mockowanie metody `foo` z zewnątrz, ponieważ jest ona bezpośrednio referencjonowana. Więc ten kod nie będzie miał wpływu na wywołanie `foo` wewnątrz `foobar` (ale wpłynie na wywołanie `foo` w innych modułach):

```ts [foobar.test.ts]
import { vi } from 'vitest'
import * as mod from './foobar.js'

// to wpłynie tylko na "foo" poza oryginalnym modułem
vi.spyOn(mod, 'foo')
vi.mock(import('./foobar.js'), async (importOriginal) => {
  return {
    ...await importOriginal(),
    // to wpłynie tylko na "foo" poza oryginalnym modułem
    foo: () => 'mocked'
  }
})
```

Możesz potwierdzić to zachowanie, dostarczając implementację do metody `foobar` bezpośrednio:

```ts [foobar.test.js]
import * as mod from './foobar.js'

vi.spyOn(mod, 'foo')

// eksportowane foo referencjonuje mockowaną metodę
mod.foobar(mod.foo)
```

```ts [foobar.js]
export function foo() {
  return 'foo'
}

export function foobar(injectedFoo) {
  return injectedFoo === foo // false
}
```

To jest zamierzone zachowanie i nie planujemy implementować obejścia. Rozważ refaktoryzację swojego kodu na wiele plików lub użyj technik takich jak [wstrzykiwanie zależności](https://en.wikipedia.org/wiki/Dependency_injection). Uważamy, że uczynienie aplikacji testowalną nie jest odpowiedzialnością runnera testów, ale architektury aplikacji.
