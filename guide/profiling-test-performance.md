# Profilowanie wydajności testów

Kiedy uruchamiasz Vitest, raportuje on wiele metryk czasowych twoich testów:

> ```bash
> RUN  v2.1.1 /x/vitest/examples/profiling
>
> ✓ test/prime-number.test.ts (1) 4517ms
>   ✓ generate prime number 4517ms
>
> Test Files  1 passed (1)
>      Tests  1 passed (1)
>   Start at  09:32:53
>   Duration  4.80s (transform 44ms, setup 0ms, import 35ms, tests 4.52s, environment 0ms)
>   # Metryki czasowe ^^
> ```

- Transform: Ile czasu zajęła transformacja plików. Zobacz [Transformacja plików](#file-transform).
- Setup: Czas spędzony na uruchamianiu plików [`setupFiles`](/config/setupfiles).
- Import: Czas potrzebny na zaimportowanie plików testowych i ich zależności. Obejmuje to również czas spędzony na zbieraniu wszystkich testów. Zauważ, że nie obejmuje to dynamicznych importów wewnątrz testów.
- Tests: Czas spędzony na faktycznym uruchamianiu przypadków testowych.
- Environment: Czas spędzony na konfiguracji testowego [`environment`](/config/#environment), na przykład JSDOM.

## Runner testów

W przypadkach, gdy czas wykonania testu jest wysoki, możesz wygenerować profil runnera testów. Zobacz dokumentację NodeJS dla następujących opcji:

- [`--cpu-prof`](https://nodejs.org/api/cli.html#--cpu-prof)
- [`--heap-prof`](https://nodejs.org/api/cli.html#--heap-prof)
- [`--prof`](https://nodejs.org/api/cli.html#--prof)

:::warning
Opcja `--prof` nie działa z `pool: 'threads'` z powodu ograniczeń `node:worker_threads`.
:::

Aby przekazać te opcje do runnera testów Vitest, zdefiniuj `execArgv` w swojej konfiguracji Vitest:

```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    fileParallelism: false,
    execArgv: [
      '--cpu-prof',
      '--cpu-prof-dir=test-runner-profile',
      '--heap-prof',
      '--heap-prof-dir=test-runner-profile'
    ],
  },
})
```

Po zakończeniu testów powinny zostać wygenerowane pliki `test-runner-profile/*.cpuprofile` i `test-runner-profile/*.heapprofile`. Zobacz [Inspekcja rekordów profilowania](#inspecting-profiling-records) po instrukcje analizy tych plików.

Zobacz [Profilowanie | Przykłady](https://github.com/vitest-dev/vitest/tree/main/examples/profiling) po przykład.

## Wątek główny

Profilowanie wątku głównego jest przydatne do debugowania użycia Vite przez Vitest i plików [`globalSetup`](/config/#globalsetup).
To również miejsce, gdzie działają twoje pluginy Vite.

:::tip
Zobacz [Wydajność | Vite](https://vitejs.dev/guide/performance.html) po więcej wskazówek dotyczących profilowania specyficznego dla Vite.

Zalecamy [`vite-plugin-inspect`](https://github.com/antfu-collective/vite-plugin-inspect) do profilowania wydajności twoich pluginów Vite.
:::

Aby to zrobić, musisz przekazać argumenty do procesu Node, który uruchamia Vitest.

```bash
$ node --cpu-prof --cpu-prof-dir=main-profile ./node_modules/vitest/vitest.mjs --run
#      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^                                  ^^^^^
#               Argumenty NodeJS                                           Argumenty Vitest
```

Po zakończeniu testów powinien zostać wygenerowany plik `main-profile/*.cpuprofile`. Zobacz [Inspekcja rekordów profilowania](#inspecting-profiling-records) po instrukcje analizy tych plików.

## Transformacja plików

Ta strategia profilowania jest dobrym sposobem na identyfikację niepotrzebnych transformacji spowodowanych [plikami barrel](https://vitejs.dev/guide/performance.html#avoid-barrel-files).
Jeśli te logi zawierają pliki, które nie powinny być ładowane podczas uruchamiania testu, możesz mieć pliki barrel, które niepotrzebnie importują pliki.

Możesz również użyć [Vitest UI](/guide/ui), aby debugować spowolnienia spowodowane plikami barrel.
Poniższy przykład pokazuje, jak importowanie plików bez pliku barrel redukuje liczbę transformowanych plików o ~85%.

::: code-group
``` [Drzewo plików]
├── src
│   └── utils
│       ├── currency.ts
│       ├── formatters.ts  <-- Plik do testowania
│       ├── index.ts
│       ├── location.ts
│       ├── math.ts
│       ├── time.ts
│       └── users.ts
├── test
│   └── formatters.test.ts
└── vitest.config.ts
```
```ts [example.test.ts]
import { expect, test } from 'vitest'
import { formatter } from '../src/utils' // [!code --]
import { formatter } from '../src/utils/formatters' // [!code ++]

test('formatter works', () => {
  expect(formatter).not.toThrow()
})
```
:::

<img src="/module-graph-barrel-file.png" alt="Vitest UI demonstrujące problemy z plikami barrel" />

Aby zobaczyć, jak pliki są transformowane, możesz użyć zmiennej środowiskowej `VITEST_DEBUG_DUMP`, aby zapisać transformowane pliki w systemie plików:

```bash
$ VITEST_DEBUG_DUMP=true vitest --run

 RUN  v2.1.1 /x/vitest/examples/profiling
...

$ ls .vitest-dump/
_x_examples_profiling_global-setup_ts-1292904907.js
_x_examples_profiling_test_prime-number_test_ts-1413378098.js
_src_prime-number_ts-525172412.js
```

## Pokrycie kodu

Jeśli generowanie pokrycia kodu jest wolne w twoim projekcie, możesz użyć zmiennej środowiskowej `DEBUG=vitest:coverage`, aby włączyć logowanie wydajności.

```bash
$ DEBUG=vitest:coverage vitest --run --coverage

 RUN  v3.1.1 /x/vitest-example

  vitest:coverage Reading coverage results 2/2
  vitest:coverage Converting 1/2
  vitest:coverage 4 ms /x/src/multiply.ts
  vitest:coverage Converting 2/2
  vitest:coverage 552 ms /x/src/add.ts
  vitest:coverage Uncovered files 1/2
  vitest:coverage File "/x/src/large-file.ts" is taking longer than 3s # [!code error]
  vitest:coverage 3027 ms /x/src/large-file.ts
  vitest:coverage Uncovered files 2/2
  vitest:coverage 4 ms /x/src/untested-file.ts
  vitest:coverage Generate coverage total time 3521 ms
```

To podejście do profilowania jest świetne do wykrywania dużych plików, które są przypadkowo wybierane przez dostawców pokrycia.
Na przykład, jeśli twoja konfiguracja przypadkowo uwzględnia duże zbudowane zminifikowane pliki Javascript w pokryciu kodu, powinny pojawić się w logach.
W takich przypadkach możesz chcieć dostosować opcje [`coverage.include`](/config/#coverage-include) i [`coverage.exclude`](/config/#coverage-exclude).

## Inspekcja rekordów profilowania

Możesz sprawdzić zawartość plików `*.cpuprofile` i `*.heapprofile` za pomocą różnych narzędzi. Zobacz listę poniżej po przykłady.

- [Speedscope](https://www.speedscope.app/)
- [Profilowanie wydajności JavaScript w Visual Studio Code](https://code.visualstudio.com/docs/nodejs/profiling#_analyzing-a-profile)
- [Profiluj wydajność Node.js za pomocą panelu Performance | developer.chrome.com](https://developer.chrome.com/docs/devtools/performance/nodejs#analyze)
- [Przegląd panelu Memory | developer.chrome.com](https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots#view_snapshots)
