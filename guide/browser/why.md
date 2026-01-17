---
title: Dlaczego Tryb Przeglądarki | Tryb Przeglądarki
outline: deep
---

# Dlaczego Tryb Przeglądarki

## Motywacja

Stworzyliśmy funkcję trybu przeglądarki Vitest, aby pomóc usprawnić przepływy pracy testowania i osiągnąć dokładniejsze i bardziej wiarygodne wyniki testów. Ten dodatek do naszego API testowego pozwala programistom uruchamiać testy w natywnym środowisku przeglądarki. W tej sekcji zbadamy motywacje stojące za tą funkcją i jej korzyści dla testowania.

### Różne sposoby testowania

Istnieją różne sposoby testowania kodu JavaScript. Niektóre frameworki testowe symulują środowiska przeglądarki w Node.js, podczas gdy inne uruchamiają testy w prawdziwych przeglądarkach. W tym kontekście [jsdom](https://www.npmjs.com/package/jsdom) jest przykładem implementacji specyfikacji, która symuluje środowisko przeglądarki, używana z runnerem testowym takim jak Jest lub Vitest, podczas gdy inne narzędzia testowe, takie jak [WebdriverIO](https://webdriver.io/) lub [Cypress](https://www.cypress.io/), pozwalają programistom testować aplikacje w prawdziwej przeglądarce, a w przypadku [Playwright](https://playwright.dev/) zapewniają silnik przeglądarki.

### Zastrzeżenia dotyczące symulacji

Testowanie programów JavaScript w symulowanych środowiskach, takich jak jsdom lub happy-dom, uprościło konfigurację testów i zapewniło łatwe w użyciu API, czyniąc je odpowiednimi dla wielu projektów i zwiększając pewność co do wyników testów. Jednak kluczowe jest pamiętanie, że te narzędzia tylko symulują środowisko przeglądarki, a nie prawdziwą przeglądarkę, co może skutkować rozbieżnościami między środowiskiem symulowanym a rzeczywistym. Dlatego mogą wystąpić fałszywie pozytywne lub negatywne wyniki testów.

Aby osiągnąć najwyższy poziom pewności w naszych testach, kluczowe jest testowanie w prawdziwym środowisku przeglądarki. Dlatego stworzyliśmy funkcję trybu przeglądarki w Vitest, pozwalając programistom uruchamiać testy natywnie w przeglądarce i uzyskiwać dokładniejsze i bardziej wiarygodne wyniki testów. Dzięki testowaniu na poziomie przeglądarki programiści mogą być bardziej pewni, że ich aplikacja będzie działać zgodnie z zamierzeniami w rzeczywistym scenariuszu.

## Wady

Podczas korzystania z trybu przeglądarki Vitest ważne jest uwzględnienie następujących wad:

### Wczesny etap rozwoju

Funkcja trybu przeglądarki Vitest jest wciąż na wczesnym etapie rozwoju. W związku z tym może nie być jeszcze w pełni zoptymalizowana i mogą występować błędy lub problemy, które nie zostały jeszcze rozwiązane. Zaleca się, aby użytkownicy uzupełnili swoje doświadczenie z trybem przeglądarki Vitest o samodzielny runner testowy po stronie przeglądarki, taki jak WebdriverIO, Cypress lub Playwright.

### Dłuższa inicjalizacja

Tryb przeglądarki Vitest wymaga uruchomienia dostawcy i przeglądarki podczas procesu inicjalizacji, co może zająć trochę czasu. Może to skutkować dłuższymi czasami inicjalizacji w porównaniu z innymi wzorcami testowania.
