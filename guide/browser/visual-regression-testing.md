---
title: Testowanie regresji wizualnej
outline: [2, 3]
---

# Testowanie regresji wizualnej

Vitest może uruchamiać testy regresji wizualnej od razu. Przechwytuje zrzuty ekranu
Twoich komponentów UI i stron, a następnie porównuje je z obrazami referencyjnymi,
aby wykryć niezamierzone zmiany wizualne.

W przeciwieństwie do testów funkcjonalnych, które weryfikują zachowanie, testy wizualne
wychwytują problemy ze stylami, przesunięcia układu i problemy z renderowaniem,
które w przeciwnym razie mogłyby pozostać niezauważone bez dokładnego testowania ręcznego.

## Dlaczego testowanie regresji wizualnej?

Błędy wizualne nie rzucają wyjątków, po prostu źle wyglądają. Tu wkracza
testowanie wizualne.

- Ten przycisk nadal wysyła formularz... ale dlaczego jest teraz różowy?
- Tekst idealnie się mieści... dopóki ktoś nie otworzy go na telefonie
- Wszystko działa świetnie... z wyjątkiem tych dwóch kontenerów poza viewport
- Ten staranny refaktor CSS działa... ale zepsuł układ na stronie, której nikt nie testuje

Testowanie regresji wizualnej działa jako siatka bezpieczeństwa dla Twojego UI,
automatycznie wychwytując te zmiany wizualne zanim trafią na produkcję.

## Rozpoczęcie pracy

::: warning Różnice w renderowaniu przeglądarek
Testy regresji wizualnej są **z natury niestabilne w różnych
środowiskach**. Zrzuty ekranu będą wyglądać inaczej na różnych maszynach z powodu:

- Renderowania czcionek (to główny problem. Windows, macOS, Linux, wszystkie renderują tekst
inaczej)
- Sterowników GPU i akceleracji sprzętowej
- Tego, czy uruchamiasz w trybie headless czy nie
- Ustawień i wersji przeglądarki
- ...i szczerze mówiąc, czasem po prostu fazy księżyca

Dlatego Vitest uwzględnia przeglądarkę i platformę w nazwach zrzutów ekranu (jak
`button-chromium-darwin.png`).

Dla stabilnych testów używaj tego samego środowiska wszędzie. **Zdecydowanie zalecamy**
usługi chmurowe takie jak
[Azure App Testing](https://azure.microsoft.com/en-us/products/app-testing/)
lub [kontenery Docker](https://playwright.dev/docs/docker).
:::

Testowanie regresji wizualnej w Vitest można wykonać poprzez
[asercję `toMatchScreenshot`](/api/browser/assertions.html#tomatchscreenshot):

```ts
import { expect, test } from 'vitest'
import { page } from 'vitest/browser'

test('hero section looks correct', async () => {
  // ...reszta testu

  // przechwytuj i porównuj zrzut ekranu
  await expect(page.getByTestId('hero')).toMatchScreenshot('hero-section')
})
```

### Tworzenie referencji

Gdy uruchamiasz test wizualny po raz pierwszy, Vitest tworzy referencyjny (zwany również
bazowym) zrzut ekranu i kończy test niepowodzeniem z następującym komunikatem błędu:

```
expect(element).toMatchScreenshot()

No existing reference screenshot found; a new one was created. Review it before running tests again.

Reference screenshot:
  tests/__screenshots__/hero.test.ts/hero-section-chromium-darwin.png
```

To normalne. Sprawdź, czy zrzut ekranu wygląda dobrze, a następnie uruchom test ponownie.
Vitest będzie teraz porównywał przyszłe uruchomienia z tą bazą.

::: tip
Referencyjne zrzuty ekranu znajdują się w folderach `__screenshots__` obok Twoich testów.
**Nie zapomnij je zacommitować!**
:::

### Organizacja zrzutów ekranu

Domyślnie zrzuty ekranu są zorganizowane jako:

```
.
├── __screenshots__
│   └── test-file.test.ts
│       ├── test-name-chromium-darwin.png
│       ├── test-name-firefox-linux.png
│       └── test-name-webkit-win32.png
└── test-file.test.ts
```

Konwencja nazewnictwa obejmuje:
- **Nazwa testu**: albo pierwszy argument wywołania `toMatchScreenshot()`,
albo automatycznie wygenerowana z nazwy testu.
- **Nazwa przeglądarki**: `chrome`, `chromium`, `firefox` lub `webkit`.
- **Platforma**: `aix`, `darwin`, `freebsd`, `linux`, `openbsd`, `sunos` lub
`win32`.

Zapewnia to, że zrzuty ekranu z różnych środowisk nie nadpisują się nawzajem.

### Aktualizowanie referencji

Gdy celowo zmieniasz swój UI, będziesz musiał zaktualizować referencyjne
zrzuty ekranu:

```bash
$ vitest --update
```

Przejrzyj zaktualizowane zrzuty ekranu przed zacommitowaniem, aby upewnić się, że zmiany są
zamierzone.

## Jak działają testy wizualne

Testy regresji wizualnej potrzebują stabilnych zrzutów ekranu do porównania. Ale strony nie są natychmiast stabilne, gdy obrazy się ładują, animacje kończą, czcionki renderują i układy się ustalają.

Vitest obsługuje to automatycznie poprzez "Wykrywanie Stabilnego Zrzutu Ekranu":

1. Vitest robi pierwszy zrzut ekranu (lub używa referencyjnego zrzutu ekranu, jeśli jest dostępny) jako bazę
1. Robi kolejny zrzut ekranu i porównuje go z bazą
    - Jeśli zrzuty ekranu się zgadzają, strona jest stabilna i testowanie kontynuuje
    - Jeśli się różnią, Vitest używa najnowszego zrzutu ekranu jako bazy i powtarza
1. To kontynuuje, aż stabilność zostanie osiągnięta lub timeout zostanie przekroczony

Zapewnia to, że przejściowe zmiany wizualne (jak wskaźniki ładowania lub animacje) nie powodują fałszywych niepowodzeń. Jeśli jednak coś nigdy nie przestaje się animować, przekroczysz timeout, więc rozważ [wyłączenie animacji podczas testowania](#disable-animations).

Jeśli stabilny zrzut ekranu zostanie przechwycony po ponownych próbach (jednej lub więcej) i istnieje referencyjny zrzut ekranu, Vitest wykonuje końcowe porównanie z referencją używając `createDiff: true`. Wygeneruje to obraz różnic, jeśli się nie zgadzają.

Podczas wykrywania stabilności Vitest wywołuje komparatory z `createDiff: false`, ponieważ musi tylko wiedzieć, czy zrzuty ekranu się zgadzają. Utrzymuje to szybki proces wykrywania.

## Konfigurowanie testów wizualnych

### Konfiguracja globalna

Skonfiguruj domyślne ustawienia testowania regresji wizualnej w swojej
[konfiguracji Vitest](/config/browser/expect#tomatchscreenshot):

```ts [vitest.config.ts]
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    browser: {
      expect: {
        toMatchScreenshot: {
          comparatorName: 'pixelmatch',
          comparatorOptions: {
            // 0-1, jak bardzo różne mogą być kolory?
            threshold: 0.2,
            // 1% pikseli może się różnić
            allowedMismatchedPixelRatio: 0.01,
          },
        },
      },
    },
  },
})
```

### Konfiguracja per-test

Nadpisz globalne ustawienia dla konkretnych testów:

```ts
await expect(element).toMatchScreenshot('button-hover', {
  comparatorName: 'pixelmatch',
  comparatorOptions: {
    // bardziej tolerancyjne porównanie dla elementów z dużą ilością tekstu
    allowedMismatchedPixelRatio: 0.1,
  },
})
```

## Najlepsze praktyki

### Testuj konkretne elementy

Chyba że wyraźnie chcesz testować całą stronę, preferuj przechwytywanie konkretnych
komponentów, aby zmniejszyć fałszywe alarmy:

```ts
// ❌ Przechwytuje całą stronę; podatne na niezwiązane zmiany
await expect(page).toMatchScreenshot()

// ✅ Przechwytuje tylko testowany komponent
await expect(page.getByTestId('product-card')).toMatchScreenshot()
```

### Obsługuj dynamiczną zawartość

Dynamiczna zawartość jak znaczniki czasu, dane użytkownika lub losowe wartości spowodują
niepowodzenie testów. Możesz albo mockować źródła dynamicznej zawartości, albo je maskować
używając dostawcy Playwright poprzez
[opcję `mask`](https://playwright.dev/docs/api/class-page#page-screenshot-option-mask)
w `screenshotOptions`.

```ts
await expect(page.getByTestId('profile')).toMatchScreenshot({
  screenshotOptions: {
    mask: [page.getByTestId('last-seen')],
  },
})
```

### Wyłącz animacje

Animacje mogą powodować niestabilne testy. Wyłącz je podczas testowania, wstrzykując
niestandardowy snippet CSS:

```css
*, *::before, *::after {
  animation-duration: 0s !important;
  animation-delay: 0s !important;
  transition-duration: 0s !important;
  transition-delay: 0s !important;
}
```

::: tip
Używając dostawcy Playwright, animacje są automatycznie wyłączane
podczas używania asercji: wartość opcji `animations` w `screenshotOptions`
jest domyślnie ustawiona na `"disabled"`.
:::

### Ustaw odpowiednie progi

Dostrajanie progów jest trudne. Zależy od zawartości, środowiska testowego,
tego co jest akceptowalne dla Twojej aplikacji i może się również zmieniać w zależności od testu.

Vitest nie ustawia domyślnej wartości dla niedopasowanych pikseli, to użytkownik
decyduje na podstawie swoich potrzeb. Zaleceniem jest użycie
`allowedMismatchedPixelRatio`, aby próg był obliczany na podstawie rozmiaru
zrzutu ekranu, a nie stałej liczby.

Ustawiając zarówno `allowedMismatchedPixelRatio`, jak i
`allowedMismatchedPixels`, Vitest używa tego limitu, który jest bardziej restrykcyjny.

### Ustaw spójne rozmiary viewport

Ponieważ instancja przeglądarki może mieć inny domyślny rozmiar, najlepiej jest
ustawić konkretny rozmiar viewport, albo w teście, albo w konfiguracji
instancji:

```ts
await page.viewport(1280, 720)
```

```ts [vitest.config.ts]
import { playwright } from '@vitest/browser-playwright'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    browser: {
      enabled: true,
      provider: playwright(),
      instances: [
        {
          browser: 'chromium',
          viewport: { width: 1280, height: 720 },
        },
      ],
    },
  },
})
```

### Używaj Git LFS

Przechowuj referencyjne zrzuty ekranu w
[Git LFS](https://github.com/git-lfs/git-lfs?tab=readme-ov-file), jeśli planujesz
mieć duży zestaw testów.

## Debugowanie nieudanych testów

Gdy test wizualny zawiedzie, Vitest dostarcza trzy obrazy, aby pomóc w debugowaniu:

1. **Referencyjny zrzut ekranu**: oczekiwany bazowy obraz
1. **Aktualny zrzut ekranu**: co zostało przechwycone podczas testu
1. **Obraz różnic**: podkreśla różnice, ale może nie zostać wygenerowany

Zobaczysz coś takiego:

```
expect(element).toMatchScreenshot()

Screenshot does not match the stored reference.
245 pixels (ratio 0.03) differ.

Reference screenshot:
  tests/__screenshots__/button.test.ts/button-chromium-darwin.png

Actual screenshot:
  tests/.vitest-attachments/button.test.ts/button-chromium-darwin-actual.png

Diff image:
  tests/.vitest-attachments/button.test.ts/button-chromium-darwin-diff.png
```

### Zrozumienie obrazu różnic

- **Czerwone piksele** to obszary, które różnią się między referencją a aktualnym
- **Żółte piksele** to różnice w anti-aliasingu (gdy anti-alias nie jest ignorowany)
- **Przezroczyste/oryginalne** to niezmienione obszary

:::tip
Jeśli różnica jest głównie czerwona, coś jest naprawdę nie tak. Jeśli ma rozrzucone
kilka czerwonych pikseli wokół tekstu, prawdopodobnie musisz tylko zwiększyć próg.
:::

## Częste problemy i rozwiązania

### Fałszywe alarmy z renderowania czcionek

Dostępność i renderowanie czcionek znacznie różni się między systemami. Niektóre
możliwe rozwiązania to:

- Używaj web fontów i czekaj na ich załadowanie:

  ```ts
  // czekaj na załadowanie czcionek
  await document.fonts.ready

  // kontynuuj testy
  ```

- Zwiększ próg porównania dla obszarów z dużą ilością tekstu:

  ```ts
  await expect(page.getByTestId('article-summary')).toMatchScreenshot({
    comparatorName: 'pixelmatch',
    comparatorOptions: {
      // 10% pikseli może się zmienić
      allowedMismatchedPixelRatio: 0.1,
    },
  })
  ```

- Używaj usługi chmurowej lub skonteneryzowanego środowiska dla spójnego renderowania czcionek.

### Niestabilne testy lub różne rozmiary zrzutów ekranu

Jeśli testy przechodzą i nie przechodzą losowo, lub jeśli zrzuty ekranu mają różne wymiary
między uruchomieniami:

- Czekaj na załadowanie wszystkiego, włącznie ze wskaźnikami ładowania
- Ustaw jawne rozmiary viewport: `await page.viewport(1920, 1080)`
- Sprawdź zachowanie responsywne na granicach viewport
- Sprawdź niezamierzone animacje lub przejścia
- Zwiększ timeout testu dla dużych zrzutów ekranu
- Używaj usługi chmurowej lub skonteneryzowanego środowiska

## Testowanie regresji wizualnej dla zespołów

Pamiętasz, gdy wspomnieliśmy, że testy wizualne potrzebują stabilnego środowiska? No cóż, oto
problem: Twoja lokalna maszyna takim nie jest.

Dla zespołów masz w zasadzie trzy opcje:

1. **Self-hosted runners**, skomplikowane w konfiguracji, bolesne w utrzymaniu
1. **GitHub Actions**, darmowe (dla open source), działa z dowolnym dostawcą
1. **Usługi chmurowe**, jak
[Azure App Testing](https://azure.microsoft.com/en-us/products/app-testing/),
stworzone dokładnie do tego problemu

Skupimy się na opcjach 2 i 3, ponieważ najszybciej je uruchomisz.

Aby być szczerym, główne kompromisy dla każdej z nich to:

- **GitHub Actions**: testy wizualne działają tylko w CI (deweloperzy nie mogą ich uruchamiać
lokalnie)
- **Usługa Microsoft**: działa wszędzie, ale kosztuje i działa tylko
z Playwright

:::: tabs key:vrt-for-teams
=== GitHub Actions

Sztuczka polega na trzymaniu testów wizualnych oddzielnie od zwykłych testów,
w przeciwnym razie zmarnujesz godziny sprawdzając niepowodzenia logów z niedopasowaniami zrzutów ekranu.

#### Organizowanie testów

Najpierw wyizoluj swoje testy wizualne. Umieść je w folderze `visual` (lub cokolwiek
ma sens dla Twojego projektu):

```json [package.json]
{
  "scripts": {
    "test:unit": "vitest --exclude tests/visual/*.test.ts",
    "test:visual": "vitest tests/visual/*.test.ts"
  }
}
```

Teraz deweloperzy mogą uruchamiać `npm run test:unit` lokalnie bez testów wizualnych
przeszkadzających. Testy wizualne pozostają w CI, gdzie środowisko jest spójne.

::: tip Alternatywa
Nie lubisz wzorców glob? Możesz również użyć oddzielnych
[Projektów testowych](/guide/projects) i uruchamiać je używając:

- `vitest --project unit`
- `vitest --project visual`
:::

#### Konfiguracja CI

Twoje CI potrzebuje zainstalowanych przeglądarek. Sposób, w jaki to robisz, zależy od dostawcy:

::: tabs key:provider
== Playwright

[Playwright](https://npmjs.com/package/playwright) ułatwia to. Po prostu przypnij
swoją wersję i dodaj to przed uruchomieniem testów:

```yaml [.github/workflows/ci.yml]
# ...reszta workflow
- name: Install Playwright Browsers
  run: npx --no playwright install --with-deps --only-shell
```

== WebdriverIO

[WebdriverIO](https://www.npmjs.com/package/webdriverio) oczekuje, że sam dostarczysz
przeglądarki. Ludzie z
[@browser-actions](https://github.com/browser-actions) Ci pomogą:

```yaml [.github/workflows/ci.yml]
# ...reszta workflow
- uses: browser-actions/setup-chrome@v1
  with:
    chrome-version: 120
```

:::

Następnie uruchom swoje testy wizualne:

```yaml [.github/workflows/ci.yml]
# ...reszta workflow
# ...konfiguracja przeglądarki
- name: Visual Regression Testing
  run: npm run test:visual
```

#### Workflow aktualizacji

Tutaj robi się ciekawie. Nie chcesz automatycznie aktualizować zrzutów ekranu przy każdym
PR <small>*(chaos!)*</small>. Zamiast tego stwórz
ręcznie wyzwalany workflow, który deweloperzy mogą uruchomić, gdy celowo
zmieniają UI.

Poniższy workflow:
- Uruchamia się tylko na gałęziach feature (nigdy na main)
- Przypisuje osobę, która go wyzwoliła, jako współautora
- Zapobiega równoczesnym uruchomieniom na tej samej gałęzi
- Pokazuje ładne podsumowanie:
  - **Gdy zrzuty ekranu się zmieniły**, wyświetla co się zmieniło

    <img alt="Podsumowanie akcji po aktualizacjach" img-light src="/vrt-gha-summary-update-light.png">
    <img alt="Podsumowanie akcji po aktualizacjach" img-dark src="/vrt-gha-summary-update-dark.png">

  - **Gdy nic się nie zmieniło**, no cóż, też Ci to powie

    <img alt="Podsumowanie akcji bez aktualizacji" img-light src="/vrt-gha-summary-no-update-light.png">
    <img alt="Podsumowanie akcji bez aktualizacji" img-dark src="/vrt-gha-summary-no-update-dark.png">

::: tip
To tylko jedno podejście. Niektóre zespoły preferują komentarze PR (`/update-screenshots`),
inne używają etykiet. Dostosuj to do swojego workflow!

Ważną częścią jest posiadanie kontrolowanego sposobu aktualizowania baz.
:::

```yaml [.github/workflows/update-screenshots.yml]
name: Update Visual Regression Screenshots

on:
  workflow_dispatch: # manual trigger only

env:
  AUTHOR_NAME: 'github-actions[bot]'
  AUTHOR_EMAIL: '41898282+github-actions[bot]@users.noreply.github.com'
  COMMIT_MESSAGE: |
    test: update visual regression screenshots

    Co-authored-by: ${{ github.actor }} <${{ github.actor_id }}+${{ github.actor }}@users.noreply.github.com>

jobs:
  update-screenshots:
    runs-on: ubuntu-24.04

    # safety first: don't run on main
    if: github.ref_name != github.event.repository.default_branch

    # one at a time per branch
    concurrency:
      group: visual-regression-screenshots@${{ github.ref_name }}
      cancel-in-progress: true

    permissions:
      contents: write # needs to push changes

    steps:
      - name: Checkout selected branch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.ref_name }}
          # use PAT if triggering other workflows
          # token: ${{ secrets.GITHUB_TOKEN }}

      - name: Configure Git
        run: |
          git config --global user.name "${{ env.AUTHOR_NAME }}"
          git config --global user.email "${{ env.AUTHOR_EMAIL }}"

      # your setup steps here (node, pnpm, whatever)
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 24

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx --no playwright install --with-deps --only-shell

      # the magic happens below 🪄
      - name: Update Visual Regression Screenshots
        run: npm run test:visual --update

      # check what changed
      - name: Check for changes
        id: check_changes
        run: |
          CHANGED_FILES=$(git status --porcelain | awk '{print $2}')
          if [ "${CHANGED_FILES:+x}" ]; then
            echo "changes=true" >> $GITHUB_OUTPUT
            echo "Changes detected"

            # save the list for the summary
            echo "changed_files<<EOF" >> $GITHUB_OUTPUT
            echo "$CHANGED_FILES" >> $GITHUB_OUTPUT
            echo "EOF" >> $GITHUB_OUTPUT
            echo "changed_count=$(echo "$CHANGED_FILES" | wc -l)" >> $GITHUB_OUTPUT
          else
            echo "changes=false" >> $GITHUB_OUTPUT
            echo "No changes detected"
          fi

      # commit if there are changes
      - name: Commit changes
        if: steps.check_changes.outputs.changes == 'true'
        run: |
          git add -A
          git commit -m "${{ env.COMMIT_MESSAGE }}"

      - name: Push changes
        if: steps.check_changes.outputs.changes == 'true'
        run: git push origin ${{ github.ref_name }}

      # pretty summary for humans
      - name: Summary
        run: |
          if [[ "${{ steps.check_changes.outputs.changes }}" == "true" ]]; then
            echo "### 📸 Visual Regression Screenshots Updated" >> $GITHUB_STEP_SUMMARY
            echo "" >> $GITHUB_STEP_SUMMARY
            echo "Successfully updated **${{ steps.check_changes.outputs.changed_count }}** screenshot(s) on \`${{ github.ref_name }}\`" >> $GITHUB_STEP_SUMMARY
            echo "" >> $GITHUB_STEP_SUMMARY
            echo "#### Changed Files:" >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
            echo "${{ steps.check_changes.outputs.changed_files }}" >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
            echo "" >> $GITHUB_STEP_SUMMARY
            echo "✅ The updated screenshots have been committed and pushed. Your visual regression baseline is now up to date!" >> $GITHUB_STEP_SUMMARY
          else
            echo "### ℹ️ No Screenshot Updates Required" >> $GITHUB_STEP_SUMMARY
            echo "" >> $GITHUB_STEP_SUMMARY
            echo "The visual regression test command ran successfully but no screenshots needed updating." >> $GITHUB_STEP_SUMMARY
            echo "" >> $GITHUB_STEP_SUMMARY
            echo "All screenshots are already up to date! 🎉" >> $GITHUB_STEP_SUMMARY
          fi
```

=== Azure App Testing

Twoje testy pozostają lokalne, tylko przeglądarki działają w chmurze. To funkcja
zdalnej przeglądarki Playwright, ale Microsoft zarządza całą infrastrukturą.

#### Organizowanie testów

Trzymaj testy wizualne oddzielnie, aby kontrolować koszty. Tylko testy, które faktycznie robią
zrzuty ekranu, powinny używać usługi.

Najczystszym podejściem jest używanie [Projektów testowych](/guide/projects):

<!-- eslint-disable style/quote-props -->
```ts [vitest.config.ts]
import { env } from 'node:process'
import { defineConfig } from 'vitest/config'
import { playwright } from '@vitest/browser-playwright'

export default defineConfig({
  // ...global Vite config
  tests: {
    // ...global Vitest config
    projects: [
      {
        extends: true,
        test: {
          name: 'unit',
          include: ['tests/**/*.test.ts'],
          // regular config, can use local browsers
        },
      },
      {
        extends: true,
        test: {
          name: 'visual',
          // or you could use a different suffix, e.g.,: `tests/**/*.visual.ts?(x)`
          include: ['visual-regression-tests/**/*.test.ts?(x)'],
          browser: {
            enabled: true,
            provider: playwright({
              connectOptions: {
                wsEndpoint: `${env.PLAYWRIGHT_SERVICE_URL}?${new URLSearchParams({
                  'api-version': '2025-09-01',
                  os: 'linux', // always use Linux for consistency
                  // helps identifying runs in the service's dashboard
                  runName: `Vitest ${env.CI ? 'CI' : 'local'} run @${new Date().toISOString()}`,
                })}`,
                exposeNetwork: '<loopback>',
                headers: {
                  Authorization: `Bearer ${env.PLAYWRIGHT_SERVICE_ACCESS_TOKEN}`,
                },
                timeout: 30_000,
              }
            }),
            headless: true,
            instances: [
              {
                browser: 'chromium',
                viewport: { width: 2560, height: 1440 },
              },
            ],
          },
        },
      },
    ],
  },
})
```
<!-- eslint-enable style/quote-props -->

Postępuj zgodnie z [oficjalnym przewodnikiem tworzenia Playwright Workspace](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/quickstart-run-end-to-end-tests?tabs=playwrightcli&pivots=playwright-test-runner#create-a-workspace).

Gdy Twój workspace zostanie utworzony, skonfiguruj Vitest, aby go używał:

1. **Ustaw URL endpointu**: postępując zgodnie z [oficjalnym przewodnikiem](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/quickstart-run-end-to-end-tests?tabs=playwrightcli&pivots=playwright-test-runner#configure-the-browser-endpoint), pobierz URL i ustaw go jako zmienną środowiskową `PLAYWRIGHT_SERVICE_URL`.
1. **Włącz uwierzytelnianie tokenem**: [włącz tokeny dostępu](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/how-to-manage-authentication?pivots=playwright-test-runner#enable-authentication-using-access-tokens) dla swojego workspace, następnie [wygeneruj token](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/how-to-manage-access-tokens#generate-a-workspace-access-token) i ustaw go jako zmienną środowiskową `PLAYWRIGHT_SERVICE_ACCESS_TOKEN`.

::: danger Trzymaj ten token w sekrecie!
Nigdy nie commituj `PLAYWRIGHT_SERVICE_ACCESS_TOKEN` do swojego repozytorium. Każdy z
tokenem może nabijać Twój rachunek. Używaj zmiennych środowiskowych lokalnie i sekretów
w CI.
:::

Następnie podziel swój skrypt `test` w ten sposób:

```json [package.json]
{
  "scripts": {
    "test:visual": "vitest --project visual",
    "test:unit": "vitest --project unit"
  }
}
```

#### Uruchamianie testów

```bash
# Lokalny development
npm run test:unit    # darmowe, uruchamiane lokalnie
npm run test:visual  # używa przeglądarek w chmurze

# Aktualizuj zrzuty ekranu
npm run test:visual -- --update
```

Najlepszą częścią tego podejścia jest to, że po prostu działa:

- **Spójne zrzuty ekranu**, wszyscy używają tych samych przeglądarek w chmurze
- **Działa lokalnie**, deweloperzy mogą uruchamiać i aktualizować testy wizualne na swoich maszynach
- **Płacisz za to, co używasz**, tylko testy wizualne zużywają minuty usługi
- **Nie potrzeba konfiguracji Docker ani workflow**, nic do zarządzania ani utrzymywania

#### Konfiguracja CI

W swoim CI dodaj sekrety:

```yaml
env:
  PLAYWRIGHT_SERVICE_URL: ${{ vars.PLAYWRIGHT_SERVICE_URL }}
  PLAYWRIGHT_SERVICE_ACCESS_TOKEN: ${{ secrets.PLAYWRIGHT_SERVICE_ACCESS_TOKEN }}
```

Następnie uruchom swoje testy jak zwykle. Usługa zajmie się resztą.

::::

### Więc które wybrać?

Oba podejścia działają. Prawdziwe pytanie brzmi, jakie punkty bólu są najważniejsze dla Twojego
zespołu.

Jeśli jesteś już głęboko w ekosystemie GitHub, trudno pobić GitHub Actions.
Darmowe dla open source, działa z dowolnym dostawcą przeglądarek i kontrolujesz
wszystko.

Wada? Ta rozmowa "działa na mojej maszynie", gdy ktoś generuje
zrzuty ekranu lokalnie i nie pasują już do oczekiwań CI.

Usługa chmurowa ma sens, jeśli deweloperzy muszą uruchamiać testy wizualne lokalnie.

Niektóre zespoły mają projektantów sprawdzających swoją pracę lub deweloperów, którzy wolą wychwytywać
problemy przed pushowaniem. Pozwala to pominąć cykl push-czekaj-sprawdź-napraw-push.

Nadal niezdecydowany? Zacznij od GitHub Actions. Zawsze możesz dodać usługę
chmurową później, jeśli lokalne testowanie stanie się problemem.
