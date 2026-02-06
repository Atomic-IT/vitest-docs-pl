---
title: Przepisy | Przewodnik
---

# Przepisy

## Wyłączanie izolacji tylko dla określonych plików testowych

Możesz przyspieszyć uruchamianie testów, wyłączając izolację dla określonego zestawu plików przez określenie `isolate` dla wpisów `projects`:

```ts [vitest.config.ts]
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

## Równoległe i sekwencyjne pliki testowe

Możesz podzielić pliki testowe na grupy równoległe i sekwencyjne, używając opcji `projects`:

```ts [vitest.config.ts]
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
