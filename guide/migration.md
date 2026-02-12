---
title: Przewodnik migracji | Przewodnik
outline: deep
---

# Przewodnik migracji

## Migracja do Vitest 4.0 {#vitest-4}

### Główne zmiany w pokryciu kodu V8

Dostawca pokrycia kodu V8 w Vitest używa teraz dokładniejszej logiki remapowania wyników pokrycia.
Oczekuje się, że użytkownicy zobaczą zmiany w swoich raportach pokrycia podczas aktualizacji z Vitest v3.

W przeszłości Vitest używał [`v8-to-istanbul`](https://github.com/istanbuljs/v8-to-istanbul) do remapowania wyników pokrycia V8 na pliki źródłowe.
Ta metoda nie była zbyt dokładna i dostarczała wiele fałszywych pozytywów w raportach pokrycia.
Opracowaliśmy teraz nowy pakiet, który wykorzystuje analizę opartą na AST dla pokrycia V8.
To pozwala raportom V8 być tak dokładnymi jak raporty `@vitest/coverage-istanbul`.

- Podpowiedzi ignorowania pokrycia zostały zaktualizowane. Zobacz [Pokrycie | Ignorowanie kodu](/guide/coverage.html#ignoring-code).
- `coverage.ignoreEmptyLines` zostało usunięte. Linie bez kodu runtime nie są już uwzględniane w raportach.
- `coverage.experimentalAstAwareRemapping` zostało usunięte. Ta opcja jest teraz domyślnie włączona i jest jedyną wspieraną metodą remapowania.
- `coverage.ignoreClassMethods` jest teraz wspierane również przez dostawcę V8.

### Usunięte opcje `coverage.all` i `coverage.extensions`

W poprzednich wersjach Vitest domyślnie uwzględniał wszystkie niepokryte pliki w raporcie pokrycia.
Było to spowodowane domyślną wartością `coverage.all` ustawioną na `true` i `coverage.include` ustawioną na `**`.
Te domyślne wartości zostały wybrane z dobrego powodu - narzędzia testowe nie mogą zgadnąć, gdzie użytkownicy przechowują swoje pliki źródłowe.

To skutkowało przetwarzaniem przez dostawców pokrycia Vitest nieoczekiwanych plików, takich jak zminifikowany Javascript, prowadząc do wolnego/zablokowanego generowania raportów pokrycia.
W Vitest v4 całkowicie usunęliśmy `coverage.all` i <ins>**domyślnie uwzględniamy tylko pokryte pliki w raporcie**</ins>.

Podczas aktualizacji do v4 zaleca się zdefiniowanie `coverage.include` w konfiguracji, a następnie stosowanie prostych wzorców `coverage.exclude` w razie potrzeby.

```ts [vitest.config.ts]
export default defineConfig({
  test: {
    coverage: {
      // Uwzględnij pokryte i niepokryte pliki pasujące do tego wzorca:
      include: ['packages/**/src/**.{js,jsx,ts,tsx}'], // [!code ++]

      // Wykluczenie jest stosowane dla plików pasujących do powyższego wzorca include
      // Nie trzeba definiować plików *.config.ts na poziomie root ani node_modules, ponieważ nie dodaliśmy ich w include
      exclude: ['**/some-pattern/**'], // [!code ++]

      // Te opcje są teraz usunięte
      all: true, // [!code --]
      extensions: ['js', 'ts'], // [!code --]
    }
  }
})
```

Jeśli `coverage.include` nie jest zdefiniowane, raport pokrycia będzie zawierał tylko pliki załadowane podczas uruchomienia testu:
```ts [vitest.config.ts]
export default defineConfig({
  test: {
    coverage: {
      // Include nie ustawione, uwzględnij tylko pliki załadowane podczas uruchomienia testu
      include: undefined, // [!code ++]

      // Załadowane pliki pasujące do tego wzorca będą wykluczone:
      exclude: ['**/some-pattern/**'], // [!code ++]
    }
  }
})
```

Zobacz również nowe przewodniki:
- [Włączanie i wykluczanie plików z raportu pokrycia](/guide/coverage.html#including-and-excluding-files-from-coverage-report) po przykłady
- [Profilowanie wydajności testów | Pokrycie kodu](/guide/profiling-test-performance.html#code-coverage) po wskazówki dotyczące debugowania generowania pokrycia

### Uproszczone `exclude`

Domyślnie Vitest teraz wyklucza testy tylko z folderów `node_modules` i `.git`. Oznacza to, że Vitest nie wyklucza już:

- folderów `dist` i `cypress`
- folderów `.idea`, `.cache`, `.output`, `.temp`
- plików konfiguracyjnych jak `rollup.config.js`, `prettier.config.js`, `ava.config.js` itd.

Aby ograniczyć katalog z plikami testowymi, użyj opcji [`test.dir`](/config/dir), ponieważ jest bardziej wydajna niż wykluczanie plików:

```ts
import { configDefaults, defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    dir: './frontend/tests', // [!code ++]
  },
})
```

Aby przywrócić poprzednie zachowanie, określ ręcznie stare `excludes`:

```ts
import { configDefaults, defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    exclude: [
      ...configDefaults.exclude,
      '**/dist/**', // [!code ++]
      '**/cypress/**', // [!code ++]
      '**/.{idea,git,cache,output,temp}/**', // [!code ++]
      '**/{karma,rollup,webpack,vite,vitest,jest,ava,babel,nyc,cypress,tsup,build,eslint,prettier}.config.*' // [!code ++]
    ],
  },
})
```

### `spyOn` i `fn` wspierają konstruktory

Wcześniej, jeśli próbowałeś szpiegować konstruktor za pomocą `vi.spyOn`, otrzymywałeś błąd jak `Constructor <name> requires 'new'`. Od Vitest 4 wszystkie mocki wywoływane ze słowem kluczowym `new` konstruują instancję zamiast wywoływać `mock.apply`. Oznacza to, że implementacja mocka musi używać albo słowa kluczowego `function`, albo `class` w tych przypadkach:

```ts {12-14,16-20}
const cart = {
  Apples: class Apples {
    getApples() {
      return 42
    }
  }
}

const Spy = vi.spyOn(cart, 'Apples')
  .mockImplementation(() => ({ getApples: () => 0 })) // [!code --]
  // ze słowem kluczowym function
  .mockImplementation(function () {
    this.getApples = () => 0
  })
  // z niestandardową klasą
  .mockImplementation(class MockApples {
    getApples() {
      return 0
    }
  })

const mock = new Spy()
```

Zauważ, że teraz jeśli podasz funkcję strzałkową, otrzymasz [błąd `<anonymous> is not a constructor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Not_a_constructor), gdy mock zostanie wywołany.

### Zmiany w mockowaniu

Wraz z nowymi funkcjami jak wsparcie konstruktorów, Vitest 4 tworzy mocki inaczej, aby rozwiązać kilka problemów z mockowaniem modułów, które otrzymaliśmy przez lata. Ta wersja próbuje uczynić szpiegów modułów mniej mylącymi, szczególnie podczas pracy z klasami.

- `vi.fn().getMockName()` teraz domyślnie zwraca `vi.fn()` zamiast `spy`. Może to wpłynąć na snapshoty z mockami - nazwa zostanie zmieniona z `[MockFunction spy]` na `[MockFunction]`. Szpiegi utworzone za pomocą `vi.spyOn` będą domyślnie nadal używać oryginalnej nazwy dla lepszego doświadczenia debugowania
- `vi.restoreAllMocks` nie resetuje już stanu szpiegów i przywraca tylko szpiegów utworzonych ręcznie za pomocą `vi.spyOn`, automocki nie są już dotknięte przez tę funkcję (dotyczy to również opcji konfiguracji [`restoreMocks`](/config/#restoremocks)). Zauważ, że `.mockRestore` nadal będzie resetować implementację mocka i czyścić stan
- Wywołanie `vi.spyOn` na mocku teraz zwraca ten sam mock
- `mock.settledResults` są teraz wypełniane natychmiast przy wywołaniu funkcji z wynikiem `'incomplete'`. Gdy promise jest zakończony, typ jest zmieniany zgodnie z wynikiem.
- Automockowane metody instancji są teraz prawidłowo izolowane, ale współdzielą stan z prototypem. Nadpisanie implementacji prototypu zawsze wpłynie na metody instancji, chyba że metody mają własną niestandardową implementację mocka. Wywołanie `.mockReset` na mocku również nie łamie już tego dziedziczenia.
```ts
import { AutoMockedClass } from './example.js'
const instance1 = new AutoMockedClass()
const instance2 = new AutoMockedClass()

instance1.method.mockReturnValue(42)

expect(instance1.method()).toBe(42)
expect(instance2.method()).toBe(undefined)

expect(AutoMockedClass.prototype.method).toHaveBeenCalledTimes(2)

instance1.method.mockReset()
AutoMockedClass.prototype.method.mockReturnValue(100)

expect(instance1.method()).toBe(100)
expect(instance2.method()).toBe(100)

expect(AutoMockedClass.prototype.method).toHaveBeenCalledTimes(4)
```
- Automockowane metody nie mogą być już przywracane, nawet ręcznym `.mockRestore`. Automockowane moduły z `spy: true` będą nadal działać jak wcześniej
- Automockowane gettery nie wywołują już oryginalnego gettera. Domyślnie automockowane gettery teraz zwracają `undefined`. Możesz nadal używać `vi.spyOn(object, name, 'get')`, aby szpiegować getter i zmienić jego implementację
- Mock `vi.fn(implementation).mockReset()` teraz prawidłowo zwraca implementację mocka w `.getMockImplementation()`
- `vi.fn().mock.invocationCallOrder` teraz zaczyna od `1`, jak w Jest, zamiast od `0`

### Tryb standalone z filtrem nazwy pliku

Aby poprawić doświadczenie użytkownika, Vitest teraz zacznie uruchamiać dopasowane pliki, gdy [`--standalone`](/guide/cli#standalone) jest używane z filtrem nazwy pliku.

```sh
# W Vitest v3 i wcześniejszych to polecenie ignorowałoby filtr nazwy pliku "math.test.ts".
# W Vitest v4 math.test.ts uruchomi się automatycznie.
$ vitest --standalone math.test.ts
```

To pozwala użytkownikom tworzyć wielokrotnego użytku skrypty `package.json` dla trybu standalone.

::: code-group
```json [package.json]
{
  "scripts": {
    "test:dev": "vitest --standalone"
  }
}
```
```bash [CLI]
# Uruchom Vitest w trybie standalone, bez uruchamiania żadnych plików na starcie
$ pnpm run test:dev

# Uruchom math.test.ts natychmiast
$ pnpm run test:dev math.test.ts
```
:::

### Zastąpienie `vite-node` przez [Module Runner](https://vite.dev/guide/api-environment-runtimes.html#modulerunner)

Module Runner jest następcą `vite-node` zaimplementowanym bezpośrednio w Vite. Vitest teraz używa go bezpośrednio zamiast mieć wrapper wokół handlera Vite SSR. Oznacza to, że pewne funkcje nie są już dostępne:

- Zmienna środowiskowa `VITE_NODE_DEPS_MODULE_DIRECTORIES` została zastąpiona przez `VITEST_MODULE_DIRECTORIES`
- Vitest nie wstrzykuje już `__vitest_executor` do każdego [runnera testów](/api/advanced/runner). Zamiast tego wstrzykuje `moduleRunner`, który jest instancją [`ModuleRunner`](https://vite.dev/guide/api-environment-runtimes.html#modulerunner)
- Punkt wejścia `vitest/execute` został usunięty. Zawsze był przeznaczony do użytku wewnętrznego
- [Niestandardowe środowiska](/guide/environment) nie muszą już dostarczać właściwości `transformMode`. Zamiast tego dostarcz `viteEnvironment`. Jeśli nie jest dostarczone, Vitest użyje nazwy środowiska do transformacji plików na serwerze (zobacz [`server.environments`](https://vite.dev/guide/api-environment-instances.html))
- `vite-node` nie jest już zależnością Vitest
- `deps.optimizer.web` zostało przemianowane na [`deps.optimizer.client`](/config/#deps-optimizer-client). Możesz również używać dowolnych niestandardowych nazw do stosowania konfiguracji optymalizatora podczas używania innych środowisk serwera

Vite ma swój własny mechanizm eksternalizacji, ale zdecydowaliśmy się nadal używać starego, aby zmniejszyć ilość przełomowych zmian. Możesz nadal używać [`server.deps`](/config/#server-deps) do inline'owania lub eksternalizacji pakietów.

Ta aktualizacja nie powinna być zauważalna, chyba że polegasz na zaawansowanych funkcjach wymienionych powyżej.

### `workspace` zostało zastąpione przez `projects`

Opcja konfiguracji `workspace` została przemianowana na [`projects`](/guide/projects) w Vitest 3.2. Są funkcjonalnie takie same, z wyjątkiem tego, że nie możesz określić innego pliku jako źródła swojego workspace (wcześniej mogłeś określić plik, który eksportowałby tablicę projektów). Migracja do `projects` jest łatwa, po prostu przenieś kod z `vitest.workspace.js` do `vitest.config.ts`:

::: code-group
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    workspace: './vitest.workspace.js', // [!code --]
    projects: [ // [!code ++]
      './packages/*', // [!code ++]
      { // [!code ++]
        test: { // [!code ++]
          name: 'unit', // [!code ++]
        }, // [!code ++]
      }, // [!code ++]
    ] // [!code ++]
  }
})
```
```ts [vitest.workspace.js]
import { defineWorkspace } from 'vitest/config' // [!code --]

export default defineWorkspace([ // [!code --]
  './packages/*', // [!code --]
  { // [!code --]
    test: { // [!code --]
      name: 'unit', // [!code --]
    }, // [!code --]
  } // [!code --]
]) // [!code --]
```
:::

### Przebudowa dostawcy przeglądarki

W Vitest 4.0 dostawca przeglądarki teraz przyjmuje obiekt zamiast stringa (`'playwright'`, `'webdriverio'`). `preview` nie jest już domyślny. To ułatwia pracę z niestandardowymi opcjami i nie wymaga już dodawania komentarzy `/// <reference`.

```ts
import { playwright } from '@vitest/browser-playwright' // [!code ++]

export default defineConfig({
  test: {
    browser: {
      provider: 'playwright', // [!code --]
      provider: playwright({ // [!code ++]
        launchOptions: { // [!code ++]
          slowMo: 100, // [!code ++]
        }, // [!code ++]
      }), // [!code ++]
      instances: [
        {
          browser: 'chromium',
          launch: { // [!code --]
            slowMo: 100, // [!code --]
          }, // [!code --]
        },
      ],
    },
  },
})
```

Nazewnictwo właściwości w fabryce `playwright` jest teraz również zgodne z [dokumentacją Playwright](https://playwright.dev/docs/api/class-testoptions#test-options-launch-options), co ułatwia znajdowanie.

Z tą zmianą pakiet `@vitest/browser` nie jest już potrzebny i możesz go usunąć ze swoich zależności. Aby wspierać import kontekstu, powinieneś zaktualizować `@vitest/browser/context` na `vitest/browser`:

```ts
import { page } from '@vitest/browser/context' // [!code --]
import { page } from 'vitest/browser' // [!code ++]

test('przykład', async () => {
  await page.getByRole('button').click()
})
```

Moduły są identyczne, więc proste "Znajdź i zamień" powinno wystarczyć.

Jeśli używałeś modułu `@vitest/browser/utils`, możesz teraz importować te narzędzia również z `vitest/browser`:

```ts
import { getElementError } from '@vitest/browser/utils' // [!code --]
import { utils } from 'vitest/browser' // [!code ++]
const { getElementError } = utils // [!code ++]
```

::: warning
Zarówno `@vitest/browser/context`, jak i `@vitest/browser/utils` działają w runtime podczas okresu przejściowego, ale zostaną usunięte w przyszłej wersji.
:::

### Przebudowa pool

Vitest używał [`tinypool`](https://github.com/tinylibs/tinypool) do orkiestracji sposobu uruchamiania plików testowych w workerach runnera testów. Tinypool kontrolował, jak złożone zadania takie jak równoległość, izolacja i komunikacja IPC działają wewnętrznie. Jednak stwierdziliśmy, że Tinypool ma pewne wady, które spowalniają rozwój Vitest. W Vitest v4 całkowicie usunęliśmy Tinypool i przepisaliśmy sposób działania pool bez nowych zależności. Przeczytaj więcej o rozumowaniu w [feat!: rewrite pools without tinypool #8705](https://github.com/vitest-dev/vitest/pull/8705).

Nowa architektura pool pozwala Vitest uprościć wiele wcześniej złożonych opcji konfiguracyjnych:

- `maxThreads` i `maxForks` to teraz `maxWorkers`.
- Zmienne środowiskowe `VITEST_MAX_THREADS` i `VITEST_MAX_FORKS` to teraz `VITEST_MAX_WORKERS`.
- `singleThread` i `singleFork` to teraz `maxWorkers: 1, isolate: false`. Jeśli testy polegały na resecie modułów między testami, trzeba dodać [setupFile](/config/setupfiles), który wywołuje [`vi.resetModules()`](/api/vi.html#vi-resetmodules) w [hooku `beforeAll`](/api/#beforeall).
- `poolOptions` zostało usunięte. Wszystkie poprzednie `poolOptions` są teraz opcjami najwyższego poziomu. `memoryLimit` pool VM zostało przemianowane na `vmMemoryLimit`.
- `threads.useAtomics` zostało usunięte. Jeśli masz przypadek użycia dla tego, otwórz nowe żądanie funkcji.
- Interfejs niestandardowego pool został przepisany, zobacz [Niestandardowy pool](/guide/advanced/pool#custom-pool)

```ts
export default defineConfig({
  test: {
    poolOptions: { // [!code --]
      forks: { // [!code --]
        execArgv: ['--expose-gc'], // [!code --]
        isolate: false, // [!code --]
        singleFork: true, // [!code --]
      }, // [!code --]
      vmThreads: { // [!code --]
        memoryLimit: '300Mb' // [!code --]
      }, // [!code --]
    }, // [!code --]
    execArgv: ['--expose-gc'], // [!code ++]
    isolate: false, // [!code ++]
    maxWorkers: 1, // [!code ++]
    vmMemoryLimit: '300Mb', // [!code ++]
  }
})
```

Wcześniej nie było możliwe określenie niektórych opcji związanych z pool dla projektu podczas używania [Vitest Projects](/guide/projects). Z nową architekturą to nie jest już blokadą.

::: code-group
```ts [Izolacja na projekt]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      {
        // Nieizolowane testy jednostkowe
        name: 'Unit tests',
        isolate: false,
        exclude: ['**.integration.test.ts'],
      },
      {
        // Izolowane testy integracyjne
        name: 'Integration tests',
        include: ['**.integration.test.ts'],
      },
    ],
  },
})
```
```ts [Projekty równoległe i sekwencyjne]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      {
        name: 'Parallel',
        exclude: ['**.sequential.test.ts'],
      },
      {
        name: 'Sequential',
        include: ['**.sequential.test.ts'],
        fileParallelism: false,
      },
    ],
  },
})
```
```ts [Opcje Node CLI na projekt]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      {
        name: 'Production env',
        execArgv: ['--env-file=.env.prod']
      },
      {
        name: 'Staging env',
        execArgv: ['--env-file=.env.staging']
      },
    ],
  },
})
```
:::

Zobacz [Przepisy](/guide/recipes) po więcej przykładów.

### Aktualizacje reporterów

API reporterów `onCollected`, `onSpecsCollected`, `onPathsCollected`, `onTaskUpdate` i `onFinished` zostały usunięte. Zobacz [`API Reporterów`](/api/advanced/reporters) po nowe alternatywy. Nowe API zostały wprowadzone w Vitest `v3.0.0`.

Reporter `basic` został usunięty, ponieważ jest równoważny z:

```ts
export default defineConfig({
  test: {
    reporters: [
      ['default', { summary: false }]
    ]
  }
})
```

Reporter [`verbose`](/guide/reporters#verbose-reporter) teraz drukuje przypadki testowe jako płaską listę. Aby przywrócić poprzednie zachowanie, użyj `--reporter=tree`:

```ts
export default defineConfig({
  test: {
    reporters: ['verbose'], // [!code --]
    reporters: ['tree'], // [!code ++]
  }
})
```

### Snapshoty używające Custom Elements drukują Shadow Root

W Vitest 4.0 snapshoty zawierające custom elements będą drukować zawartość shadow root. Aby przywrócić poprzednie zachowanie, ustaw [opcję `printShadowRoot`](/config/#snapshotformat) na `false`.

```js{15-22}
// przed Vite 4.0
exports[`custom element with shadow root 1`] = `
"<body>
  <div>
    <custom-element />
  </div>
</body>"
`

// po Vite 4.0
exports[`custom element with shadow root 1`] = `
"<body>
  <div>
    <custom-element>
      #shadow-root
        <span
          class="some-name"
          data-test-id="33"
          id="5"
        >
          hello
        </span>
    </custom-element>
  </div>
</body>"
`
```

### Przestarzałe API zostały usunięte

Vitest 4.0 usuwa niektóre przestarzałe API, w tym:

- Opcja konfiguracji `poolMatchGlobs`. Użyj zamiast tego [`projects`](/guide/projects).
- Opcja konfiguracji `environmentMatchGlobs`. Użyj zamiast tego [`projects`](/guide/projects).
- Opcje konfiguracji `deps.external`, `deps.inline`, `deps.fallbackCJS`. Użyj zamiast tego `server.deps.external`, `server.deps.inline` lub `server.deps.fallbackCJS`.
- Opcja konfiguracji `browser.testerScripts`. Użyj zamiast tego [`browser.testerHtmlPath`](/config/browser/testerhtmlpath).
- Opcja konfiguracji `minWorkers`. Tylko `maxWorkers` ma wpływ na sposób uruchamiania testów, więc usuwamy tę publiczną opcję.
- Vitest nie wspiera już podawania obiektu opcji testu jako trzeciego argumentu do `test` i `describe`. Użyj zamiast tego drugiego argumentu:

```ts
test('przykład', () => { /* ... */ }, { retry: 2 }) // [!code --]
test('przykład', { retry: 2 }, () => { /* ... */ }) // [!code ++]
```

Zauważ, że podawanie liczby timeout jako ostatniego argumentu jest nadal wspierane:

```ts
test('przykład', () => { /* ... */ }, 1000) // ✅
```

Ta wersja usuwa również wszystkie przestarzałe typy. To w końcu naprawia problem, gdzie Vitest przypadkowo ściągał `@types/node` (zobacz [#5481](https://github.com/vitest-dev/vitest/issues/5481) i [#6141](https://github.com/vitest-dev/vitest/issues/6141)).

## Migracja z Jest {#jest}

Vitest został zaprojektowany z API kompatybilnym z Jest, aby uczynić migrację z Jest tak prostą jak to możliwe. Pomimo tych wysiłków, możesz nadal napotkać następujące różnice:

### Globale jako domyślne

Jest ma domyślnie włączone [API globalne](https://jestjs.io/docs/api). Vitest nie. Możesz albo włączyć globale przez [ustawienie konfiguracji `globals`](/config/#globals), albo zaktualizować swój kod, aby używać importów z modułu `vitest`.

Jeśli zdecydujesz się zachować globale wyłączone, pamiętaj, że popularne biblioteki jak [`testing-library`](https://testing-library.com/) nie będą automatycznie uruchamiać [cleanup](https://testing-library.com/docs/svelte-testing-library/api/#cleanup) DOM.

### `mock.mockReset`

[`mockReset`](https://jestjs.io/docs/mock-function-api#mockfnmockreset) Jesta zastępuje implementację mocka pustą funkcją, która zwraca `undefined`.

[`mockReset`](/api/mock#mockreset) Vitest resetuje implementację mocka do jej oryginału.
Oznacza to, że resetowanie mocka utworzonego przez `vi.fn(impl)` zresetuje implementację mocka do `impl`.

### `mock.mock` jest trwały

Jest odtworzy stan mocka, gdy wywołane zostanie `.mockClear`, co oznacza, że zawsze musisz uzyskiwać do niego dostęp jako getter. Vitest natomiast przechowuje trwałą referencję do stanu, co oznacza, że możesz go ponownie użyć:

```ts
const mock = vi.fn()
const state = mock.mock
mock.mockClear()

expect(state).toBe(mock.mock) // nie powiedzie się w Jest
```

### Mocki modułów

Podczas mockowania modułu w Jest, wartość zwracana przez argument fabryki jest domyślnym eksportem. W Vitest argument fabryki musi zwrócić obiekt z każdym eksportem jawnie zdefiniowanym. Na przykład następujący `jest.mock` musiałby być zaktualizowany w następujący sposób:

```ts
jest.mock('./some-path', () => 'hello') // [!code --]
vi.mock('./some-path', () => ({ // [!code ++]
  default: 'hello', // [!code ++]
})) // [!code ++]
```

Po więcej szczegółów odwołaj się do [sekcji API `vi.mock`](/api/vi#vi-mock).

### Zachowanie auto-mockowania

W przeciwieństwie do Jest, mockowane moduły w `<root>/__mocks__` nie są ładowane, chyba że wywołane zostanie `vi.mock()`. Jeśli potrzebujesz, aby były mockowane w każdym teście, jak w Jest, możesz je mockować wewnątrz [`setupFiles`](/config/setupfiles).

### Importowanie oryginału mockowanego pakietu

Jeśli tylko częściowo mockujesz pakiet, mogłeś wcześniej używać funkcji Jesta `requireActual`. W Vitest powinieneś zastąpić te wywołania `vi.importActual`.

```ts
const { cloneDeep } = jest.requireActual('lodash/cloneDeep') // [!code --]
const { cloneDeep } = await vi.importActual('lodash/cloneDeep') // [!code ++]
```

### Rozszerzanie mockowania na zewnętrzne biblioteki

Tam gdzie Jest robi to domyślnie, podczas mockowania modułu i chęci rozszerzenia tego mockowania na inne zewnętrzne biblioteki używające tego samego modułu, należy jawnie określić, którą bibliotekę 3rd-party ma być mockowana, aby zewnętrzna biblioteka była częścią kodu źródłowego, używając [server.deps.inline](/config/#server-deps-inline).

```
server.deps.inline: ["lib-name"]
```

### expect.getState().currentTestName

Nazwy `test` Vitest są łączone symbolem `>`, aby ułatwić rozróżnienie testów od suite'ów, podczas gdy Jest używa pustej spacji (` `).

```diff
- `${describeTitle} ${testTitle}`
+ `${describeTitle} > ${testTitle}`
```

### Zmienne środowiskowe

Tak jak Jest, Vitest ustawia `NODE_ENV` na `test`, jeśli nie było ustawione wcześniej. Vitest ma również odpowiednik dla `JEST_WORKER_ID` o nazwie `VITEST_POOL_ID` (zawsze mniejszy lub równy `maxWorkers`), więc jeśli na nim polegasz, nie zapomnij go przemianować. Vitest eksponuje również `VITEST_WORKER_ID`, który jest unikalnym ID działającego workera - ta liczba nie jest dotknięta przez `maxWorkers` i będzie rosła z każdym utworzonym workerem.

### Zastępowanie właściwości

Jeśli chcesz zmodyfikować obiekt, użyjesz [API replaceProperty](https://jestjs.io/docs/jest-object#jestreplacepropertyobject-propertykey-value) w Jest, możesz użyć [`vi.stubEnv`](/api/#vi-stubenv) lub [`vi.spyOn`](/api/vi#vi-spyon), aby zrobić to samo również w Vitest.

### Callback done

Vitest nie wspiera stylu callback'owego deklarowania testów. Możesz przepisać je, aby używać funkcji `async`/`await`, lub użyć Promise, aby naśladować styl callback'owy.

<!--@include: ./examples/promise-done.md-->

### Hooki

Hooki `beforeAll`/`beforeEach` mogą zwracać [funkcję teardown](/api/#setup-and-teardown) w Vitest. Z tego powodu możesz potrzebować przepisać deklaracje swoich hooków, jeśli zwracają coś innego niż `undefined` lub `null`:

```ts
beforeEach(() => setActivePinia(createTestingPinia())) // [!code --]
beforeEach(() => { setActivePinia(createTestingPinia()) }) // [!code ++]
```

W Jest hooki są wywoływane sekwencyjnie (jeden po drugim). Domyślnie Vitest uruchamia hooki na stosie. Aby użyć zachowania Jesta, zaktualizuj opcję [`sequence.hooks`](/config/#sequence-hooks):

```ts
export default defineConfig({
  test: {
    sequence: { // [!code ++]
      hooks: 'list', // [!code ++]
    } // [!code ++]
  }
})
```

### Typy

Vitest nie ma odpowiednika przestrzeni nazw `jest`, więc musisz importować typy bezpośrednio z `vitest`:

```ts
let fn: jest.Mock<(name: string) => number> // [!code --]
import type { Mock } from 'vitest' // [!code ++]
let fn: Mock<(name: string) => number> // [!code ++]
```

### Timery

Vitest nie wspiera starszych timerów Jesta.

### Timeout

Jeśli używałeś `jest.setTimeout`, musisz migrować do `vi.setConfig`:

```ts
jest.setTimeout(5_000) // [!code --]
vi.setConfig({ testTimeout: 5_000 }) // [!code ++]
```

### Snapshoty Vue

To nie jest funkcja specyficzna dla Jesta, ale jeśli wcześniej używałeś Jesta z presetem vue-cli, musisz zainstalować pakiet [`jest-serializer-vue`](https://github.com/eddyerburgh/jest-serializer-vue) i określić go w [`snapshotSerializers`](/config/#snapshotserializers):

```js [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    snapshotSerializers: ['jest-serializer-vue']
  }
})
```

W przeciwnym razie snapshoty będą miały wiele escapowanych znaków `"`.
