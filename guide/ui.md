---
title: Vitest UI | Przewodnik
---

# Vitest UI

Napędzany przez Vite, Vitest ma również serwer deweloperski pod spodem podczas uruchamiania testów. To pozwala Vitest dostarczyć piękne UI do przeglądania i interakcji z testami. Vitest UI jest opcjonalne, więc trzeba je zainstalować:

```bash
npm i -D @vitest/ui
```

Następnie możesz uruchomić testy z UI, przekazując flagę `--ui`:

```bash
vitest --ui
```

Potem możesz odwiedzić Vitest UI pod adresem <a href="http://localhost:51204/__vitest__/">`http://localhost:51204/__vitest__/`</a>

::: warning
UI jest interaktywne i wymaga działającego serwera Vite, więc upewnij się, że uruchamiasz Vitest w trybie `watch` (domyślny). Alternatywnie możesz wygenerować statyczny raport HTML, który wygląda identycznie jak Vitest UI, określając `html` w opcji `reporters` konfiguracji.
:::

<img alt="Vitest UI" img-light src="/ui-1-light.png">
<img alt="Vitest UI" img-dark src="/ui-1-dark.png">

UI może być również używane jako reporter. Użyj reportera `'html'` w konfiguracji Vitest, aby wygenerować wyjście HTML i podejrzeć wyniki testów:

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    reporters: ['html'],
  },
})
```

Możesz sprawdzić swój raport pokrycia w Vitest UI: zobacz [Vitest UI Coverage](/guide/coverage#vitest-ui) po więcej szczegółów.

::: warning
Aby nadal widzieć, jak testy działają w czasie rzeczywistym w terminalu, dodaj reporter `default` do opcji `reporters`: `['default', 'html']`.
:::

::: tip
Aby podejrzeć swój raport HTML, możesz użyć polecenia [vite preview](https://vitejs.dev/guide/cli.html#vite-preview):

```sh
npx vite preview --outDir ./html
```

Możesz skonfigurować wyjście za pomocą opcji konfiguracji [`outputFile`](/config/#outputfile). Musisz tam określić ścieżkę `.html`. Na przykład `./html/index.html` jest wartością domyślną.
:::

## Graf modułów

Zakładka Graf modułów wyświetla graf modułów wybranego pliku testowego.

::: info
Wszystkie dostarczone obrazy używają repozytorium [Zammad](https://github.com/zammad/zammad) jako przykładu.
:::

<img alt="Widok grafu modułów" img-light src="/ui/light-module-graph.png">
<img alt="Widok grafu modułów" img-dark src="/ui/dark-module-graph.png">

Jeśli jest więcej niż 50 modułów, graf modułów wyświetla tylko pierwsze dwa poziomy grafu, aby zredukować wizualny bałagan. Zawsze możesz kliknąć ikonę "Show Full Graph", aby podejrzeć pełny graf.

<center>
  <img alt="Przycisk 'Show Full Graph' zlokalizowany blisko legendy" img-light src="/ui/light-ui-show-graph.png">
  <img alt="Przycisk 'Show Full Graph' zlokalizowany blisko legendy" img-dark src="/ui/dark-ui-show-graph.png">
</center>

::: warning
Zauważ, że jeśli graf jest zbyt duży, może zająć trochę czasu, zanim pozycje węzłów się ustabilizują.
:::

Zawsze możesz przywrócić wejściowy graf modułów, klikając "Reset". Aby rozwinąć graf modułów, kliknij prawym przyciskiem myszy lub przytrzymaj <kbd>Shift</kbd> podczas klikania węzła, który cię interesuje. Wyświetli wszystkie węzły związane z wybranym.

Domyślnie Vitest nie pokazuje modułów z `node_modules`. Zazwyczaj te moduły są eksternalizowane. Możesz je włączyć, odznaczając "Hide node_modules".

### Informacje o module

Klikając lewym przyciskiem myszy na węzeł modułu, otwierasz widok Informacji o module.

<img alt="Widok informacji o module dla modułu inline" img-light src="/ui/light-module-info.png">
<img alt="Widok informacji o module dla modułu inline" img-dark src="/ui/dark-module-info.png">

Ten widok jest podzielony na dwie części. Górna część pokazuje pełny ID modułu i pewne diagnostyki o module. Jeśli [`experimental.fsModuleCache`](/config/experimental#experimental-fsmodulecache) jest włączone, będzie badge "cached" lub "not cached". Po prawej możesz zobaczyć diagnostyki czasowe:

- Self Time: czas potrzebny na zaimportowanie modułu, wykluczając statyczne importy.
- Total Time: czas potrzebny na zaimportowanie modułu, włączając statyczne importy. Zauważ, że to nie obejmuje czasu `transform` bieżącego modułu.
- Transform: czas potrzebny na transformację modułu.

Jeśli otworzyłeś ten widok klikając na import, zobaczysz również przycisk "Back" na początku, który przeniesie cię do poprzedniego modułu.

Dolna część zależy od typu modułu. Jeśli moduł jest zewnętrzny, zobaczysz tylko kod źródłowy tego pliku. Nie będziesz mógł dalej przemierzać grafu modułów i nie zobaczysz, jak długo trwało importowanie statycznych importów.

<img alt="Widok informacji o module dla modułu zewnętrznego" img-light src="/ui/light-module-info-external.png">
<img alt="Widok informacji o module dla modułu zewnętrznego" img-dark src="/ui/dark-module-info-external.png">

Jeśli moduł był inline, zobaczysz trzy dodatkowe okna:

- Source: niezmieniony kod źródłowy modułu
- Transformed: transformowany kod, który Vitest wykonuje używając [module runner](https://vite.dev/guide/api-environment-runtimes#modulerunner) Vite
- Source Map (v3): mapowania source map

Wszystkie statyczne importy w oknie "Source" pokazują całkowity czas potrzebny do ich ewaluacji przez bieżący moduł. Jeśli import był już ewaluowany w grafie modułów, pokaże `0ms`, ponieważ jest cachowany w tym momencie.

Jeśli moduł potrzebował więcej niż 500 milisekund do załadowania, czas będzie wyświetlony na czerwono. Jeśli moduł potrzebował więcej niż 100 milisekund, czas będzie wyświetlony na pomarańczowo.

Możesz kliknąć na źródło importu, aby przejść do tego modułu i przemierzać graf dalej (zauważ `./support/assertions/index.ts` poniżej).

<img alt="Widok informacji o module dla modułu wewnętrznego" img-light src="/ui/light-module-info-traverse.png">
<img alt="Widok informacji o module dla modułu wewnętrznego" img-dark src="/ui/dark-module-info-traverse.png">

::: warning
Zauważ, że importy tylko typów nie są wykonywane w runtime i nie wyświetlają całkowitego czasu trwania. Nie mogą być również otwierane.
:::

Jeśli inny plugin wstrzykuje import modułu podczas transformacji, te importy będą wyświetlane na początku modułu w szarym kolorze (na przykład moduły wstrzyknięte przez `import.meta.glob`). Również pokazują całkowity czas i mogą być dalej przemierzane.

<img alt="Widok informacji o module dla modułu wewnętrznego" img-light src="/ui/light-module-info-shadow.png">
<img alt="Widok informacji o module dla modułu wewnętrznego" img-dark src="/ui/dark-module-info-shadow.png">

::: tip
Jeśli rozwijasz niestandardową integrację na bazie Vitest, możesz użyć [`vitest.experimental_getSourceModuleDiagnostic`](/api/advanced/vitest#getsourcemodulediagnostic), aby pobrać te informacje.
:::

### Rozkład importów

::: tip FEEDBACK
Proszę, zostaw opinię dotyczącą tej funkcji w [GitHub Discussion](https://github.com/vitest-dev/vitest/discussions/9224).
:::

Zakładka Graf modułów również dostarcza Rozkład importów z listą modułów, które potrzebują najdłuższego czasu do załadowania (domyślnie top 10, ale możesz nacisnąć "Show more", aby załadować 10 więcej), posortowanych według Total Time.

<img alt="Rozkład importów z listą top 10 modułów, które potrzebują najdłuższego czasu do załadowania" img-light src="/ui/light-import-breakdown.png">
<img alt="Rozkład importów z listą top 10 modułów, które potrzebują najdłuższego czasu do załadowania" img-dark src="/ui/dark-import-breakdown.png">

Możesz kliknąć na moduł, aby zobaczyć Informacje o module. Jeśli moduł jest zewnętrzny, będzie miał żółty kolor (taki sam kolor jak w grafie modułów).

Rozkład pokazuje listę modułów z self time, total time i procentem względem czasu potrzebnego do załadowania całego pliku testowego.

Ikona "Show Import Breakdown" będzie miała czerwony kolor, jeśli jest co najmniej jeden plik, który potrzebował więcej niż 500 milisekund do załadowania, i będzie pomarańczowa, jeśli jest co najmniej jeden plik, który potrzebował więcej niż 100 milisekund.

Domyślnie Vitest pokazuje rozkład automatycznie, jeśli jest co najmniej jeden moduł, który potrzebował więcej niż 500 milisekund do załadowania. Możesz kontrolować zachowanie, ustawiając opcję [`experimental.printImportBreakdown`](/config/experimental#experimental-printimportbreakdown).
