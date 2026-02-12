# Mockowanie systemu plików

Mockowanie systemu plików zapewnia, że testy nie zależą od rzeczywistego systemu plików, czyniąc testy bardziej niezawodnymi i przewidywalnymi. Ta izolacja pomaga unikać efektów ubocznych z poprzednich testów. Pozwala testować warunki błędów i przypadki brzegowe, które mogłyby być trudne lub niemożliwe do odtworzenia z rzeczywistym systemem plików, takie jak problemy z uprawnieniami, scenariusze pełnego dysku czy błędy odczytu/zapisu.

Vitest nie dostarcza żadnego API do mockowania systemu plików od razu. Można użyć `vi.mock`, aby ręcznie mockować moduł `fs`, ale jest to trudne do utrzymania. Zamiast tego zalecamy użycie [`memfs`](https://www.npmjs.com/package/memfs). `memfs` tworzy system plików w pamięci, który symuluje operacje na systemie plików bez dotykania rzeczywistego dysku. To podejście jest szybkie i bezpieczne, unikając potencjalnych efektów ubocznych na prawdziwym systemie plików.

## Przykład

Aby automatycznie przekierować każde wywołanie `fs` do `memfs`, możesz utworzyć pliki `__mocks__/fs.cjs` i `__mocks__/fs/promises.cjs` w katalogu głównym swojego projektu:

::: code-group
```ts [__mocks__/fs.cjs]
// możemy również użyć `import`, ale wtedy
// każdy eksport powinien być jawnie zdefiniowany

const { fs } = require('memfs')
module.exports = fs
```

```ts [__mocks__/fs/promises.cjs]
// możemy również użyć `import`, ale wtedy
// każdy eksport powinien być jawnie zdefiniowany

const { fs } = require('memfs')
module.exports = fs.promises
```
:::

```ts [read-hello-world.js]
import { readFileSync } from 'node:fs'

export function readHelloWorld(path) {
  return readFileSync(path, 'utf-8')
}
```

```ts [hello-world.test.js]
import { beforeEach, expect, it, vi } from 'vitest'
import { fs, vol } from 'memfs'
import { readHelloWorld } from './read-hello-world.js'

// informujemy vitest, aby użył mocka fs z folderu __mocks__
// można to zrobić w pliku setup, jeśli fs powinien być zawsze mockowany
vi.mock('node:fs')
vi.mock('node:fs/promises')

beforeEach(() => {
  // resetujemy stan systemu plików w pamięci
  vol.reset()
})

it('powinien zwrócić poprawny tekst', () => {
  const path = '/hello-world.txt'
  fs.writeFileSync(path, 'hello world')

  const text = readHelloWorld(path)
  expect(text).toBe('hello world')
})

it('może zwrócić wartość wielokrotnie', () => {
  // możesz użyć vol.fromJSON, aby zdefiniować kilka plików
  vol.fromJSON(
    {
      './dir1/hw.txt': 'hello dir1',
      './dir2/hw.txt': 'hello dir2',
    },
    // domyślny cwd
    '/tmp',
  )

  expect(readHelloWorld('/tmp/dir1/hw.txt')).toBe('hello dir1')
  expect(readHelloWorld('/tmp/dir2/hw.txt')).toBe('hello dir2')
})
```
