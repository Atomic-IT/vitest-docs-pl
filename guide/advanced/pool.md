# Niestandardowy pool <Badge type="danger">zaawansowane</Badge> {#custom-pool}

::: warning
To jest zaawansowane, eksperymentalne i bardzo niskopoziomowe API. Jeśli chcesz tylko [uruchamiać testy](/guide/), prawdopodobnie tego nie potrzebujesz. Jest przeznaczony głównie dla autorów bibliotek.
:::

Vitest uruchamia testy w pool. Domyślnie istnieje kilka runnerów pool:

- `threads` do uruchamiania testów używając `node:worker_threads` (izolacja jest zapewniana przez nowy kontekst workera)
- `forks` do uruchamiania testów używając `node:child_process` (izolacja jest zapewniana przez nowy proces `child_process.fork`)
- `vmThreads` do uruchamiania testów używając `node:worker_threads` (ale izolacja jest zapewniana przez moduł `vm` zamiast nowego kontekstu workera)
- `browser` do uruchamiania testów używając dostawców przeglądarek
- `typescript` do uruchamiania sprawdzania typów na testach

::: tip
Zobacz [`vitest-pool-example`](https://www.npmjs.com/package/vitest-pool-example) po przykład implementacji niestandardowego runnera pool.
:::

## Użycie

Możesz dostarczyć własny runner pool za pomocą funkcji zwracającej `PoolRunnerInitializer`.

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import customPool from './my-custom-pool.ts'

export default defineConfig({
  test: {
    // domyślnie uruchomi każdy plik z niestandardowym pool
    pool: customPool({
      customProperty: true,
    })
  },
})
```

Jeśli potrzebujesz uruchamiać testy w różnych pool, użyj funkcji [`projects`](/guide/projects):

```ts [vitest.config.ts]
import customPool from './my-custom-pool.ts'

export default defineConfig({
  test: {
    projects: [
      {
        extends: true,
        test: {
          pool: 'threads',
        },
      },
      {
        extends: true,
        test: {
          pool: customPool({
            customProperty: true,
          })
        }
      }
    ],
  },
})
```

## API

Opcja `pool` akceptuje `PoolRunnerInitializer`, który może być używany dla niestandardowych runnerów pool. Właściwość `name` powinna wskazywać nazwę niestandardowego runnera pool. Powinna być identyczna z właściwością `name` twojego workera.

```ts [my-custom-pool.ts]
import type { PoolRunnerInitializer } from 'vitest/node'

export function customPool(customOptions: CustomOptions): PoolRunnerInitializer {
  return {
    name: 'custom-pool',
    createPoolWorker: options => new CustomPoolWorker(options, customOptions),
  }
}
```

W swoim `CustomPoolWorker` musisz zdefiniować wszystkie wymagane metody:

```ts [my-custom-pool.ts]
import type { PoolOptions, PoolWorker, WorkerRequest } from 'vitest/node'

class CustomPoolWorker implements PoolWorker {
  name = 'custom-pool'
  private customOptions: CustomOptions

  constructor(options: PoolOptions, customOptions: CustomOptions) {
    this.customOptions = customOptions
  }

  send(message: WorkerRequest): void {
    // Zapewnij sposób wysyłania wiadomości do workera
  }

  on(event: string, callback: (arg: any) => void): void {
    // Zapewnij sposób nasłuchiwania zdarzeń workera, np. message, error, exit
  }

  off(event: string, callback: (arg: any) => void): void {
    // Zapewnij sposób wypisywania się z nasłuchiwaczy `on`
  }

  async start() {
    // zrób coś gdy worker jest uruchomiony
  }

  async stop() {
    // wyczyść stan
  }

  deserialize(data) {
    return data
  }
}
```

Twój `CustomPoolRunner` będzie kontrolować cykle życia niestandardowego workera runnera testów i kanał komunikacji. Na przykład twój `CustomPoolRunner` mógłby uruchomić `Worker` z `node:worker_threads` i zapewnić komunikację przez `Worker.postMessage` i `parentPort`.

W pliku workera możesz importować narzędzia pomocnicze z `vitest/worker`:

```ts [my-worker.ts]
import { init, runBaseTests, setupEnvironment } from 'vitest/worker'

init({
  post: (response) => {
    // Zapewnij sposób wysyłania tej wiadomości do onWorker CustomPoolRunner jako zdarzenie message
  },
  on: (callback) => {
    // Zapewnij sposób nasłuchiwania wywołań "postMessage" CustomPoolRunner
  },
  off: (callback) => {
    // Opcjonalnie, zapewnij sposób usuwania nasłuchiwaczy dodanych przez wywołania "on"
  },
  teardown: () => {
    // Opcjonalnie, zapewnij sposób zamknięcia workera, np. wypisanie wszystkich nasłuchiwaczy `on`
  },
  serialize: (value) => {
    // Opcjonalnie, zapewnij niestandardowy serializator dla wywołań `post`
  },
  deserialize: (value) => {
    // Opcjonalnie, zapewnij niestandardowy deserializator dla callbacków `on`
  },
  runTests: (state, traces) => runBaseTests('run', state, traces),
  collectTests: (state, traces) => runBaseTests('collect', state, traces),
  setup: setupEnvironment,
})
```
