# Wsparcie Open Telemetry <Experimental /> {#open-telemetry-support}

::: tip FEEDBACK
Proszę, zostaw opinię na temat tej funkcji w [GitHub Discussion](https://github.com/vitest-dev/vitest/discussions/9222).
:::

::: tip Przykładowy projekt
[GitHub](https://github.com/vitest-dev/vitest/tree/main/examples/opentelemetry)
:::

Ślady [OpenTelemetry](https://opentelemetry.io/) mogą być przydatnym narzędziem do debugowania wydajności i zachowania twojej aplikacji wewnątrz testów.

Jeśli jest włączona, integracja Vitest generuje spany, które są ograniczone do workera twojego testu.

::: warning
Inicjalizacja OpenTelemetry zwiększa czas uruchomienia każdego testu, chyba że Vitest działa bez [izolacji](/config/isolate). Możesz to zobaczyć jako span `vitest.runtime.traces` wewnątrz `vitest.worker.start`.
:::

Aby zacząć używać OpenTelemetry w Vitest, określ ścieżkę modułu SDK za pomocą [`experimental.openTelemetry.sdkPath`](/config/experimental#experimental-opentelemetry) i ustaw `experimental.openTelemetry.enabled` na `true`. Vitest automatycznie zinstrumentuje cały proces i każdego indywidualnego workera testowego.

Upewnij się, że eksportujesz SDK jako domyślny eksport, aby Vitest mógł opróżnić żądania sieciowe przed zamknięciem procesu. Zauważ, że Vitest nie wywołuje automatycznie `start`.

## Szybki start

Przed podglądem śladów twojej aplikacji zainstaluj wymagane pakiety i określ ścieżkę do pliku instrumentacji w konfiguracji.

```shell
npm i @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node @opentelemetry/exporter-trace-otlp-proto
```

::: code-group
```js{12} [otel.js]
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-proto'
import { NodeSDK } from '@opentelemetry/sdk-node'

const sdk = new NodeSDK({
  serviceName: 'vitest',
  traceExporter: new OTLPTraceExporter(),
  instrumentations: [getNodeAutoInstrumentations()],
})

sdk.start()
export default sdk
```
```js [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    experimental: {
      openTelemetry: {
        enabled: true,
        sdkPath: './otel.js',
      },
    },
  },
})
```
:::

::: danger FAŁSZYWE TIMERY
Jeśli używasz fałszywych timerów, ważne jest, aby zresetować je przed zakończeniem testu, w przeciwnym razie ślady mogą nie być prawidłowo śledzone.
:::

Vitest nie przetwarza modułu `sdkPath`, więc ważne jest, aby SDK mogło być importowane w twoim środowisku Node.js. Idealne jest użycie rozszerzenia `.js` dla tego pliku. Użycie innego rozszerzenia spowolni twoje testy i może wymagać podania dodatkowych argumentów Node.js.

Jeśli chcesz podać plik TypeScript, upewnij się, że zapoznałeś się ze stroną [TypeScript](https://nodejs.org/api/typescript.html#type-stripping) w dokumentacji Node.js.

## Niestandardowe ślady

Możesz użyć API OpenTelemetry samodzielnie, aby śledzić określone operacje w swoim kodzie. Niestandardowe ślady automatycznie dziedziczą kontekst OpenTelemetry Vitest:

```ts
import { trace } from '@opentelemetry/api'
import { test } from 'vitest'
import { db } from './src/db'

const tracer = trace.getTracer('vitest')

test('db connects properly', async () => {
  // to jest widoczne wewnątrz spanu `vitest.test.runner.test.callback`
  await tracer.startActiveSpan('db.connect', () => db.connect())
})
```

## Tryb przeglądarki

Podczas uruchamiania testów w [trybie przeglądarki](/guide/browser/), Vitest propaguje kontekst śledzenia między Node.js a przeglądarką. Ślady po stronie Node.js (orkiestracja testów, komunikacja ze sterownikiem przeglądarki) są dostępne bez dodatkowej konfiguracji.

Aby przechwycić ślady z runtime przeglądarki, podaj SDK kompatybilne z przeglądarką za pomocą `browserSdkPath`:

```shell
npm i @opentelemetry/sdk-trace-web @opentelemetry/exporter-trace-otlp-proto
```

::: code-group
```js [otel-browser.js]
import {
  BatchSpanProcessor,
  WebTracerProvider,
} from '@opentelemetry/sdk-trace-web'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-proto'

const provider = new WebTracerProvider({
  spanProcessors: [
    new BatchSpanProcessor(new OTLPTraceExporter()),
  ],
})

provider.register()
export default provider
```
```js [vitest.config.js]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      instances: [{ browser: 'chromium' }],
    },
    experimental: {
      openTelemetry: {
        enabled: true,
        sdkPath: './otel.js',
        browserSdkPath: './otel-browser.js',
      },
    },
  },
})
```
:::

::: warning KONTEKST ASYNCHRONICZNY
W przeciwieństwie do Node.js, przeglądarki nie mają automatycznej propagacji kontekstu asynchronicznego. Vitest obsługuje to wewnętrznie dla wykonywania testów, ale niestandardowe spany w głęboko zagnieżdżonym kodzie asynchronicznym mogą nie propagować kontekstu automatycznie.
:::

## Przeglądanie śladów

Aby wygenerować ślady, uruchom Vitest jak zwykle. Możesz uruchomić Vitest w trybie watch lub trybie run. Vitest wywoła ręcznie `sdk.shutdown()` po zakończeniu wszystkiego, aby upewnić się, że ślady są prawidłowo obsługiwane.

Możesz przeglądać ślady używając dowolnych produktów open source lub komercyjnych, które wspierają API OpenTelemetry. Jeśli nie używałeś wcześniej OpenTelemetry, zalecamy rozpoczęcie od [Jaeger](https://www.jaegertracing.io/docs/2.11/getting-started/#all-in-one), ponieważ jest naprawdę łatwy w konfiguracji.

<img src="/otel-jaeger.png" />

## `@opentelemetry/api`

Vitest deklaruje `@opentelemetry/api` jako opcjonalną zależność peer, której używa wewnętrznie do generowania spanów. Gdy zbieranie śladów nie jest włączone, Vitest nie będzie próbował używać tej zależności.

Podczas konfigurowania Vitest do używania OpenTelemetry, zazwyczaj zainstalujesz `@opentelemetry/sdk-node`, który zawiera `@opentelemetry/api` jako zależność przechodnią, tym samym spełniając wymaganie zależności peer Vitest. Jeśli napotkasz błąd wskazujący, że `@opentelemetry/api` nie może być znalezione, zazwyczaj oznacza to, że zbieranie śladów nie zostało włączone. Jeśli błąd utrzymuje się po prawidłowej konfiguracji, możesz potrzebować zainstalować `@opentelemetry/api` jawnie.

## Propagacja kontekstu między procesami

Vitest wspiera automatyczną propagację kontekstu z procesów nadrzędnych za pomocą zmiennych środowiskowych `TRACEPARENT` i `TRACESTATE`, jak zdefiniowano w [specyfikacji OpenTelemetry](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/context/env-carriers.md). Jest to szczególnie przydatne podczas uruchamiania Vitest jako części większego rozproszonego systemu śledzenia (np. potoki CI/CD z instrumentacją OpenTelemetry).
