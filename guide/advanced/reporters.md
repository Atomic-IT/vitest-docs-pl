# Rozszerzanie reporterów <Badge type="danger">zaawansowane</Badge> {#extending-reporters}

::: warning
To jest zaawansowane API. Jeśli chcesz tylko skonfigurować wbudowane reportery, przeczytaj przewodnik ["Reportery"](/guide/reporters).
:::

Możesz importować reportery z `vitest/reporters` i rozszerzać je, aby tworzyć niestandardowe reportery.

## Rozszerzanie wbudowanych reporterów

Ogólnie nie trzeba tworzyć reportera od podstaw. `vitest` zawiera kilka domyślnych reporterów, które można rozszerzyć.

```ts
import { DefaultReporter } from 'vitest/reporters'

export default class MyDefaultReporter extends DefaultReporter {
  // zrób coś
}
```

Oczywiście możesz stworzyć swojego reportera od podstaw. Po prostu rozszerz klasę `BaseReporter` i zaimplementuj potrzebne metody.

A oto przykład niestandardowego reportera:

```ts [custom-reporter.js]
import { BaseReporter } from 'vitest/reporters'

export default class CustomReporter extends BaseReporter {
  onTestModuleCollected() {
    const files = this.ctx.state.getFiles(this.watchFilters)
    this.reportTestSummary(files)
  }
}
```

Lub zaimplementuj interfejs `Reporter`:

```ts [custom-reporter.js]
import type { Reporter } from 'vitest/node'

export default class CustomReporter implements Reporter {
  onTestModuleCollected() {
    // wydrukuj coś
  }
}
```

Następnie możesz użyć swojego niestandardowego reportera w pliku `vitest.config.ts`:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'
import CustomReporter from './custom-reporter.js'

export default defineConfig({
  test: {
    reporters: [new CustomReporter()],
  },
})
```

## Raportowane zadania

Zamiast używać zadań, które otrzymują reportery, zaleca się używanie API Reported Tasks.

Możesz uzyskać dostęp do tego API wywołując `vitest.state.getReportedEntity(runnerTask)`:

```ts twoslash
import type { Reporter, TestModule } from 'vitest/node'

class MyReporter implements Reporter {
  onTestRunEnd(testModules: ReadonlyArray<TestModule>) {
    for (const testModule of testModules) {
      for (const task of testModule.children) {
        //                          ^?
        console.log('test run end', task.type, task.fullName)
      }
    }
  }
}
```

## Eksportowane reportery

`vitest` zawiera kilka [wbudowanych reporterów](/guide/reporters), których możesz używać od razu.

### Wbudowane reportery:

1. `DefaultReporter`
2. `DotReporter`
3. `JsonReporter`
4. `VerboseReporter`
5. `TapReporter`
6. `JUnitReporter`
7. `TapFlatReporter`
8. `HangingProcessReporter`
9. `TreeReporter`

### Bazowe abstrakcyjne reportery:

1. `BaseReporter`

### Reportery interfejsowe:

1. `Reporter`
