---
title: Dlaczego Vitest | Przewodnik
---

# Dlaczego Vitest

:::tip UWAGA
Ten przewodnik zakłada, że znasz Vite. Dobrym sposobem na rozpoczęcie nauki jest przeczytanie [Przewodnika Dlaczego Vite](https://vitejs.dev/guide/why.html) oraz obejrzenie [Next generation frontend tooling with ViteJS](https://www.youtube.com/watch?v=UJypSr8IhKY), streamu, na którym [Evan You](https://bsky.app/profile/evanyou.me) zrobił demo wyjaśniające główne koncepcje.
:::

## Potrzeba Natywnego Runnera Testowego dla Vite

Wbudowane wsparcie Vite dla powszechnych wzorców webowych, funkcje takie jak importy glob i prymitywy SSR, oraz liczne wtyczki i integracje tworzą dynamiczny ekosystem. Historia developmentu i budowania to klucz do jego sukcesu. Dla dokumentacji istnieje kilka alternatyw opartych na SSG napędzanych przez Vite. Jednak historia testów jednostkowych Vite nie była jasna. Istniejące opcje jak [Jest](https://jestjs.io/) zostały stworzone w innym kontekście. Istnieje wiele duplikacji między Jest a Vite, zmuszając użytkowników do konfigurowania dwóch różnych pipeline'ów.

Używanie serwera deweloperskiego Vite do transformacji plików podczas testowania umożliwia stworzenie prostego runnera, który nie musi zajmować się złożonością transformacji plików źródłowych i może skupić się wyłącznie na zapewnieniu najlepszego DX podczas testowania. Runner testowy, który używa tej samej konfiguracji co Twoja aplikacja (przez `vite.config.js`), dzieląc wspólny pipeline transformacji podczas developmentu, budowania i testowania. Rozszerzalny tym samym API wtyczek, które pozwala Tobie i opiekunom Twoich narzędzi zapewnić pierwszorzędną integrację z Vite. Narzędzie, które od początku zostało zbudowane z myślą o Vite, wykorzystując jego ulepszenia w DX, takie jak natychmiastowy Hot Module Reload (HMR). To jest Vitest, framework testowy nowej generacji napędzany przez Vite.

Biorąc pod uwagę masową adopcję Jest, Vitest zapewnia kompatybilne API, które pozwala używać go jako bezpośredniego zamiennika w większości projektów. Zawiera również najczęściej wymagane funkcje podczas konfigurowania testów jednostkowych (mockowanie, snapshoty, pokrycie kodu). Vitest bardzo dba o wydajność i używa wątków Worker do uruchamiania jak największej ilości testów równolegle. Niektóre porty odnotowały uruchamianie testów o rząd wielkości szybciej. Tryb obserwacji jest domyślnie włączony, dostosowując się do sposobu, w jaki Vite promuje doświadczenie "najpierw development". Pomimo wszystkich tych ulepszeń w DX, Vitest pozostaje lekki, starannie dobierając swoje zależności (lub bezpośrednio włączając potrzebne fragmenty).

**Vitest ma na celu pozycjonowanie się jako Runner Testowy z wyboru dla projektów Vite i jako solidna alternatywa nawet dla projektów nieużywających Vite.**

Kontynuuj czytanie w [Przewodniku Rozpoczęcie Pracy](./index)

## Czym Vitest Różni się od X?

Możesz sprawdzić sekcję [Porównania](./comparisons), aby uzyskać więcej szczegółów na temat tego, jak Vitest różni się od innych podobnych narzędzi.
