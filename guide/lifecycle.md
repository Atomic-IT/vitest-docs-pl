---
title: Cykl życia uruchomienia testów | Przewodnik
outline: deep
---

# Cykl życia uruchomienia testów

Zrozumienie cyklu życia uruchomienia testów jest kluczowe dla pisania efektywnych testów, debugowania problemów i optymalizacji zestawu testów. Ten przewodnik wyjaśnia, kiedy i w jakiej kolejności występują różne fazy cyklu życia w Vitest, od inicjalizacji po fazę czyszczenia.

## Przegląd

Typowe uruchomienie testów Vitest przechodzi przez następujące główne fazy:

1. **Inicjalizacja** - Ładowanie konfiguracji i przygotowanie projektu
2. **Globalne przygotowanie** - Jednorazowe przygotowanie przed uruchomieniem jakichkolwiek testów
3. **Tworzenie workerów** - Workery testowe są uruchamiane zgodnie z konfiguracją [pool](/config/pool)
4. **Zbieranie plików testowych** - Pliki testowe są odkrywane i organizowane
5. **Wykonywanie testów** - Testy są uruchamiane z ich hookami i asercjami
6. **Raportowanie** - Wyniki są zbierane i raportowane
7. **Globalne czyszczenie** - Końcowe czyszczenie po zakończeniu wszystkich testów

Fazy 4–6 uruchamiają się raz dla każdego pliku testowego, więc w całym zestawie testów będą wykonywane wielokrotnie i mogą również działać równolegle w różnych plikach, gdy używasz więcej niż [1 workera](/config/maxworkers).

## Szczegółowe fazy cyklu życia

### 1. Faza inicjalizacji

Gdy uruchamiasz `vitest`, framework najpierw ładuje konfigurację i przygotowuje środowisko testowe.

**Co się dzieje:**
- Argumenty [wiersza poleceń](/guide/cli) są parsowane
- [Plik konfiguracyjny](/config/) jest ładowany
- Struktura projektu jest walidowana

Ta faza może uruchomić się ponownie, jeśli plik konfiguracyjny lub jeden z jego importów się zmieni.

**Zakres:** Główny proces (przed utworzeniem jakichkolwiek workerów testowych)

### 2. Faza globalnego przygotowania

Jeśli skonfigurowałeś pliki [`globalSetup`](/config/globalsetup), są one uruchamiane raz przed utworzeniem jakichkolwiek workerów testowych.

**Co się dzieje:**
- Funkcje `setup()` (lub wyeksportowana funkcja `default`) z plików globalnego przygotowania wykonują się sekwencyjnie
- Wiele plików globalnego przygotowania uruchamia się w kolejności, w jakiej zostały zdefiniowane

**Zakres:** Główny proces (oddzielony od workerów testowych)

**Ważne uwagi:**
- Globalne przygotowanie działa w **innym zakresie globalnym** niż twoje testy
- Testy nie mają dostępu do zmiennych zdefiniowanych w globalnym przygotowaniu (użyj zamiast tego [`provide`/`inject`](/config/provide))
- Globalne przygotowanie uruchamia się tylko wtedy, gdy co najmniej jeden test jest w kolejce

```ts [globalSetup.ts]
export function setup(project) {
  // Uruchamia się raz przed wszystkimi testami
  console.log('Globalne przygotowanie')

  // Udostępnij dane testom
  project.provide('apiUrl', 'http://localhost:3000')
}

export function teardown() {
  // Uruchamia się raz po wszystkich testach
  console.log('Globalne czyszczenie')
}
```

### 3. Faza tworzenia workerów

Po zakończeniu globalnego przygotowania, Vitest tworzy workery testowe na podstawie [konfiguracji pool](/config/pool).

**Co się dzieje:**
- Workery są uruchamiane zgodnie z ustawieniem `browser.enabled` lub `pool` (`threads`, `forks`, `vmThreads` lub `vmForks`)
- Każdy worker otrzymuje własne izolowane środowisko (chyba że [izolacja](/config/isolate) jest wyłączona)
- Domyślnie workery nie są ponownie używane, aby zapewnić izolację. Workery są ponownie używane tylko jeśli:
  - [izolacja](/config/isolate) jest wyłączona
  - LUB pool to `vmThreads` lub `vmForks`, ponieważ [VM](https://nodejs.org/api/vm.html) zapewnia wystarczającą izolację

**Zakres:** Procesy/wątki workerów

### 4. Faza przygotowania pliku testowego

Przed uruchomieniem każdego pliku testowego, wykonywane są [pliki setup](/config/setupfiles).

**Co się dzieje:**
- Pliki setup uruchamiają się w tym samym procesie co twoje testy
- Domyślnie pliki setup uruchamiają się **równolegle** (konfigurowalne przez [`sequence.setupFiles`](/config/sequence#sequence-setupfiles))
- Pliki setup wykonują się przed **każdym plikiem testowym**
- Tutaj można zainicjalizować dowolny globalny _stan_ lub konfigurację

**Zakres:** Proces workera (taki sam jak twoje testy)

**Ważne uwagi:**
- Jeśli [izolacja](/config/isolate) jest wyłączona, pliki setup nadal uruchamiają się ponownie przed każdym plikiem testowym, aby wywołać efekty uboczne, ale zaimportowane moduły są cachowane
- Edycja pliku setup wywołuje ponowne uruchomienie wszystkich testów w trybie watch

```ts [setupFile.ts]
import { afterEach } from 'vitest'

// Uruchamia się przed każdym plikiem testowym
console.log('Wykonywanie pliku setup')

// Rejestruj hooki, które dotyczą wszystkich testów
afterEach(() => {
  cleanup()
})
```

### 5. Faza zbierania i wykonywania testów

To jest główna faza, w której twoje testy faktycznie się uruchamiają.

#### Kolejność wykonywania plików testowych

Pliki testowe są wykonywane na podstawie twojej konfiguracji:

- **Domyślnie sekwencyjnie** w ramach jednego workera
- Pliki będą uruchamiane **równolegle** w różnych workerach, konfigurowanych przez [`maxWorkers`](/config/maxworkers)
- Kolejność może być losowana za pomocą [`sequence.shuffle`](/config/sequence#sequence-shuffle) lub dostrojona za pomocą [`sequence.sequencer`](/config/sequence#sequence-sequencer)
- Długo działające testy zazwyczaj zaczynają się wcześniej (na podstawie cache), chyba że shuffle jest włączony

#### W ramach każdego pliku testowego

Wykonywanie przebiega w następującej kolejności:

1. **Kod na poziomie pliku** - Cały kod poza blokami `describe` uruchamia się natychmiast
2. **Zbieranie testów** - Bloki `describe` są przetwarzane, a testy są rejestrowane jako efekty uboczne importowania pliku testowego
3. **Hooki `beforeAll`** - Uruchamiają się raz przed jakimikolwiek testami w suite
4. **Dla każdego testu:**
   - Hooki `beforeEach` wykonują się (w zdefiniowanej kolejności lub na podstawie [`sequence.hooks`](/config/sequence#sequence-hooks))
   - Funkcja testu wykonuje się
   - Hooki `afterEach` wykonują się (domyślnie w odwrotnej kolejności z `sequence.hooks: 'stack'`)
   - Callbacki [`onTestFinished`](/api/#ontestfinished) uruchamiają się (zawsze w odwrotnej kolejności)
   - Jeśli test nie powiódł się: callbacki [`onTestFailed`](/api/#ontestfailed) uruchamiają się
   - Uwaga: jeśli ustawiono `repeats` lub `retry`, wszystkie te kroki są wykonywane ponownie
5. **Hooki `afterAll`** - Uruchamiają się raz po zakończeniu wszystkich testów w suite

**Przykładowy przebieg wykonania:**

```ts
// To uruchamia się natychmiast (faza zbierania)
console.log('Plik załadowany')

describe('User API', () => {
  // To uruchamia się natychmiast (faza zbierania)
  console.log('Suite zdefiniowany')

  beforeAll(() => {
    // Uruchamia się raz przed wszystkimi testami w tym suite
    console.log('beforeAll')
  })

  beforeEach(() => {
    // Uruchamia się przed każdym testem
    console.log('beforeEach')
  })

  test('tworzy użytkownika', () => {
    // Test wykonuje się
    console.log('test 1')
  })

  test('aktualizuje użytkownika', () => {
    // Test wykonuje się
    console.log('test 2')
  })

  afterEach(() => {
    // Uruchamia się po każdym teście
    console.log('afterEach')
  })

  afterAll(() => {
    // Uruchamia się raz po wszystkich testach w tym suite
    console.log('afterAll')
  })
})

// Wyjście:
// Plik załadowany
// Suite zdefiniowany
// beforeAll
// beforeEach
// test 1
// afterEach
// beforeEach
// test 2
// afterEach
// afterAll
```

#### Zagnieżdżone suite

Przy używaniu zagnieżdżonych bloków `describe`, hooki podążają za hierarchicznym wzorcem:

```ts
describe('zewnętrzny', () => {
  beforeAll(() => console.log('zewnętrzny beforeAll'))
  beforeEach(() => console.log('zewnętrzny beforeEach'))

  test('zewnętrzny test', () => console.log('zewnętrzny test'))

  describe('wewnętrzny', () => {
    beforeAll(() => console.log('wewnętrzny beforeAll'))
    beforeEach(() => console.log('wewnętrzny beforeEach'))

    test('wewnętrzny test', () => console.log('wewnętrzny test'))

    afterEach(() => console.log('wewnętrzny afterEach'))
    afterAll(() => console.log('wewnętrzny afterAll'))
  })

  afterEach(() => console.log('zewnętrzny afterEach'))
  afterAll(() => console.log('zewnętrzny afterAll'))
})

// Wyjście:
// zewnętrzny beforeAll
// zewnętrzny beforeEach
// zewnętrzny test
// zewnętrzny afterEach
// wewnętrzny beforeAll
// zewnętrzny beforeEach
// wewnętrzny beforeEach
// wewnętrzny test
// wewnętrzny afterEach (w trybie stack)
// zewnętrzny afterEach (w trybie stack)
// wewnętrzny afterAll
// zewnętrzny afterAll
```

#### Testy współbieżne

Przy używaniu `test.concurrent` lub [`sequence.concurrent`](/config/sequence#sequence-concurrent):

- Testy w tym samym pliku mogą uruchamiać się równolegle
- Każdy współbieżny test nadal uruchamia swoje własne hooki `beforeEach` i `afterEach`
- Użyj [kontekstu testu](/guide/test-context) dla współbieżnych snapshotów: `test.concurrent('nazwa', async ({ expect }) => {})`

### 6. Faza raportowania

W trakcie uruchomienia testów, reportery otrzymują zdarzenia cyklu życia i wyświetlają wyniki.

**Co się dzieje:**
- Reportery otrzymują zdarzenia w miarę postępu testów
- Wyniki są zbierane i formatowane
- Generowane są podsumowania testów
- Generowane są raporty pokrycia (jeśli włączone)

Szczegółowe informacje o cyklu życia reporterów znajdziesz w przewodniku [Reportery](/api/advanced/reporters).

### 7. Faza globalnego czyszczenia

Po zakończeniu wszystkich testów wykonywane są funkcje globalnego czyszczenia.

**Co się dzieje:**
- Funkcje `teardown()` z plików [`globalSetup`](/config/globalsetup) uruchamiają się
- Wiele funkcji teardown uruchamia się w **odwrotnej kolejności** niż ich setup
- W trybie watch, teardown uruchamia się przed zakończeniem procesu, nie między ponownymi uruchomieniami testów

**Zakres:** Główny proces

```ts [globalSetup.ts]
export function teardown() {
  // Wyczyść globalne zasoby
  console.log('Globalne czyszczenie zakończone')
}
```

## Cykl życia w różnych zakresach

Zrozumienie, gdzie wykonuje się kod, jest kluczowe dla uniknięcia typowych pułapek:

| Faza | Zakres | Dostęp do kontekstu testu | Uruchamia się |
|------|--------|---------------------------|---------------|
| Plik konfiguracyjny | Główny proces | ❌ Nie | Raz na uruchomienie Vitest |
| Globalne przygotowanie | Główny proces | ❌ Nie (użyj `provide`/`inject`) | Raz na uruchomienie Vitest |
| Pliki setup | Worker (tak samo jak testy) | ✅ Tak | Przed każdym plikiem testowym |
| Kod na poziomie pliku | Worker | ✅ Tak | Raz na plik testowy |
| `beforeAll` / `afterAll` | Worker | ✅ Tak | Raz na suite |
| `beforeEach` / `afterEach` | Worker | ✅ Tak | Na test |
| Funkcja testu | Worker | ✅ Tak | Raz (lub więcej z retry/repeats) |
| Globalne czyszczenie | Główny proces | ❌ Nie | Raz na uruchomienie Vitest |

## Cykl życia w trybie watch

W trybie watch cykl życia powtarza się z pewnymi różnicami:

1. **Początkowe uruchomienie** - Pełny cykl życia jak opisano powyżej
2. **Przy zmianie pliku:**
   - Rozpoczyna się nowe [uruchomienie testów](/api/advanced/reporters#ontestrunstart)
   - Tylko dotknięte pliki testowe są ponownie uruchamiane
   - [Pliki setup](/config/setupfiles) uruchamiają się ponownie dla tych plików testowych
   - [Globalne przygotowanie](/config/globalsetup) **nie** uruchamia się ponownie (użyj [`project.onTestsRerun`](/config/globalsetup#handling-test-reruns) dla logiki specyficznej dla ponownych uruchomień)
3. **Przy wyjściu:**
   - Wykonuje się globalne czyszczenie
   - Proces kończy się

## Uwagi dotyczące wydajności

Zrozumienie cyklu życia pomaga optymalizować wydajność testów:

- **Globalne przygotowanie** jest idealne dla kosztownych jednorazowych operacji (seedowanie bazy danych, uruchamianie serwera)
- **Pliki setup** uruchamiają się przed każdym plikiem testowym - unikaj ciężkich operacji tutaj, jeśli masz wiele plików testowych
- **`beforeAll`** jest lepsze niż `beforeEach` dla kosztownego przygotowania, które nie wymaga izolacji
- **Wyłączenie [izolacji](/config/isolate)** poprawia wydajność, ale pliki setup nadal wykonują się przed każdym plikiem
- **[Konfiguracja pool](/config/pool)** wpływa na zrównoleglenie i dostępne API

Wskazówki dotyczące poprawy wydajności znajdziesz w przewodniku [Poprawa wydajności](/guide/improving-performance).

## Powiązana dokumentacja

- [Konfiguracja globalnego przygotowania](/config/globalsetup)
- [Konfiguracja plików setup](/config/setupfiles)
- [Opcje sekwencjonowania testów](/config/sequence)
- [Konfiguracja izolacji](/config/isolate)
- [Konfiguracja pool](/config/pool)
- [Rozszerzanie reporterów](/guide/advanced/reporters) - dla zdarzeń cyklu życia reporterów
- [Referencja API testów](/api/) - dla API hooków i funkcji testowych
