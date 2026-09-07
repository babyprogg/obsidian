---
type: atomic
status: seedling
domain:
created: 2026-09-01
updated: 2026-09-01
tags:
  - 
related: []
---
# Angular Signals

## Core Idea

**Angular Signals — это реактивная система, основанная на графе зависимостей, которая отслеживает изменения состояния и выполняет точечные обновления UI.**

Вместо того чтобы постоянно проверять всё дерево компонентов, Angular знает:

```text
Signal изменился
      ↓
Какие значения от него зависят?
      ↓
Какие части UI используют эти значения?
      ↓
Обновить только их
```

Основные примитивы:

```text
signal()    → состояние
computed()  → производное состояние
effect()    → side effect
```

## Why It Matters

Signals упрощают управление состоянием и позволяют Angular более точно отслеживать зависимости между данными.

Без Signals состояние часто выглядит примерно так:

```text
State
 ↓
Change Detection
 ↓
Проверка компонентов
 ↓
Проверка bindings
 ↓
UI update
```

С Signals Angular получает явный граф зависимостей:

```text
count
  ↓
doubleCount
  ↓
Template
```

Когда `count` изменяется, Angular знает, какие значения зависят от него.

Signals также позволяют:

- избавиться от части ручного управления subscriptions для UI state;
    
- использовать memoization для производных значений;
    
- автоматически отслеживать зависимости;
    
- создавать более предсказуемую модель реактивного состояния;
    
- сочетать локальное состояние с Angular template reactivity.
    

## Key Points

### 1. `signal()` — writable state

`signal()` создаёт реактивное состояние, значение которого можно изменять.

```typescript
const count = signal(0);
```

Чтение:

```typescript
count()
```

Изменение:

```typescript
count.set(5);
```

Или относительно текущего значения:

```typescript
count.update(value => value + 1);
```

Ментальная модель:

```text
signal(0)
   ↓
  count
   ↓
count.set(5)
   ↓
  count()
   ↓
    5
```

---

### 2. `computed()` — derived state

`computed()` создаёт **read-only производное состояние**, которое зависит от других Signals.

```typescript
const count = signal(0);

const doubleCount = computed(() => count() * 2);
```

Теперь:

```text
count = 5
   ↓
doubleCount
   ↓
10
```

`computed()` обладает двумя важными свойствами.

#### Lazy Evaluation

Значение вычисляется только тогда, когда оно действительно требуется.

#### Memoization

Результат сохраняется и пересчитывается только при изменении зависимостей.

Например:

```typescript
const count = signal(5);

const doubleCount = computed(() => {
  console.log('Calculating...');

  return count() * 2;
});
```

Angular не будет выполнять вычисление заново без необходимости.

Ментальная модель:

```text
       count
         ↓
    ┌──────────┐
    │computed  │
    └──────────┘
         ↓
   cached result
```

---

### 3. `effect()` — side effects

`effect()` выполняет функцию, которая автоматически запускается снова, когда изменяются Signals, которые она прочитала.

```typescript
const count = signal(0);

effect(() => {
  console.log(`Current count: ${count()}`);
});
```

Когда:

```typescript
count.set(5);
```

effect автоматически выполнится снова.

```text
count
 ↓
effect()
 ↓
console.log()
```

### Главное правило

`effect()` предназначен прежде всего для **side effects**, а не для вычисления состояния.

Хорошие варианты:

- logging;
    
- синхронизация с `localStorage`;
    
- взаимодействие с внешними API;
    
- ручные DOM-операции;
    
- интеграция с внешними библиотеками.
    

Не стоит использовать `effect()` как замену `computed()` для создания derived state.

Плохо:

```typescript
const count = signal(0);
const doubleCount = signal(0);

effect(() => {
  doubleCount.set(count() * 2);
});
```

Лучше:

```typescript
const count = signal(0);

const doubleCount = computed(() => count() * 2);
```

Ментальная модель:

```text
computed()
    ↓
"Какое значение должно быть?"

effect()
    ↓
"Что сделать из-за изменения?"
```

---

### 4. Dependency Graph

Angular Signals строят граф зависимостей.

Например:

```typescript
const count = signal(0);

const doubleCount = computed(() => count() * 2);

effect(() => {
  console.log(doubleCount());
});
```

Зависимости:

```text
        count
          │
          ↓
     doubleCount
          │
          ↓
       effect
```

Если `count` изменится:

```text
count.set(10)
      ↓
doubleCount → 20
      ↓
effect()
      ↓
console.log()
```

Angular знает всю цепочку зависимостей автоматически.

---

### 5. Signals в Templates

Signal читается в template как функция:

```html
<h1>{{ count() }}</h1>
```

Angular видит, что template зависит от `count`.

Если:

```typescript
count.set(10);
```

Angular знает, что соответствующая часть UI использует этот Signal.

```text
count
 ↓
Template dependency
 ↓
UI update
```

---

## Examples

### Example 1

### Basic Reactive Counter

```typescript
import {
  signal,
  computed,
  effect
} from '@angular/core';

const count = signal(0);

const doubleCount = computed(() => count() * 2);

effect(() => {
  console.log(
    `Current count is ${count()} and double is ${doubleCount()}`
  );
});

count.set(5);
```

После:

```typescript
count.set(5);
```

получаем:

```text
count       → 5
doubleCount → 10
effect      → запускается снова
```

Архитектурно:

```text
        count
          ↓
    doubleCount
          ↓
       effect
```

---

### Example 2

### E-commerce Shopping Cart

Есть корзина:

```typescript
const items = signal([
  {
    name: 'Laptop',
    price: 1000
  },
  {
    name: 'Mouse',
    price: 50
  }
]);
```

Общую стоимость можно получить через `computed()`:

```typescript
const totalPrice = computed(() =>
  items().reduce(
    (sum, item) => sum + item.price,
    0
  )
);
```

Налог также является производным значением:

```typescript
const taxAmount = computed(
  () => totalPrice() * 0.2
);
```

Получается:

```text
items
  ↓
totalPrice
  ↓
taxAmount
```

Если товары изменились:

```typescript
items.set([...])
```

Angular автоматически пересчитает зависимые значения.

Для синхронизации с `localStorage` можно использовать `effect()`:

```typescript
effect(() => {
  localStorage.setItem(
    'cart_total',
    totalPrice().toString()
  );
});
```

Здесь разделение ответственности выглядит правильно:

```text
signal()
   ↓
State

computed()
   ↓
Derived State

effect()
   ↓
External Side Effect
```

## How to Apply

### 1. Используй `signal()` для локального состояния

Вместо обычного mutable state:

```typescript
count = 0;
```

можно использовать:

```typescript
count = signal(0);
```

Изменение:

```typescript
this.count.set(10);
```

Чтение:

```typescript
this.count()
```

---

### 2. Используй `computed()` для derived state

Если значение можно получить из других Signals:

```typescript
const fullName = computed(
  () => `${firstName()} ${lastName()}`
);
```

не нужно вручную синхронизировать отдельный Signal.

```text
firstName ──┐
            ├──→ fullName
lastName  ──┘
```

---

### 3. Используй `effect()` только для side effects

Хорошо:

```typescript
effect(() => {
  localStorage.setItem(
    'theme',
    theme()
  );
});
```

Плохо:

```typescript
effect(() => {
  total.set(price() * quantity());
});
```

Вместо этого:

```typescript
const total = computed(
  () => price() * quantity()
);
```

---

### 4. Читай Signals в template через `()`

```html
<p>{{ count() }}</p>
<p>{{ doubleCount() }}</p>
```

Так Angular может отслеживать dependency relationship между template и Signals.

---

### 5. Не пытайся заменить RxJS полностью

Signals и RxJS решают разные задачи.

Упрощённо:

```text
Signals
   ↓
State / synchronous reactivity

RxJS
   ↓
Async streams / events
```

Например:

```text
User clicks
     ↓
Observable
     ↓
HTTP request
     ↓
Response
     ↓
Signal
     ↓
UI
```

Для интеграции используются:

```typescript
toSignal()
```

и:

```typescript
toObservable()
```

## Connections

### Supports

- [[Change Detection Strategy OnPush]] — Signals хорошо сочетаются с `OnPush` и позволяют Angular точнее отслеживать изменения состояния.
    
- [[Angular Templates]] — чтение Signals в template создаёт реактивную зависимость между состоянием и UI.
    
- [[Angular Dependency Injection]] — Signals и `effect()` часто используются внутри сервисов, получаемых через DI.
    

### Contradicts

- [[Manual Change Detection]] — Signals уменьшают необходимость вручную вызывать `ChangeDetectorRef` для обновления UI.
    
- [[Zone.js]] — Signals позволяют Angular отслеживать зависимости состояния более точно, уменьшая необходимость полагаться исключительно на глобальное обнаружение изменений через Zone.js.
    

### Extends

- [[RxJS Observables]] — Signals и Observables дополняют друг друга: Signals хорошо подходят для состояния, а RxJS — для асинхронных потоков и событий.
    
- [[Reactive Programming]] — Signals предоставляют декларативную модель реактивного состояния.
    
- [[Angular State Management]] — Signals могут использоваться как основа для локального и shared state.
    

## Questions

- Как Signals взаимодействуют с RxJS в сложных async workflows?
    
- Когда использовать `toSignal()`, а когда `toObservable()`?
    
- Как лучше организовывать Signal-based state в глобальных сервисах?
    
- Когда Signal должен быть локальным состоянием компонента, а когда shared state?
    
- Как Signals взаимодействуют с `OnPush` внутри глубоко вложенного component tree?
    
- Как работает dependency tracking внутри `computed()`?
    
- Как Angular определяет, какие template bindings зависят от конкретного Signal?
    
- Какие ограничения и особенности есть у `effect()`?
    
- Как правильно тестировать `computed()` и `effect()`?
    

## Sources

- Angular Official Documentation — Signals
    
- Angular Official Documentation — `signal()`
    
- Angular Official Documentation — `computed()`
    
- Angular Official Documentation — `effect()`
    
- Angular Official Documentation — RxJS Interop
    

## Feynman Analogy

### Smart Excel Sheet

Можно представить Signals как Excel:

```text
A1 = signal
    ↓
обычная ячейка со значением

B1 = computed
    ↓
формула, зависящая от A1

effect
    ↓
внешнее действие,
которое происходит при изменении результата
```

Например:

```text
A1 = 10

B1 = A1 * 2
    ↓
20

effect
    ↓
записать 20 в localStorage
```

Главная идея:

> **`signal()` хранит состояние, `computed()` вычисляет производное состояние, а `effect()` реагирует на изменения внешним действием.**

---

**Review Status**: Draft

- Last reviewed: 2026-09-01
    
- Next review: 2026-10-01