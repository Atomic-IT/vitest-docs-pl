---
title: Porównania z innymi runnerami testów | Przewodnik
---

# Porównania z innymi runnerami testów

## Jest

[Jest](https://jestjs.io/) przejął przestrzeń frameworków testowych, zapewniając gotowe wsparcie dla większości projektów JavaScript, wygodne API (`it` i `expect`) oraz pełen pakiet funkcji testowych, których wymaga większość konfiguracji (snapshoty, mocki, pokrycie). Jesteśmy wdzięczni zespołowi Jest i społeczności za stworzenie przyjemnego API testowego i promowanie wielu wzorców testowych, które są teraz standardem w ekosystemie webowym.

Możliwe jest używanie Jest w konfiguracjach Vite. [@sodatea](https://bsky.app/profile/haoqun.dev) zbudował [vite-jest](https://github.com/sodatea/vite-jest#readme), który ma na celu zapewnienie pierwszorzędnej integracji Vite dla [Jest](https://jestjs.io/). Ostatnie [blokady w Jest](https://github.com/sodatea/vite-jest/blob/main/packages/vite-jest/README.md#vite-jest) zostały rozwiązane, więc jest to prawidłowa opcja dla twoich testów jednostkowych.

Jednak w świecie, gdzie [Vite](https://vitejs.dev) zapewnia wsparcie dla najpopularniejszych narzędzi webowych (TypeScript, JSX, najpopularniejsze frameworki UI), Jest reprezentuje duplikację złożoności. Jeśli twoja aplikacja jest napędzana przez Vite, posiadanie dwóch różnych pipeline'ów do konfiguracji i utrzymania nie jest uzasadnione. Z Vitest możesz zdefiniować konfigurację dla swoich środowisk dev, build i test jako jeden pipeline, dzieląc te same pluginy i ten sam vite.config.js.

Nawet jeśli twoja biblioteka nie używa Vite (na przykład, jeśli jest budowana z esbuild lub Rollup), Vitest jest interesującą opcją, ponieważ daje ci szybsze uruchamianie testów jednostkowych i skok w DX dzięki domyślnemu trybowi watch używającemu natychmiastowego Hot Module Reload (HMR) Vite. Vitest oferuje kompatybilność z większością API Jest i bibliotek ekosystemu, więc w większości projektów powinien być zamiennikiem drop-in dla Jest.

## Cypress

[Cypress](https://www.cypress.io/) jest runnerem testów opartym na przeglądarce i narzędziem uzupełniającym do Vitest. Jeśli chciałbyś używać Cypress, sugerujemy używanie Vitest dla całej logiki headless w twojej aplikacji i Cypress dla całej logiki opartej na przeglądarce.

Cypress jest znany jako narzędzie do testów end-to-end, ale ich [nowy runner testów komponentów](https://on.cypress.io/component) ma świetne wsparcie dla testowania komponentów Vite i jest idealnym wyborem do testowania czegokolwiek, co renderuje się w przeglądarce.

Runnery oparte na przeglądarce, takie jak Cypress, WebdriverIO i Web Test Runner, wyłapią problemy, których Vitest nie może, ponieważ używają prawdziwej przeglądarki i prawdziwych API przeglądarki.

Driver testowy Cypress jest skupiony na określaniu, czy elementy są widoczne, dostępne i interaktywne. Cypress jest celowo zbudowany do rozwoju i testowania UI, a jego DX jest skoncentrowane wokół testowego prowadzenia twoich wizualnych komponentów. Widzisz swój komponent wyrenderowany obok reportera testów. Po zakończeniu testu komponent pozostaje interaktywny i możesz debugować wszelkie błędy, które wystąpią, używając devtools przeglądarki.

W przeciwieństwie do tego, Vitest jest skupiony na dostarczaniu najlepszego możliwego DX dla błyskawicznego, *headless* testowania. Runnery oparte na Node, takie jak Vitest, wspierają różne częściowo zaimplementowane środowiska przeglądarki, takie jak `jsdom`, które implementują wystarczająco dużo, aby szybko testować jednostkowo dowolny kod, który odwołuje się do API przeglądarki. Kompromisem jest to, że te środowiska przeglądarki mają ograniczenia w tym, co mogą zaimplementować. Na przykład [jsdom brakuje wielu funkcji](https://github.com/jsdom/jsdom/issues?q=is%3Aissue+is%3Aopen+sort%3Acomments-desc) takich jak `window.navigation` lub silnik layoutu (`offsetTop`, itp).

Na koniec, w przeciwieństwie do Web Test Runner, runner testowy Cypress jest bardziej jak IDE niż runner testowy, ponieważ widzisz również prawdziwy wyrenderowany komponent w przeglądarce, wraz z wynikami testów i logami.

Cypress również [integruje Vite w swoich produktach](https://www.youtube.com/watch?v=7S5cbY8iYLk): przebudowując UI swojej aplikacji używając [Vitesse](https://github.com/antfu/vitesse) i używając Vite do testowego prowadzenia rozwoju swojego projektu.

Uważamy, że Cypress nie jest dobrą opcją do testowania jednostkowego kodu headless, ale że używanie Cypress (do E2E i testów komponentów) i Vitest (do testów jednostkowych) pokryje potrzeby testowe twojej aplikacji.

## WebdriverIO

[WebdriverIO](https://webdriver.io/) jest, podobnie jak Cypress, alternatywnym runnerem testów opartym na przeglądarce i narzędziem uzupełniającym do Vitest. Może być używany jako narzędzie do testów end-to-end, jak również do testowania [komponentów webowych](https://webdriver.io/docs/component-testing). Używa nawet komponentów Vitest pod spodem, np. do [mockowania i stubbowania](https://webdriver.io/docs/mocksandspies/) w testach komponentów.

WebdriverIO ma te same zalety co Cypress, pozwalając ci testować twoją logikę w prawdziwej przeglądarce. Jednak używa rzeczywistych [standardów webowych](https://w3c.github.io/webdriver/) do automatyzacji, co przezwycięża niektóre kompromisy i ograniczenia podczas uruchamiania testów w Cypress. Ponadto pozwala ci uruchamiać testy na urządzeniach mobilnych, dając dostęp do testowania twojej aplikacji w jeszcze większej liczbie środowisk.

## Web Test Runner

[@web/test-runner](https://modern-web.dev/docs/test-runner/overview/) uruchamia testy wewnątrz headless przeglądarki, zapewniając to samo środowisko wykonawcze co twoja aplikacja webowa bez potrzeby mockowania API przeglądarki lub DOM. To również umożliwia debugowanie wewnątrz prawdziwej przeglądarki używając devtools, chociaż nie ma wyświetlanego UI do krokowego przechodzenia przez test, jak w testach Cypress.

Aby użyć @web/test-runner z projektem Vite, użyj [@remcovaes/web-test-runner-vite-plugin](https://github.com/remcovaes/web-test-runner-vite-plugin). @web/test-runner nie zawiera bibliotek asercji ani mockowania, więc to od ciebie zależy, czy je dodasz.

## uvu

[uvu](https://github.com/lukeed/uvu) jest runnerem testów dla Node.js i przeglądarki. Uruchamia testy w jednym wątku, więc testy nie są izolowane i mogą przeciekać między plikami. Vitest jednak używa wątków workerów do izolowania testów i uruchamiania ich równolegle.

Do transformacji kodu uvu polega na hookach require i loader. Vitest używa [Vite](https://vitejs.dev), więc pliki są transformowane z pełną mocą systemu pluginów Vite. W świecie, gdzie Vite zapewnia wsparcie dla najpopularniejszych narzędzi webowych (TypeScript, JSX, najpopularniejsze frameworki UI), uvu reprezentuje duplikację złożoności. Jeśli twoja aplikacja jest napędzana przez Vite, posiadanie dwóch różnych pipeline'ów do konfiguracji i utrzymania nie jest uzasadnione. Z Vitest możesz zdefiniować konfigurację dla swoich środowisk dev, build i test jako jeden pipeline, dzieląc te same pluginy i tę samą konfigurację.

uvu nie zapewnia inteligentnego trybu watch do ponownego uruchamiania zmienionych testów, podczas gdy Vitest daje niesamowite DX dzięki domyślnemu trybowi watch używającemu natychmiastowego Hot Module Reload (HMR) Vite.

uvu jest szybką opcją do uruchamiania prostych testów, ale Vitest może być szybszy i bardziej niezawodny dla bardziej złożonych testów i projektów.

## Mocha

[Mocha](https://mochajs.org) jest frameworkiem testowym działającym na Node.js i w przeglądarce. Mocha jest popularnym wyborem do testowania po stronie serwera. Mocha jest wysoce konfigurowalna i domyślnie nie zawiera pewnych funkcji. Na przykład nie zawiera biblioteki asercji, z założeniem, że wbudowany runner asercji Node jest wystarczająco dobry dla większości przypadków użycia. Innym popularnym wyborem do asercji z Mocha jest [Chai](https://www.chaijs.com).

Vitest również zapewnia gotową konfigurację dla kilku innych funkcji, które wymagają dodatkowej konfiguracji lub dodania innych bibliotek w Mocha, na przykład:

- Testowanie snapshotów
- TypeScript
- Wsparcie JSX
- Pokrycie kodu
- Mockowanie
- Inteligentny tryb watch (ponownie uruchamia tylko dotknięte testy)

Chociaż Mocha wspiera natywne ESM, ma ograniczenia i ograniczenia konfiguracyjne. Tryb watch nie działa z plikami ES Module, na przykład.

Pod względem wydajności, Mocha domyślnie uruchamia testy szeregowo, ale wspiera wykonywanie równoległe z flagą `--parallel` (chociaż niektóre reportery i funkcje nie działają w trybie równoległym).

Jeśli już używasz Vite w swoim pipeline'ie budowania, Vitest pozwala ci ponownie użyć tej samej konfiguracji i pluginów do testowania, podczas gdy Mocha wymagałaby osobnej konfiguracji testów. Vitest zapewnia API kompatybilne z Jest, jednocześnie wspierając znaną składnię `describe`, `it` i hooków Mocha, co sprawia, że migracja jest prosta dla większości zestawów testów.

Mocha pozostaje solidnym wyborem dla projektów, które potrzebują minimalnego, elastycznego runnera testów z pełną kontrolą nad swoim stosem testowym. Jednak jeśli chcesz nowoczesnego doświadczenia testowania ze wszystkim zawartym od razu - szczególnie dla aplikacji napędzanych Vite - Vitest cię obsłuży.

## Playwright

[Playwright](https://playwright.dev) jest frameworkiem testowym od Microsoft, który wyróżnia się w testach end-to-end w wielu przeglądarkach (Chromium, Firefox i WebKit). Kontroluje prawdziwe przeglądarki do testowania kompletnych przepływów użytkownika - od logowania i nawigacji po twojej aplikacji do wysyłania formularzy i weryfikowania wyników. Vitest natomiast jest zoptymalizowany do szybkich, izolowanych testów jednostkowych i komponentów w środowisku headless. Te różnice czynią go idealnym uzupełnieniem dla Vitest.

Standardową konfiguracją jest używanie Vitest do wszystkich testów jednostkowych i komponentów (logika biznesowa, narzędzia, hooki i testy komponentów UI), a Playwright do testów end-to-end, które weryfikują krytyczne ścieżki użytkownika i kompatybilność między przeglądarkami. Ta kombinacja daje ci szybki feedback podczas rozwoju z Vitest, jednocześnie zapewniając, że twoja kompletna aplikacja działa poprawnie w prawdziwych przeglądarkach z Playwright.

Vitest niedawno wprowadził [tryb przeglądarki](https://vitest.dev/guide/browser), który uruchamia testy w prawdziwych przeglądarkach. Jednak istnieją kluczowe różnice architektoniczne: testy komponentów Playwright działają w procesie Node.js i kontrolują przeglądarkę zdalnie. Tryb przeglądarki Vitest uruchamia testy natywnie w przeglądarce, utrzymując spójność z runnerem testów Vitest i doświadczeniem programisty, ale ma pewne [ograniczenia](https://vitest.dev/guide/browser/#limitations).
