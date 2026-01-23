---
title: Filtrowanie testów | Przewodnik
---

# Filtrowanie testów

Filtrowanie, limity czasu, współbieżność dla zestawów i testów

## CLI

Możesz użyć CLI do filtrowania plików testowych według nazwy:

```bash
$ vitest basic
```

Wykona tylko pliki testowe zawierające `basic`, np.

```
basic.test.ts
basic-foo.test.ts
basic/foo.test.ts
```

Możesz również użyć opcji `-t, --testNamePattern <pattern>`, aby filtrować testy według pełnej nazwy. Może to być pomocne, gdy chcesz filtrować według nazwy zdefiniowanej w pliku, a nie według samej nazwy pliku.

Od Vitest 3 możesz również określić test według nazwy pliku i numeru linii:

```bash
$ vitest basic/foo.test.ts:10
```

::: warning
Zauważ, że Vitest wymaga pełnej nazwy pliku, aby ta funkcja działała. Może być względna do bieżącego katalogu roboczego lub absolutną ścieżką do pliku.

```bash
$ vitest basic/foo.js:10 # ✅
$ vitest ./basic/foo.js:10 # ✅
$ vitest /users/project/basic/foo.js:10 # ✅
$ vitest foo:10 # ❌
$ vitest ./basic/foo:10 # ❌
```

W tej chwili Vitest również nie obsługuje zakresów:

```bash
$ vitest basic/foo.test.ts:10, basic/foo.test.ts:25 # ✅
$ vitest basic/foo.test.ts:10-25 # ❌
```
:::

## Określanie limitu czasu

Możesz opcjonalnie przekazać limit czasu w milisekundach jako trzeci argument do testów. Domyślna wartość to [5 sekund](/config/#testtimeout).

```ts
import { test } from 'vitest'

test('name', async () => { /* ... */ }, 1000)
```

Hooki również mogą otrzymać limit czasu, z tym samym domyślnym limitem 5 sekund.

```ts
import { beforeAll } from 'vitest'

beforeAll(async () => { /* ... */ }, 1000)
```

## Pomijanie zestawów i testów

Użyj `.skip`, aby uniknąć uruchamiania określonych zestawów lub testów

```ts
import { assert, describe, it } from 'vitest'

describe.skip('pominięty zestaw', () => {
  it('test', () => {
    // Zestaw pominięty, brak błędu
    assert.equal(Math.sqrt(4), 3)
  })
})

describe('zestaw', () => {
  it.skip('pominięty test', () => {
    // Test pominięty, brak błędu
    assert.equal(Math.sqrt(4), 3)
  })
})
```

## Wybieranie zestawów i testów do uruchomienia

Użyj `.only`, aby uruchomić tylko określone zestawy lub testy

```ts
import { assert, describe, it } from 'vitest'

// Tylko ten zestaw (i inne oznaczone only) są uruchamiane
describe.only('zestaw', () => {
  it('test', () => {
    assert.equal(Math.sqrt(4), 3)
  })
})

describe('inny zestaw', () => {
  it('pominięty test', () => {
    // Test pominięty, ponieważ testy działają w trybie Only
    assert.equal(Math.sqrt(4), 3)
  })

  it.only('test', () => {
    // Tylko ten test (i inne oznaczone only) są uruchamiane
    assert.equal(Math.sqrt(4), 2)
  })
})
```

Uruchom Vitest z filtrem pliku i numerem linii:

```shell
vitest ./test/example.test.ts:5
```

```ts:line-numbers
import { assert, describe, it } from 'vitest'

describe('zestaw', () => {
  // Uruchom tylko ten test
  it('test', () => {
    assert.equal(Math.sqrt(4), 3)
  })
})
```

## Niezaimplementowane zestawy i testy

Użyj `.todo`, aby oznaczyć zestawy i testy, które powinny być zaimplementowane

```ts
import { describe, it } from 'vitest'

// W raporcie pojawi się wpis dla tego zestawu
describe.todo('niezaimplementowany zestaw')

// W raporcie pojawi się wpis dla tego testu
describe('zestaw', () => {
  it.todo('niezaimplementowany test')
})
```
