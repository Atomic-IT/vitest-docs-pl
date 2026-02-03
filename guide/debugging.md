---
title: Debugowanie | Przewodnik
---

# Debugowanie

:::tip
Podczas debugowania testów możesz chcieć użyć następujących opcji:

- [`--test-timeout=0`](/guide/cli#testtimeout) aby zapobiec timeout testów przy zatrzymywaniu na breakpointach
- [`--no-file-parallelism`](/guide/cli#fileparallelism) aby zapobiec równoległemu uruchamianiu plików testowych

:::

## VS Code

Szybkim sposobem na debugowanie testów w VS Code jest przez `JavaScript Debug Terminal`. Otwórz nowy `JavaScript Debug Terminal` i uruchom `npm run test` lub `vitest` bezpośrednio. *to działa z każdym kodem uruchamianym w Node, więc będzie działać z większością frameworków testowych JS*

![image](https://user-images.githubusercontent.com/5594348/212169143-72bf39ce-f763-48f5-822a-0c8b2e6a8484.png)

Możesz również dodać dedykowaną konfigurację uruchamiania do debugowania pliku testowego w VS Code:

```json
{
  // Więcej informacji: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debuguj bieżący plik testowy",
      "autoAttachChildProcesses": true,
      "skipFiles": ["<node_internals>/**", "**/node_modules/**"],
      "program": "${workspaceRoot}/node_modules/vitest/vitest.mjs",
      "args": ["run", "${relativeFile}"],
      "smartStep": true,
      "console": "integratedTerminal"
    }
  ]
}
```

Następnie w zakładce debug upewnij się, że wybrano 'Debuguj bieżący plik testowy'. Możesz wtedy otworzyć plik testowy, który chcesz debugować, i nacisnąć F5, aby rozpocząć debugowanie.

### Tryb przeglądarki

Aby debugować [Vitest Browser Mode](/guide/browser/index.md), przekaż `--inspect` lub `--inspect-brk` w CLI lub zdefiniuj to w swojej konfiguracji Vitest:

::: code-group
```bash [CLI]
vitest --inspect-brk --browser --no-file-parallelism
```
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    inspectBrk: true,
    fileParallelism: false,
    browser: {
      provider: playwright(),
      instances: [{ browser: 'chromium' }]
    },
  },
})
```
:::

Domyślnie Vitest użyje portu `9229` jako portu debugowania. Możesz go nadpisać, przekazując wartość w `--inspect-brk`:

```bash
vitest --inspect-brk=127.0.0.1:3000 --browser --no-file-parallelism
```

Użyj następującej [konfiguracji Compound VSCode](https://code.visualstudio.com/docs/editor/debugging#_compound-launch-configurations) do uruchamiania Vitest i podłączania debuggera w przeglądarce:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Uruchom Vitest Browser",
      "program": "${workspaceRoot}/node_modules/vitest/vitest.mjs",
      "console": "integratedTerminal",
      "args": ["--inspect-brk", "--browser", "--no-file-parallelism"]
    },
    {
      "type": "chrome",
      "request": "attach",
      "name": "Podłącz do Vitest Browser",
      "port": 9229
    }
  ],
  "compounds": [
    {
      "name": "Debuguj Vitest Browser",
      "configurations": ["Podłącz do Vitest Browser", "Uruchom Vitest Browser"],
      "stopAll": true
    }
  ]
}
```

## IntelliJ IDEA

Utwórz konfigurację uruchamiania [vitest](https://www.jetbrains.com/help/idea/vitest.html#createRunConfigVitest). Użyj następujących ustawień, aby uruchomić wszystkie testy w trybie debug:

Ustawienie | Wartość
--- | ---
Working directory | `/path/to/your-project-root`

Następnie uruchom tę konfigurację w trybie debug. IDE zatrzyma się na breakpointach JS/TS ustawionych w edytorze.

## Node Inspector, np. Chrome DevTools

Vitest wspiera również debugowanie testów bez IDE. Jednak wymaga to, aby testy nie były uruchamiane równolegle. Użyj jednego z następujących poleceń, aby uruchomić Vitest.

```sh
# Aby uruchomić w pojedynczym workerze
vitest --inspect-brk --no-file-parallelism

# Aby uruchomić w trybie przeglądarki
vitest --inspect-brk --browser --no-file-parallelism
```

Gdy Vitest się uruchomi, zatrzyma wykonywanie i poczeka, aż otworzysz narzędzia deweloperskie, które mogą połączyć się z [Node.js inspector](https://nodejs.org/en/docs/guides/debugging-getting-started/). Możesz do tego użyć Chrome DevTools, otwierając `chrome://inspect` w przeglądarce.

W trybie watch możesz utrzymać debugger otwarty podczas ponownych uruchomień testów, używając opcji `--isolate false`.
