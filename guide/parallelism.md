---
title: Równoległość | Przewodnik
outline: deep
---

# Równoległość

## Równoległość plików

Domyślnie Vitest uruchamia _pliki testowe_ równolegle. W zależności od określonego `pool`, Vitest używa różnych mechanizmów do zrównoleglenia plików testowych:

- `forks` (domyślny) i `vmForks` uruchamiają testy w różnych [procesach potomnych](https://nodejs.org/api/child_process.html)
- `threads` i `vmThreads` uruchamiają testy w różnych [wątkach workerów](https://nodejs.org/api/worker_threads.html)

Zarówno "procesy potomne", jak i "wątki workerów" są nazywane "workerami". Możesz skonfigurować liczbę działających workerów za pomocą opcji [`maxWorkers`](/config/#maxworkers).

Jeśli masz dużo testów, zwykle szybciej jest uruchamiać je równolegle, ale zależy to również od projektu, środowiska i stanu [izolacji](/config/#isolate). Aby wyłączyć równoległość plików, możesz ustawić [`fileParallelism`](/config/#fileparallelism) na `false`. Aby dowiedzieć się więcej o możliwych usprawnieniach wydajności, przeczytaj [Przewodnik wydajności](/guide/improving-performance).

## Równoległość testów

W przeciwieństwie do _plików testowych_, Vitest uruchamia _testy_ sekwencyjnie. Oznacza to, że testy wewnątrz pojedynczego pliku testowego będą uruchamiane w kolejności, w jakiej zostały zdefiniowane.

Vitest wspiera opcję [`concurrent`](/api/#test-concurrent) do uruchamiania testów razem. Jeśli ta opcja jest ustawiona, Vitest zgrupuje współbieżne testy w tym samym _pliku_ (liczba jednocześnie uruchomionych testów zależy od opcji [`maxConcurrency`](/config/#maxconcurrency)) i uruchomi je za pomocą [`Promise.all`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all).

Vitest nie wykonuje żadnej inteligentnej analizy i nie tworzy dodatkowych workerów do uruchamiania tych testów. Oznacza to, że wydajność twoich testów poprawi się tylko wtedy, gdy polegasz mocno na operacjach asynchronicznych. Na przykład te testy nadal będą uruchamiane jeden po drugim, mimo że opcja `concurrent` jest określona. Dzieje się tak, ponieważ są synchroniczne:

```ts
test.concurrent('pierwszy test', () => {
  expect(1).toBe(1)
})

test.concurrent('drugi test', () => {
  expect(2).toBe(2)
})
```

Jeśli chcesz uruchamiać wszystkie testy współbieżnie, możesz ustawić opcję [`sequence.concurrent`](/config/#sequence-concurrent) na `true`.
