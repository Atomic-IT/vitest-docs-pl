# Poprawa wydajności

## Izolacja testów

Domyślnie Vitest uruchamia każdy plik testowy w izolowanym środowisku opartym na [pool](/config/#pool):

- Pool `threads` uruchamia każdy plik testowy w osobnym [`Worker`](https://nodejs.org/api/worker_threads.html#class-worker)
- Pool `forks` uruchamia każdy plik testowy w osobnym [forked child process](https://nodejs.org/api/child_process.html#child_processforkmodulepath-args-options)
- Pool `vmThreads` uruchamia każdy plik testowy w osobnym [kontekście VM](https://nodejs.org/api/vm.html#vmcreatecontextcontextobject-options), ale używa workerów do równoległości

To znacznie zwiększa czas testów, co może nie być pożądane dla projektów, które nie polegają na efektach ubocznych i prawidłowo czyszczą swój stan (co zwykle dotyczy projektów ze środowiskiem `node`). W takim przypadku wyłączenie izolacji poprawi szybkość twoich testów. Aby to zrobić, możesz podać flagę `--no-isolate` do CLI lub ustawić właściwość [`test.isolate`](/config/#isolate) w konfiguracji na `false`.

::: code-group
```bash [CLI]
vitest --no-isolate
```
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    isolate: false,
  },
})
```
:::

Możesz również wyłączyć izolację tylko dla określonych plików używając `projects`:

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    projects: [
      {
        name: 'Isolated',
        isolate: true, // (domyślna wartość)
        exclude: ['**.non-isolated.test.ts'],
      },
      {
        name: 'Non-isolated',
        isolate: false,
        include: ['**.non-isolated.test.ts'],
      }
    ]
  },
})
```

:::tip
Jeśli używasz pola `vmThreads`, nie możesz wyłączyć izolacji. Użyj zamiast tego pola `threads`, aby poprawić wydajność testów.
:::

Dla niektórych projektów może być również pożądane wyłączenie równoległości, aby poprawić czas uruchomienia. Aby to zrobić, podaj flagę `--no-file-parallelism` do CLI lub ustaw właściwość [`test.fileParallelism`](/config/#fileparallelism) w konfiguracji na `false`.

::: code-group
```bash [CLI]
vitest --no-file-parallelism
```
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    fileParallelism: false,
  },
})
```
:::

## Ograniczanie przeszukiwania katalogów

Możesz ograniczyć katalog roboczy podczas wyszukiwania plików przez Vitest używając opcji [`test.dir`](/config/#test-dir). Powinno to przyspieszyć wyszukiwanie, jeśli masz niepowiązane foldery i pliki w katalogu głównym.

## Pool

Domyślnie Vitest uruchamia testy w `pool: 'forks'`. Chociaż pool `'forks'` jest lepszy dla problemów z kompatybilnością ([zawieszające się procesy](/guide/common-errors.html#failed-to-terminate-worker) i [segfaults](/guide/common-errors.html#segfaults-and-native-code-errors)), może być nieco wolniejszy niż `pool: 'threads'` w większych projektach.

Możesz spróbować poprawić czas wykonywania testów, przełączając opcję `pool` w konfiguracji:

::: code-group
```bash [CLI]
vitest --pool=threads
```
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    pool: 'threads',
  },
})
```
:::

## Sharding

Sharding testów to proces dzielenia zestawu testów na grupy, czyli shardy. Może to być przydatne, gdy masz duży zestaw testów i wiele maszyn, które mogą równocześnie uruchamiać podzbiory tego zestawu.

Aby podzielić testy Vitest na wiele różnych uruchomień, użyj opcji [`--shard`](/guide/cli#shard) z opcją [`--reporter=blob`](/guide/reporters#blob-reporter):

```sh
vitest run --reporter=blob --shard=1/3 # 1. maszyna
vitest run --reporter=blob --shard=2/3 # 2. maszyna
vitest run --reporter=blob --shard=3/3 # 3. maszyna
```

> Vitest dzieli twoje _pliki testowe_, a nie przypadki testowe, na shardy. Jeśli masz 1000 plików testowych, opcja `--shard=1/4` uruchomi 250 plików testowych, niezależnie od tego, ile przypadków testowych zawierają poszczególne pliki.

Zbierz wyniki przechowywane w katalogu `.vitest-reports` z każdej maszyny i połącz je za pomocą opcji [`--merge-reports`](/guide/cli#merge-reports):

```sh
vitest run --merge-reports
```

::: details Przykład GitHub Actions
Ta konfiguracja jest również używana w https://github.com/vitest-tests/test-sharding.

```yaml
# Zainspirowane https://playwright.dev/docs/test-sharding
name: Tests
on:
  push:
    branches:
      - main
jobs:
  tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install pnpm
        uses: pnpm/action-setup@a7487c7e89a18df4991f7f222e4898a00d66ddda # v4.1.0

      - name: Install dependencies
        run: pnpm i

      - name: Run tests
        run: pnpm run test --reporter=blob --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}

      - name: Upload blob report to GitHub Actions Artifacts
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: blob-report-${{ matrix.shardIndex }}
          path: .vitest-reports/*
          include-hidden-files: true
          retention-days: 1

  merge-reports:
    if: ${{ !cancelled() }}
    needs: [tests]

    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install pnpm
        uses: pnpm/action-setup@a7487c7e89a18df4991f7f222e4898a00d66ddda # v4.1.0

      - name: Install dependencies
        run: pnpm i

      - name: Download blob reports from GitHub Actions Artifacts
        uses: actions/download-artifact@v4
        with:
          path: .vitest-reports
          pattern: blob-report-*
          merge-multiple: true

      - name: Merge reports
        run: npx vitest --merge-reports
```

:::

:::tip
Sharding testów może być również przydatny na maszynach z dużą liczbą CPU.

Vitest uruchamia tylko jeden serwer Vite w swoim głównym wątku. Pozostałe wątki są używane do uruchamiania plików testowych.
Na maszynie z dużą liczbą CPU główny wątek może stać się wąskim gardłem, ponieważ nie jest w stanie obsłużyć wszystkich żądań przychodzących z wątków. Na przykład na maszynie z 32 CPU główny wątek jest odpowiedzialny za obsługę obciążenia z 31 wątków testowych.

Aby zmniejszyć obciążenie serwera Vite w głównym wątku, możesz użyć shardingu testów. Obciążenie może być rozłożone na wiele serwerów Vite.

```sh
# Przykład dzielenia testów na 32 CPU na 4 shardy.
# Ponieważ każdy proces potrzebuje 1 głównego wątku, jest 7 wątków dla runnerów testów (1+7)*4 = 32
# Użyj VITEST_MAX_WORKERS:
VITEST_MAX_WORKERS=7 vitest run --reporter=blob --shard=1/4 & \
VITEST_MAX_WORKERS=7 vitest run --reporter=blob --shard=2/4 & \
VITEST_MAX_WORKERS=7 vitest run --reporter=blob --shard=3/4 & \
VITEST_MAX_WORKERS=7 vitest run --reporter=blob --shard=4/4 & \
wait # https://man7.org/linux/man-pages/man2/waitpid.2.html

vitest run --merge-reports
```

:::
