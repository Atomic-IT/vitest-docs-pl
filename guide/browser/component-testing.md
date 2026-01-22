---
title: Testowanie komponentów | Przewodnik
outline: deep
---

# Testowanie komponentów

Testowanie komponentów to strategia testowania, która koncentruje się na testowaniu pojedynczych komponentów UI w izolacji. W przeciwieństwie do testów end-to-end, które testują całe przepływy użytkownika, testy komponentów weryfikują, że każdy komponent działa poprawnie samodzielnie, dzięki czemu są szybsze do uruchomienia i łatwiejsze do debugowania.

Vitest zapewnia kompleksowe wsparcie dla testowania komponentów w wielu frameworkach, w tym Vue, React, Svelte, Lit, Preact, Qwik, Solid, Marko i innych. Ten przewodnik obejmuje konkretne wzorce, narzędzia i najlepsze praktyki dla efektywnego testowania komponentów z Vitest.

## Dlaczego testowanie komponentów?

Testowanie komponentów znajduje się pomiędzy testami jednostkowymi a testami end-to-end, oferując kilka zalet:

- **Szybsza informacja zwrotna** - Testuj pojedyncze komponenty bez ładowania całych aplikacji
- **Izolowane testowanie** - Skup się na zachowaniu komponentu bez zewnętrznych zależności
- **Lepsze debugowanie** - Łatwiej zlokalizować problemy w konkretnych komponentach
- **Kompleksowe pokrycie** - Łatwiej testować przypadki brzegowe i stany błędów

## Tryb Przeglądarki dla testowania komponentów

Testowanie komponentów w Vitest używa **Trybu Przeglądarki** do uruchamiania testów w prawdziwych środowiskach przeglądarki przy użyciu Playwright, WebdriverIO lub trybu preview. Zapewnia to najbardziej dokładne środowisko testowe, ponieważ Twoje komponenty działają w prawdziwych przeglądarkach z rzeczywistymi implementacjami DOM, renderowaniem CSS i API przeglądarki.

### Dlaczego Tryb Przeglądarki?

Tryb Przeglądarki jest zalecanym podejściem do testowania komponentów, ponieważ zapewnia najbardziej dokładne środowisko testowe. W przeciwieństwie do bibliotek symulujących DOM, Tryb Przeglądarki wykrywa rzeczywiste problemy, które mogą wpływać na Twoich użytkowników.

::: tip
Tryb Przeglądarki wykrywa problemy, które biblioteki symulujące DOM mogą przeoczyć, w tym:
- Problemy z układem CSS i stylami
- Rzeczywiste zachowanie API przeglądarki
- Dokładne obsługiwanie zdarzeń i ich propagacja
- Właściwe zarządzanie fokusem i funkcje dostępności

:::

### Cel tego przewodnika

Ten przewodnik koncentruje się konkretnie na **wzorcach i najlepszych praktykach testowania komponentów** przy użyciu możliwości Vitest. Chociaż wiele przykładów używa Trybu Przeglądarki (ponieważ jest to zalecane podejście), skupiamy się tutaj na strategiach testowania specyficznych dla komponentów, a nie na szczegółach konfiguracji przeglądarki.

Szczegółową konfigurację przeglądarki, opcje konfiguracji i zaawansowane funkcje przeglądarki znajdziesz w [dokumentacji Trybu Przeglądarki](/guide/browser/).

## Co sprawia, że test komponentu jest dobry

Dobre testy komponentów skupiają się na **zachowaniu i doświadczeniu użytkownika**, a nie na szczegółach implementacji:

- **Testuj kontrakt** - Jak komponenty otrzymują dane wejściowe (props) i produkują dane wyjściowe (zdarzenia, renderowanie)
- **Testuj interakcje użytkownika** - Kliknięcia, wysyłanie formularzy, nawigacja klawiaturą
- **Testuj przypadki brzegowe** - Stany błędów, stany ładowania, puste stany
- **Unikaj testowania wewnętrzności** - Zmienne stanu, prywatne metody, klasy CSS

### Hierarchia testowania komponentów

```
1. Krytyczne ścieżki użytkownika → Zawsze je testuj
2. Obsługa błędów               → Testuj scenariusze awarii
3. Przypadki brzegowe           → Puste dane, ekstremalne wartości
4. Dostępność                   → Czytniki ekranu, nawigacja klawiaturą
5. Wydajność                    → Duże zbiory danych, animacje
```

## Strategie testowania komponentów

### Strategia izolacji

Testuj komponenty w izolacji, mockując zależności:

```tsx
// Dla żądań API zalecamy MSW (Mock Service Worker)
// Zobacz: https://vitest.dev/guide/mocking/requests
//
// vi.mock(import('../api/userService'), () => ({
//   fetchUser: vi.fn().mockResolvedValue({ name: 'John' })
// }))

// Mockuj komponenty potomne, aby skupić się na logice rodzica
vi.mock(import('../components/UserCard'), () => ({
  default: vi.fn(({ user }) => `<div>User: ${user.name}</div>`)
}))

test('UserProfile handles loading and data states', async () => {
  const { getByText } = render(<UserProfile userId="123" />)

  // Testuj stan ładowania
  await expect.element(getByText('Loading...')).toBeInTheDocument()

  // Testuj załadowanie danych (expect.element automatycznie ponawia próby)
  await expect.element(getByText('User: John')).toBeInTheDocument()
})
```

### Strategia integracji

Testuj współpracę komponentów i przepływ danych:

```tsx
test('ProductList filters and displays products correctly', async () => {
  const mockProducts = [
    { id: 1, name: 'Laptop', category: 'Electronics', price: 999 },
    { id: 2, name: 'Book', category: 'Education', price: 29 }
  ]

  const { getByLabelText, getByText } = render(
    <ProductList products={mockProducts} />
  )

  // Początkowo pokazuje wszystkie produkty
  await expect.element(getByText('Laptop')).toBeInTheDocument()
  await expect.element(getByText('Book')).toBeInTheDocument()

  // Filtruj według kategorii
  await userEvent.selectOptions(
    getByLabelText(/category/i),
    'Electronics'
  )

  // Powinny pozostać tylko elektronika
  await expect.element(getByText('Laptop')).toBeInTheDocument()
  await expect.element(queryByText('Book')).not.toBeInTheDocument()
})
```

## Integracja z Testing Library

Chociaż Vitest zapewnia oficjalne pakiety dla popularnych frameworków ([`vitest-browser-vue`](https://www.npmjs.com/package/vitest-browser-vue), [`vitest-browser-react`](https://www.npmjs.com/package/vitest-browser-react), [`vitest-browser-svelte`](https://www.npmjs.com/package/vitest-browser-svelte)), możesz zintegrować się z [Testing Library](https://testing-library.com/) dla frameworków, które nie są jeszcze oficjalnie wspierane.

### Kiedy używać Testing Library

- Twój framework nie ma jeszcze oficjalnego pakietu Vitest browser
- Migrujesz istniejące testy, które używają Testing Library
- Preferujesz API Testing Library dla konkretnych scenariuszy testowych

### Wzorzec integracji

Kluczem jest użycie `page.elementLocator()` do połączenia wyjścia DOM Testing Library z API trybu przeglądarki Vitest:

```jsx
// Dla komponentów Solid.js
import { render } from '@testing-library/solid'
import { page } from 'vitest/browser'

test('Solid component handles user interaction', async () => {
  // Użyj Testing Library do renderowania komponentu
  const { baseElement, getByRole } = render(() =>
    <Counter initialValue={0} />
  )

  // Połącz z trybem przeglądarki Vitest dla interakcji i asercji
  const screen = page.elementLocator(baseElement)

  // Użyj zapytań page Vitest do znajdowania elementów
  const incrementButton = screen.getByRole('button', { name: /increment/i })

  // Użyj asercji i interakcji Vitest
  await expect.element(screen.getByText('Count: 0')).toBeInTheDocument()

  // Wyzwól interakcję użytkownika używając API page Vitest
  await incrementButton.click()

  await expect.element(screen.getByText('Count: 1')).toBeInTheDocument()
})
```

### Dostępne pakiety Testing Library

Popularne pakiety Testing Library, które dobrze współpracują z Vitest:

- [`@testing-library/solid`](https://github.com/solidjs/solid-testing-library) - Dla Solid.js
- [`@marko/testing-library`](https://testing-library.com/docs/marko-testing-library/intro) - Dla Marko
- [`@testing-library/svelte`](https://testing-library.com/docs/svelte-testing-library/intro) - Alternatywa dla [`vitest-browser-svelte`](https://www.npmjs.com/package/vitest-browser-svelte)
- [`@testing-library/vue`](https://testing-library.com/docs/vue-testing-library/intro) - Alternatywa dla [`vitest-browser-vue`](https://www.npmjs.com/package/vitest-browser-vue)

::: tip Ścieżka migracji
Jeśli Twój framework otrzyma oficjalne wsparcie Vitest później, możesz stopniowo migrować, zastępując funkcję `render` Testing Library, zachowując większość logiki testów.
:::

## Najlepsze praktyki

### 1. Używaj Trybu Przeglądarki dla CI/CD
Upewnij się, że testy działają w prawdziwych środowiskach przeglądarki dla najbardziej dokładnego testowania. Tryb Przeglądarki zapewnia dokładne renderowanie CSS, prawdziwe API przeglądarki i właściwą obsługę zdarzeń.

### 2. Testuj interakcje użytkownika
Symuluj rzeczywiste zachowanie użytkownika używając [API Interaktywności](/api/browser/interactivity) Vitest. Używaj metod `page.getByRole()` i `userEvent`, jak pokazano w naszych [Zaawansowanych wzorcach testowania](#advanced-testing-patterns):

```tsx
// Dobrze: Testuj rzeczywiste interakcje użytkownika
await page.getByRole('button', { name: /submit/i }).click()
await page.getByLabelText(/email/i).fill('user@example.com')

// Unikaj: Testowania szczegółów implementacji
// component.setState({ email: 'user@example.com' })
```

### 3. Testuj dostępność
Upewnij się, że komponenty działają dla wszystkich użytkowników, testując nawigację klawiaturą, zarządzanie fokusem i atrybuty ARIA. Zobacz nasz przykład [Testowanie dostępności](#testing-accessibility) dla praktycznych wzorców:

```tsx
// Testuj nawigację klawiaturą
await userEvent.keyboard('{Tab}')
await expect.element(document.activeElement).toHaveFocus()

// Testuj atrybuty ARIA
await expect.element(modal).toHaveAttribute('aria-modal', 'true')
```

### 4. Mockuj zewnętrzne zależności
Skup testy na logice komponentu, mockując API i zewnętrzne usługi. To sprawia, że testy są szybsze i bardziej niezawodne. Zobacz naszą [Strategię izolacji](#isolation-strategy) dla przykładów:

```tsx
// Dla żądań API zalecamy używanie MSW (Mock Service Worker)
// Zobacz: https://vitest.dev/guide/mocking/requests
// Zapewnia to bardziej realistyczne mockowanie żądań/odpowiedzi

// Do mockowania modułów użyj składni import()
vi.mock(import('../components/UserCard'), () => ({
  default: vi.fn(() => <div>Mocked UserCard</div>)
}))
```

### 5. Używaj znaczących opisów testów
Pisz opisy testów, które wyjaśniają oczekiwane zachowanie, a nie szczegóły implementacji:

```tsx
// Dobrze: Opisuje zachowanie widoczne dla użytkownika
test('shows error message when email format is invalid')
test('disables submit button while form is submitting')

// Unikaj: Opisów skupionych na implementacji
test('calls validateEmail function')
test('sets isSubmitting state to true')
```

## Zaawansowane wzorce testowania

### Testowanie zarządzania stanem komponentu

```tsx
// Testowanie komponentów stanowych i przejść stanu
test('ShoppingCart manages items correctly', async () => {
  const { getByText, getByTestId } = render(<ShoppingCart />)

  // Początkowo pusty
  await expect.element(getByText('Your cart is empty')).toBeInTheDocument()

  // Dodaj element
  await page.getByRole('button', { name: /add laptop/i }).click()

  // Zweryfikuj zmianę stanu
  await expect.element(getByText('1 item')).toBeInTheDocument()
  await expect.element(getByText('Laptop - $999')).toBeInTheDocument()

  // Testuj aktualizacje ilości
  await page.getByRole('button', { name: /increase quantity/i }).click()
  await expect.element(getByText('2 items')).toBeInTheDocument()
})
```

### Testowanie komponentów asynchronicznych z pobieraniem danych

```tsx
// Opcja 1: Zalecana - Użyj MSW (Mock Service Worker) do mockowania API
import { http, HttpResponse } from 'msw'
import { setupWorker } from 'msw/browser'

// Skonfiguruj worker MSW z handlerami API
const worker = setupWorker(
  http.get('/api/users/:id', ({ params }) => {
    // Opisz happy path
    return HttpResponse.json({ id: params.id, name: 'John Doe', email: 'john@example.com' })
  })
)

// Uruchom worker przed wszystkimi testami
beforeAll(() => worker.start())
afterEach(() => worker.resetHandlers())
afterAll(() => worker.stop())

test('UserProfile handles loading, success, and error states', async () => {
  // Testuj stan sukcesu
  const { getByText } = render(<UserProfile userId="123" />)
  // expect.element automatycznie ponawia próby, aż elementy zostaną znalezione
  await expect.element(getByText('John Doe')).toBeInTheDocument()
  await expect.element(getByText('john@example.com')).toBeInTheDocument()

  // Testuj stan błędu, nadpisując handler dla tego testu
  worker.use(
    http.get('/api/users/:id', () => {
      return HttpResponse.json({ error: 'User not found' }, { status: 404 })
    })
  )

  const { getByText: getErrorText } = render(<UserProfile userId="999" />)
  await expect.element(getErrorText('Error: User not found')).toBeInTheDocument()
})
```

::: tip
Zobacz więcej szczegółów na temat [używania MSW w przeglądarce](https://mswjs.io/docs/integrations/browser).
:::

### Testowanie komunikacji komponentów

```tsx
// Testuj interakcję komponentu rodzica z potomkiem
test('parent and child components communicate correctly', async () => {
  const mockOnSelectionChange = vi.fn()

  const { getByText } = render(
    <ProductCatalog onSelectionChange={mockOnSelectionChange}>
      <ProductFilter />
      <ProductGrid />
    </ProductCatalog>
  )

  // Interakcja z komponentem potomnym
  await page.getByRole('checkbox', { name: /electronics/i }).click()

  // Zweryfikuj, że rodzic otrzymuje komunikację
  expect(mockOnSelectionChange).toHaveBeenCalledWith({
    category: 'electronics',
    filters: ['electronics']
  })

  // Zweryfikuj aktualizacje innego komponentu potomnego (expect.element automatycznie ponawia próby)
  await expect.element(getByText('Showing Electronics products')).toBeInTheDocument()
})
```

### Testowanie złożonych formularzy z walidacją

```tsx
test('ContactForm handles complex validation scenarios', async () => {
  const mockSubmit = vi.fn()
  const { getByLabelText, getByText } = render(
    <ContactForm onSubmit={mockSubmit} />
  )

  const nameInput = page.getByLabelText(/full name/i)
  const emailInput = page.getByLabelText(/email/i)
  const messageInput = page.getByLabelText(/message/i)
  const submitButton = page.getByRole('button', { name: /send message/i })

  // Testuj wyzwalanie walidacji
  await submitButton.click()

  await expect.element(getByText('Name is required')).toBeInTheDocument()
  await expect.element(getByText('Email is required')).toBeInTheDocument()
  await expect.element(getByText('Message is required')).toBeInTheDocument()

  // Testuj częściową walidację
  await nameInput.fill('John Doe')
  await submitButton.click()

  await expect.element(getByText('Name is required')).not.toBeInTheDocument()
  await expect.element(getByText('Email is required')).toBeInTheDocument()

  // Testuj walidację formatu email
  await emailInput.fill('invalid-email')
  await submitButton.click()

  await expect.element(getByText('Please enter a valid email')).toBeInTheDocument()

  // Testuj pomyślne wysłanie
  await emailInput.fill('john@example.com')
  await messageInput.fill('Hello, this is a test message.')
  await submitButton.click()

  expect(mockSubmit).toHaveBeenCalledWith({
    name: 'John Doe',
    email: 'john@example.com',
    message: 'Hello, this is a test message.'
  })
})
```

### Testowanie Error Boundaries

```tsx
// Testuj, jak komponenty obsługują i odzyskują się z błędów
function ThrowError({ shouldThrow }: { shouldThrow: boolean }) {
  if (shouldThrow) {
    throw new Error('Component error!')
  }
  return <div>Component working fine</div>
}

test('ErrorBoundary catches and displays errors gracefully', async () => {
  const { getByText, rerender } = render(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <ThrowError shouldThrow={false} />
    </ErrorBoundary>
  )

  // Początkowo działa
  await expect.element(getByText('Component working fine')).toBeInTheDocument()

  // Wyzwól błąd
  rerender(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <ThrowError shouldThrow={true} />
    </ErrorBoundary>
  )

  // Error boundary powinno go przechwycić
  await expect.element(getByText('Something went wrong')).toBeInTheDocument()
})
```

### Testowanie dostępności

```tsx
test('Modal component is accessible', async () => {
  const { getByRole, getByLabelText } = render(
    <Modal isOpen={true} title="Settings">
      <SettingsForm />
    </Modal>
  )

  // Testuj zarządzanie fokusem - modal powinien otrzymać fokus po otwarciu
  // Jest to kluczowe dla użytkowników czytników ekranu, aby wiedzieli, że modal się otworzył
  const modal = getByRole('dialog')
  await expect.element(modal).toHaveFocus()

  // Testuj atrybuty ARIA - dostarczają informacji semantycznych dla czytników ekranu
  await expect.element(modal).toHaveAttribute('aria-labelledby') // Łączy z elementem tytułu
  await expect.element(modal).toHaveAttribute('aria-modal', 'true') // Wskazuje zachowanie modalu

  // Testuj nawigację klawiaturą - klawisz Escape powinien zamknąć modal
  // Jest to wymagane przez praktyki autorskie ARIA
  await userEvent.keyboard('{Escape}')
  // expect.element automatycznie ponawia próby, aż modal zostanie usunięty
  await expect.element(modal).not.toBeInTheDocument()

  // Testuj pułapkę fokusa - nawigacja tab powinna cyklicznie przechodzić w modalu
  // Zapobiega to przechodzeniu użytkowników do treści za modalem
  const firstInput = getByLabelText(/username/i)
  const lastButton = getByRole('button', { name: /save/i })

  // Użyj click, aby ustawić fokus na pierwszym inpucie, następnie testuj nawigację tab
  await firstInput.click()
  await userEvent.keyboard('{Shift>}{Tab}{/Shift}') // Shift+Tab idzie wstecz
  await expect.element(lastButton).toHaveFocus() // Powinno zawinąć do ostatniego elementu
})
```

## Debugowanie testów komponentów

### 1. Używaj narzędzi deweloperskich przeglądarki

Tryb Przeglądarki uruchamia testy w prawdziwych przeglądarkach, dając Ci dostęp do pełnych narzędzi deweloperskich. Gdy testy zawiodą, możesz:

- **Otworzyć narzędzia deweloperskie przeglądarki** podczas wykonywania testu (F12 lub prawy klik → Zbadaj)
- **Ustawić punkty przerwania** w kodzie testu lub komponentu
- **Zbadać DOM** aby zobaczyć faktyczne wyrenderowane wyjście
- **Sprawdzić błędy konsoli** dla błędów lub ostrzeżeń JavaScript
- **Monitorować żądania sieciowe** aby debugować wywołania API

Do debugowania w trybie headful, tymczasowo dodaj `headless: false` do konfiguracji przeglądarki.

### 2. Dodaj instrukcje debugowania

Użyj strategicznego logowania, aby zrozumieć niepowodzenia testów:

```tsx
test('debug form validation', async () => {
  render(<ContactForm />)

  const submitButton = page.getByRole('button', { name: /submit/i })
  await submitButton.click()

  // Debug: Sprawdź, czy element istnieje z innym zapytaniem
  const errorElement = page.getByText('Email is required')
  console.log('Error element found:', errorElement.length)

  await expect.element(errorElement).toBeInTheDocument()
})
```

### 3. Zbadaj wyrenderowane wyjście

Gdy komponenty nie renderują się zgodnie z oczekiwaniami, badaj systematycznie:

**Użyj UI przeglądarki Vitest:**
- Uruchom testy z włączonym trybem przeglądarki
- Otwórz URL przeglądarki wyświetlony w terminalu, aby zobaczyć uruchomione testy
- Wizualna inspekcja pomaga zidentyfikować problemy z CSS, układem lub brakujące elementy

**Testuj zapytania elementów:**
```tsx
// Debuguj, dlaczego elementów nie można znaleźć
const button = page.getByRole('button', { name: /submit/i })
console.log('Button count:', button.length) // Powinno być 1

// Spróbuj alternatywnych zapytań, jeśli pierwsze zawiedzie
if (button.length === 0) {
  console.log('All buttons:', page.getByRole('button').length)
  console.log('By test ID:', page.getByTestId('submit-btn').length)
}
```

### 4. Zweryfikuj selektory

Problemy z selektorami są częstymi przyczynami niepowodzeń testów. Debuguj je systematycznie:

**Sprawdź dostępne nazwy:**
```tsx
// Jeśli getByRole zawiedzie, sprawdź jakie role/nazwy są dostępne
const buttons = page.getByRole('button').all()
for (const button of buttons) {
  // Użyj element() aby uzyskać element DOM i uzyskać dostęp do natywnych właściwości
  const element = button.element()
  const accessibleName = element.getAttribute('aria-label') || element.textContent
  console.log(`Button: "${accessibleName}"`)
}
```

**Testuj różne strategie zapytań:**
```tsx
// Wiele sposobów znalezienia tego samego elementu używając .or dla automatycznego ponawiania
const submitButton = page.getByRole('button', { name: /submit/i }) // Przez dostępną nazwę
  .or(page.getByTestId('submit-button')) // Przez test ID
  .or(page.getByText('Submit')) // Przez dokładny tekst
// Uwaga: Vitest nie ma page.locator(), użyj konkretnych metod getBy* zamiast tego
```

**Popularne wzorce debugowania selektorów:**
```tsx
test('debug element queries', async () => {
  render(<LoginForm />)

  // Sprawdź, czy element jest widoczny i włączony
  const emailInput = page.getByLabelText(/email/i)
  await expect.element(emailInput).toBeVisible() // Pokaże, czy element jest widoczny i wydrukuje DOM, jeśli nie
})
```

### 5. Debugowanie problemów asynchronicznych

Testy komponentów często wiążą się z problemami z czasem:

```tsx
test('debug async component behavior', async () => {
  render(<AsyncUserProfile userId="123" />)

  // expect.element automatycznie ponowi próby i pokaże pomocne komunikaty o błędach
  await expect.element(page.getByText('John Doe')).toBeInTheDocument()
})
```

## Migracja z innych frameworków testowych

### Z Jest + Testing Library

Większość testów Jest + Testing Library działa z minimalnymi zmianami:

```ts
// Przed (Jest)
import { render, screen } from '@testing-library/react' // [!code --]

// Po (Vitest)
import { render } from 'vitest-browser-react' // [!code ++]
```

### Kluczowe różnice

- Używaj `await expect.element()` zamiast `expect()` dla asercji DOM
- Używaj `vitest/browser` dla interakcji użytkownika zamiast `@testing-library/user-event`
- Tryb Przeglądarki zapewnia prawdziwe środowisko przeglądarki dla dokładnego testowania

## Dowiedz się więcej

- [Dokumentacja Trybu Przeglądarki](/guide/browser/)
- [API Asercji](/api/browser/assertions)
- [API Interaktywności](/api/browser/interactivity)
- [Repozytorium z przykładami](https://github.com/vitest-tests/browser-examples)
