# Mockowanie klas

Możesz mockować całą klasę za pomocą jednego wywołania [`vi.fn`](/api/vi#fn).

```ts
class Dog {
  name: string

  constructor(name: string) {
    this.name = name
  }

  static getType(): string {
    return 'animal'
  }

  greet = (): string => {
    return `Cześć! Mam na imię ${this.name}!`
  }

  speak(): string {
    return 'hau!'
  }

  isHungry() {}
  feed() {}
}
```

Możemy odtworzyć tę klasę za pomocą `vi.fn` (lub `vi.spyOn().mockImplementation()`):

```ts
const Dog = vi.fn(class {
  static getType = vi.fn(() => 'mockowane zwierzę')

  constructor(name) {
    this.name = name
  }

  greet = vi.fn(() => `Cześć! Mam na imię ${this.name}!`)
  speak = vi.fn(() => 'głośne hau!')
  feed = vi.fn()
})
```

::: warning
Jeśli z funkcji konstruktora zwrócona zostanie wartość nieprymitywna, ta wartość stanie się wynikiem wyrażenia new. W tym przypadku `[[Prototype]]` może nie być poprawnie powiązany:

```ts
const CorrectDogClass = vi.fn(function (name) {
  this.name = name
})

const IncorrectDogClass = vi.fn(name => ({
  name
}))

const Marti = new CorrectDogClass('Marti')
const Newt = new IncorrectDogClass('Newt')

Marti instanceof CorrectDogClass // ✅ true
Newt instanceof IncorrectDogClass // ❌ false!
```

Jeśli mockujesz klasy, preferuj składnię klasy zamiast funkcji.
:::

::: tip KIEDY UŻYWAĆ?
Ogólnie rzecz biorąc, odtwarzasz klasę w ten sposób wewnątrz fabryki modułu, jeśli klasa jest reeksportowana z innego modułu:

```ts
import { Dog } from './dog.js'

vi.mock(import('./dog.js'), () => {
  const Dog = vi.fn(class {
    feed = vi.fn()
    // ... inne mocki
  })
  return { Dog }
})
```

Ta metoda może być również użyta do przekazania instancji klasy do funkcji, która przyjmuje ten sam interfejs:

```ts [src/feed.ts]
function feed(dog: Dog) {
  // ...
}
```
```ts [tests/dog.test.ts]
import { expect, test, vi } from 'vitest'
import { feed } from '../src/feed.js'

const Dog = vi.fn(class {
  feed = vi.fn()
})

test('może karmić psy', () => {
  const dogMax = new Dog('Max')

  feed(dogMax)

  expect(dogMax.feed).toHaveBeenCalled()
  expect(dogMax.isHungry()).toBe(false)
})
```
:::

Teraz, gdy tworzymy nową instancję klasy `Dog`, jej metoda `speak` (wraz z `feed` i `greet`) jest już mockowana:

```ts
const Cooper = new Dog('Cooper')
Cooper.speak() // głośne hau!
Cooper.greet() // Cześć! Mam na imię Cooper!

// możesz użyć wbudowanych asercji, aby sprawdzić poprawność wywołania
expect(Cooper.speak).toHaveBeenCalled()
expect(Cooper.greet).toHaveBeenCalled()

const Max = new Dog('Max')

// metody nie są współdzielone między instancjami, jeśli przypisałeś je bezpośrednio
expect(Max.speak).not.toHaveBeenCalled()
expect(Max.greet).not.toHaveBeenCalled()
```

Możemy przypisać nową wartość zwracaną dla konkretnej instancji:

```ts
const dog = new Dog('Cooper')

// "vi.mocked" to helper typów, ponieważ
// TypeScript nie wie, że Dog jest mockowaną klasą,
// owija każdą funkcję w typ Mock<T>
// bez walidacji, czy funkcja jest mockiem
vi.mocked(dog.speak).mockReturnValue('hau hau')

dog.speak() // hau hau
```

Aby mockować właściwość, możemy użyć metody `vi.spyOn(dog, 'name', 'get')`. To umożliwia używanie asercji szpiegowskich na mockowanej właściwości:

```ts
const dog = new Dog('Cooper')

const nameSpy = vi.spyOn(dog, 'name', 'get').mockReturnValue('Max')

expect(dog.name).toBe('Max')
expect(nameSpy).toHaveBeenCalledTimes(1)
```

::: tip
Możesz również szpiegować gettery i settery używając tej samej metody.
:::

::: danger
Używanie klas z `vi.fn()` zostało wprowadzone w Vitest 4. Wcześniej trzeba było używać `function` i dziedziczenia `prototype` bezpośrednio. Zobacz [przewodnik v3](https://v3.vitest.dev/guide/mocking.html#classes).
:::
