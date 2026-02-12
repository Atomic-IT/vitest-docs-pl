---
title: Adnotacje testów | Przewodnik
outline: deep
---

# Adnotacje testów

Vitest wspiera adnotowanie testów niestandardowymi wiadomościami i plikami przez API [`context.annotate`](/guide/test-context#annotate). Te adnotacje będą dołączone do przypadku testowego i przekazane do reporterów w hooku [`onTestAnnotate`](/api/advanced/reporters#ontestannotate).

```ts
test('hello world', async ({ annotate }) => {
  await annotate('to jest mój test')

  if (condition) {
    await annotate('to powinno było zgłosić błąd', 'error')
  }

  const file = createTestSpecificFile()
  await annotate('tworzy plik', { body: file })
})
```

::: warning
Funkcja `annotate` zwraca Promise, więc musi być oczekiwana, jeśli w jakiś sposób na niej polegasz. Jednak Vitest automatycznie oczekuje również na każdą nieoczekiwaną adnotację przed zakończeniem testu.
:::

W zależności od reportera adnotacje będą wyświetlane inaczej.

## Wbudowane reportery
### default

Reporter `default` drukuje adnotacje tylko jeśli test nie powiódł się:

```
  ⎯⎯⎯⎯⎯⎯⎯ Failed Tests 1 ⎯⎯⎯⎯⎯⎯⎯

  FAIL  example.test.js > przykład testu z adnotacją
Error: wyrzucony błąd
  ❯ example.test.js:11:21
      9 |    await annotate('adnotacja 1')
      10|    await annotate('adnotacja 2', 'warning')
      11|    throw new Error('wyrzucony błąd')
        |          ^
      12|  })

  ❯ example.test.js:9:15 notice
    ↳ adnotacja 1
  ❯ example.test.js:10:15 warning
    ↳ adnotacja 2

  ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯[1/1]⎯
```

### verbose

Reporter `verbose` jest jedynym reporterem terminalowym, który raportuje adnotacje, gdy test nie kończy się niepowodzeniem.

```
✓ example.test.js > przykład testu z adnotacją

  ❯ example.test.js:9:15 notice
    ↳ adnotacja 1
  ❯ example.test.js:10:15 warning
    ↳ adnotacja 2

```

### html

Reporter HTML pokazuje adnotacje w ten sam sposób co UI. Możesz zobaczyć adnotację w linii, gdzie została wywołana. W tej chwili, jeśli adnotacja nie została wywołana w pliku testowym, nie możesz jej zobaczyć w UI. Planujemy wsparcie oddzielnego widoku podsumowania testu, gdzie będzie widoczna.

<img alt="Vitest UI" img-light src="/annotations-html-light.png">
<img alt="Vitest UI" img-dark src="/annotations-html-dark.png">

### junit

Reporter `junit` wyświetla adnotacje wewnątrz tagu `properties` przypadku testowego. Reporter JUnit zignoruje wszystkie załączniki i wydrukuje tylko typ i wiadomość.

```xml
<testcase classname="basic/example.test.js" name="przykład testu z adnotacją" time="0.14315">
    <properties>
        <property name="notice" value="wiadomość adnotacji">
        </property>
    </properties>
</testcase>
```

### github-actions

Reporter `github-actions` domyślnie wydrukuje adnotację jako [wiadomość notice](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions#setting-a-notice-message). Możesz skonfigurować `type`, przekazując drugi argument jako `notice`, `warning` lub `error`. Jeśli typ nie jest żadnym z tych, Vitest pokaże wiadomość jako notice.

<img alt="GitHub Actions" img-light src="/annotations-gha-light.png">
<img alt="GitHub Actions" img-dark src="/annotations-gha-dark.png">

### tap

Reportery `tap` i `tap-flat` drukują adnotacje jako wiadomości diagnostyczne w nowej linii zaczynającej się od symbolu `#`. Zignorują wszystkie załączniki i wydrukują tylko typ i wiadomość:

```
ok 1 - przykład testu z adnotacją # time=143.15ms
    # notice: wiadomość adnotacji
```
