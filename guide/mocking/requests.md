# Mockowanie żądań

Ponieważ Vitest działa w Node, mockowanie żądań sieciowych jest trudne; API webowe nie są dostępne, więc potrzebujemy czegoś, co będzie naśladować zachowanie sieci za nas. Zalecamy [Mock Service Worker](https://mswjs.io/), aby to osiągnąć. Pozwala mockować żądania sieciowe `http`, `WebSocket` i `GraphQL`, i jest niezależny od frameworka.

Mock Service Worker (MSW) działa poprzez przechwytywanie żądań wykonywanych przez testy, pozwalając używać go bez zmieniania kodu aplikacji. W przeglądarce wykorzystuje [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API). W Node.js i dla Vitest używa biblioteki [`@mswjs/interceptors`](https://github.com/mswjs/interceptors). Aby dowiedzieć się więcej o MSW, przeczytaj ich [wprowadzenie](https://mswjs.io/docs/)

## Konfiguracja

Możesz użyć go jak poniżej w swoim [pliku setup](/config/setupfiles)

::: code-group

```js [Konfiguracja HTTP]
import { afterAll, afterEach, beforeAll } from 'vitest'
import { setupServer } from 'msw/node'
import { http, HttpResponse } from 'msw'

const posts = [
  {
    userId: 1,
    id: 1,
    title: 'tytuł pierwszego posta',
    body: 'treść pierwszego posta',
  },
  // ...
]

export const restHandlers = [
  http.get('https://rest-endpoint.example/path/to/posts', () => {
    return HttpResponse.json(posts)
  }),
]

const server = setupServer(...restHandlers)

// Uruchom serwer przed wszystkimi testami
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))

// Zamknij serwer po wszystkich testach
afterAll(() => server.close())

// Zresetuj handlery po każdym teście dla izolacji testów
afterEach(() => server.resetHandlers())
```

```js [Konfiguracja GraphQL]
import { afterAll, afterEach, beforeAll } from 'vitest'
import { setupServer } from 'msw/node'
import { graphql, HttpResponse } from 'msw'

const posts = [
  {
    userId: 1,
    id: 1,
    title: 'tytuł pierwszego posta',
    body: 'treść pierwszego posta',
  },
  // ...
]

const graphqlHandlers = [
  graphql.query('ListPosts', () => {
    return HttpResponse.json({
      data: { posts },
    })
  }),
]

const server = setupServer(...graphqlHandlers)

// Uruchom serwer przed wszystkimi testami
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))

// Zamknij serwer po wszystkich testach
afterAll(() => server.close())

// Zresetuj handlery po każdym teście dla izolacji testów
afterEach(() => server.resetHandlers())
```

```js [Konfiguracja WebSocket]
import { afterAll, afterEach, beforeAll } from 'vitest'
import { setupServer } from 'msw/node'
import { ws } from 'msw'

const chat = ws.link('wss://chat.example.com')

const wsHandlers = [
  chat.addEventListener('connection', ({ client }) => {
    client.addEventListener('message', (event) => {
      console.log('Otrzymano wiadomość od klienta:', event.data)
      // Odesłanie otrzymanej wiadomości z powrotem do klienta
      client.send(`Serwer otrzymał: ${event.data}`)
    })
  }),
]

const server = setupServer(...wsHandlers)

// Uruchom serwer przed wszystkimi testami
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))

// Zamknij serwer po wszystkich testach
afterAll(() => server.close())

// Zresetuj handlery po każdym teście dla izolacji testów
afterEach(() => server.resetHandlers())
```
:::

> Konfiguracja serwera z `onUnhandledRequest: 'error'` zapewnia, że błąd zostanie wyrzucony za każdym razem, gdy pojawi się żądanie, które nie ma odpowiadającego mu handlera żądania.

## Więcej
MSW oferuje znacznie więcej. Możesz uzyskać dostęp do ciasteczek i parametrów zapytania, definiować mockowane odpowiedzi błędów i wiele więcej! Aby zobaczyć wszystko, co możesz zrobić z MSW, przeczytaj [ich dokumentację](https://mswjs.io/docs).
