# Widok śledzenia

Tryb Przeglądarki Vitest obsługuje generowanie [plików trace](https://playwright.dev/docs/trace-viewer#viewing-remote-traces) Playwright. Aby włączyć śledzenie, musisz ustawić opcję [`trace`](/config/browser/trace) w konfiguracji `test.browser`.

::: warning
Generowanie plików trace jest dostępne tylko przy użyciu [dostawcy Playwright](/config/browser/playwright).
:::

::: code-group
```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      provider: playwright(),
      trace: 'on',
    },
  },
})
```
```bash [CLI]
vitest --browser.trace=on
```
:::

Domyślnie Vitest wygeneruje plik trace dla każdego testu. Możesz również skonfigurować go, aby generował ślady tylko przy niepowodzeniach testów, ustawiając `trace` na `'on-first-retry'`, `'on-all-retries'` lub `'retain-on-failure'`. Pliki zostaną zapisane w folderze `__traces__` obok Twoich plików testowych. Nazwa trace zawiera nazwę projektu, nazwę testu, [liczbę `repeats` i liczbę `retry`](/api/#test-api-reference):

```
chromium-my-test-0-0.trace.zip
^^^^^^^^ nazwa projektu
         ^^^^^^ nazwa testu
                ^ liczba powtórzeń
                  ^ liczba ponowień
```

Aby zmienić katalog wyjściowy, możesz ustawić opcję `tracesDir` w konfiguracji `test.browser.trace`. W ten sposób wszystkie trace będą przechowywane w tym samym katalogu, pogrupowane według pliku testowego.

```ts [vitest.config.js]
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  test: {
    browser: {
      provider: playwright(),
      trace: {
        mode: 'on',
        // ścieżka jest względna do katalogu głównego projektu
        tracesDir: './playwright-traces',
      },
    },
  },
})
```

Ślady są dostępne w reporterach jako [adnotacje](/guide/test-annotations). Na przykład w reporterze HTML możesz znaleźć link do pliku trace w szczegółach testu.

## Podgląd

Aby otworzyć plik trace, możesz użyć Playwright Trace Viewer. Uruchom następującą komendę w terminalu:

```bash
npx playwright show-trace "ścieżka-do-pliku-trace"
```

Spowoduje to uruchomienie Trace Viewer i załadowanie określonego pliku trace.

Alternatywnie możesz otworzyć Trace Viewer w przeglądarce pod adresem https://trace.playwright.dev i przesłać tam plik trace.

## Ograniczenia

W tej chwili Vitest nie może wypełnić zakładki "Sources" w Trace Viewer. Oznacza to, że chociaż możesz zobaczyć akcje i zrzuty ekranu przechwycone podczas testu, nie będziesz mógł zobaczyć kodu źródłowego swoich testów bezpośrednio w Trace Viewer. Będziesz musiał wrócić do edytora kodu, aby zobaczyć implementację testu.
