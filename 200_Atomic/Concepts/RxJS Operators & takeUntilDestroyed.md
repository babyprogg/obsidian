---
type: atomic
status: seedling
domain:
created: 2026-09-02
updated: 2026-09-02
tags:
  - 
related: []
---
# RxJS Operators & takeUntilDestroyed

## Core Idea

**RxJS-операторы — это функции для трансформации, фильтрации и управления потоками `Observable`, а `takeUntilDestroyed` — Angular-оператор, который автоматически завершает Observable при уничтожении соответствующего Angular-контекста.**

Ментальная модель:

```text
Observable
    ↓
  map()
    ↓
 filter()
    ↓
switchMap()
    ↓
takeUntilDestroyed()
    ↓
subscribe()
```

Операторы формируют поведение потока, а `takeUntilDestroyed` связывает его жизненный цикл с Angular.

## Why It Matters

Подписка (`subscribe`) создаёт связь между Observable и потребителем данных.

Если эта связь продолжает существовать после уничтожения компонента, можно получить:

- memory leaks;
    
- ненужную обработку данных;
    
- повторные побочные эффекты;
    
- лишние сетевые операции;
    
- ошибки, связанные с уже уничтоженным UI.
    

Раньше для ручного управления подписками часто использовали:

```text
destroy$
   ↓
Subject<void>
   ↓
takeUntil(destroy$)
   ↓
ngOnDestroy()
   ↓
destroy$.next()
   ↓
destroy$.complete()
```

`takeUntilDestroyed` позволяет связать Observable непосредственно с Angular `DestroyRef` и убрать этот boilerplate.

```text
Component
    ↓
Observable
    ↓
takeUntilDestroyed()
    ↓
Component destroyed
    ↓
Observable completes
```

## Key Points

### 1. Observable и Operators

`Observable` можно представить как поток данных:

```text
Observable
──────────────────────────────→
  1    2    3    4    5    6
```

Операторы изменяют этот поток.

Например:

```typescript
id="9h4k2m"
source$.pipe(
  map(value => value * 2)
);
```

```text
1 → 2
2 → 4
3 → 6
```

Другой оператор:

```typescript
id="q7m3x8"
source$.pipe(
  filter(value => value > 10)
);
```

```text
5 → ✕
15 → 15
20 → 20
```

А `switchMap` позволяет переключаться между Observable:

```text
Search Input
     ↓
switchMap()
     ↓
HTTP Request
     ↓
Response
```

---

### 2. Проблема с `subscribe()`

Сам по себе `subscribe()` не является плохим.

Проблема появляется, когда подписка должна жить только пока существует определённый Angular-контекст.

Например:

```typescript
id="k5p8v1"
this.orderStore.orders$
  .subscribe(orders => {
    this.processOrders(orders);
  });
```

Если `orders$` является долгоживущим Observable, подписка может продолжать существовать после уничтожения компонента.

Поэтому для таких подписок необходимо учитывать lifecycle.

---

### 3. `takeUntilDestroyed()`

`takeUntilDestroyed` находится в:

```typescript
id="m2x7q9"
@angular/core/rxjs-interop
```

Он использует Angular `DestroyRef`, чтобы завершить Observable, когда уничтожается соответствующий Angular-контекст.

```typescript
id="v8n3k5"
this.userService.getUser().pipe(
  takeUntilDestroyed()
);
```

Ментальная модель:

```text
Observable
    ↓
takeUntilDestroyed()
    ↓
Component alive?
    ↓
   YES ─────→ continue
    │
    NO
    ↓
complete()
```

---

### 4. Injection Context

Без аргументов:

```typescript
id="f4q9m2"
takeUntilDestroyed()
```

оператор должен находиться в Angular **injection context**, где доступен `DestroyRef`.

Например, при инициализации поля класса:

```typescript
id="x6p3k8"
@Component({...})
export class UserProfileComponent {

  private userService = inject(UserService);

  user$ = this.userService.getUser().pipe(
    map(user => user.name),
    takeUntilDestroyed()
  );
}
```

Angular понимает, с каким `DestroyRef` нужно связать Observable.

---

### 5. Явная передача `DestroyRef`

Если `takeUntilDestroyed()` вызывается в месте, где injection context недоступен, можно передать `DestroyRef` явно.

```typescript
id="n7m4q1"
private destroyRef = inject(DestroyRef);
```

Затем:

```typescript
id="p8k2v5"
takeUntilDestroyed(this.destroyRef)
```

Например:

```typescript
id="w3q9m6"
@Component({...})
export class OrderComponent implements OnInit {

  private destroyRef = inject(DestroyRef);
  private orderStore = inject(OrderStore);

  ngOnInit() {
    this.orderStore.orders$
      .pipe(
        filter(orders => orders.length > 0),
        takeUntilDestroyed(this.destroyRef)
      )
      .subscribe(orders => {
        this.processOrders(orders);
      });
  }
}
```

Здесь:

```text
OrderComponent
      ↓
DestroyRef
      ↓
takeUntilDestroyed()
      ↓
orders$
      ↓
Component destroyed
      ↓
Subscription completed
```

---

### 6. `takeUntilDestroyed` не заменяет все способы управления Observable

Важно понимать:

> `takeUntilDestroyed` решает проблему lifecycle конкретного Angular-контекста, но не является универсальным оператором отмены всех RxJS-потоков.

Например, для поиска:

```text
User types
   ↓
debounceTime()
   ↓
switchMap()
   ↓
HTTP
```

`switchMap` решает другую задачу — переключение между конкурентными Observable.

А:

```text
takeUntilDestroyed()
```

решает задачу:

```text
"Что делать с подпиской,
когда компонент уничтожается?"
```

Это разные уровни ответственности.

## Examples

### Example 1

### Subscription в контексте инициализации поля

```typescript
import { Component, inject } from '@angular/core';
import { map } from 'rxjs';

import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({...})
export class UserProfileComponent {

  private userService = inject(UserService);

  user$ = this.userService.getUser().pipe(
    map(user => user.name),
    takeUntilDestroyed()
  );
}
```

Здесь Angular может получить `DestroyRef` автоматически.

При уничтожении компонента Observable завершается.

```text
UserProfileComponent
        ↓
      created
        ↓
    subscription
        ↓
      active
        ↓
component destroyed
        ↓
 Observable completes
```

---

### Example 2

### Подписка внутри `ngOnInit`

Когда подписка создаётся внутри метода жизненного цикла, можно явно передать `DestroyRef`:

```typescript
import {
  Component,
  DestroyRef,
  inject,
  OnInit
} from '@angular/core';

import { filter } from 'rxjs';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({...})
export class OrderComponent implements OnInit {

  private destroyRef = inject(DestroyRef);
  private orderStore = inject(OrderStore);

  ngOnInit() {
    this.orderStore.orders$
      .pipe(
        filter(orders => orders.length > 0),
        takeUntilDestroyed(this.destroyRef)
      )
      .subscribe(orders => {
        this.processOrders(orders);
      });
  }

  private processOrders(orders: Order[]) {
    // ...
  }
}
```

---

### Example 3

### Search + `switchMap` + `takeUntilDestroyed`

Здесь можно увидеть, как разные RxJS-инструменты решают разные задачи:

```typescript
id="c5m8q2"
searchTerm$
  .pipe(
    debounceTime(300),

    switchMap(term =>
      this.userService.search(term)
    ),

    takeUntilDestroyed()
  )
  .subscribe(users => {
    this.users.set(users);
  });
```

Ответственность каждого оператора:

```text
searchTerm$
    ↓
debounceTime()
    ↓
Не отправлять запрос
на каждый символ
    ↓
switchMap()
    ↓
Переключиться на
последний запрос
    ↓
takeUntilDestroyed()
    ↓
Остановиться при
уничтожении компонента
```

Это хороший пример того, почему не стоит воспринимать RxJS-операторы как взаимозаменяемые инструменты.

## How to Apply

### 1. Импортировать `takeUntilDestroyed`

```typescript
id="r7k3m9"
import {
  takeUntilDestroyed
} from '@angular/core/rxjs-interop';
```

---

### 2. Добавлять его в Observable pipeline

Например:

```typescript
id="x4p8q1"
observable$
  .pipe(
    map(...),
    filter(...),
    takeUntilDestroyed()
  )
  .subscribe(...);
```

Обычно его удобно располагать ближе к концу pipeline.

---

### 3. Использовать без аргументов там, где доступен injection context

Например:

```typescript
id="n5m2v8"
user$ = this.userService.getUser().pipe(
  takeUntilDestroyed()
);
```

---

### 4. Передавать `DestroyRef`, если контекст нужно указать явно

```typescript
id="q9k4x6"
private destroyRef = inject(DestroyRef);
```

и:

```typescript
id="b7m3p1"
takeUntilDestroyed(this.destroyRef)
```

---

### 5. Не делать `subscribe()` внутри сервисов без необходимости

Предпочтительно:

```typescript
id="m8q2v5"
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

А lifecycle подписки контролировать там, где происходит subscription:

```text
Service
  ↓
Observable
  ↓
Component
  ↓
subscribe()
  ↓
takeUntilDestroyed()
```

---

### 6. Использовать AsyncPipe, когда Observable нужен только в template

Если Observable используется непосредственно в шаблоне:

```html
id="v3n7k2"
@for (user of users$ | async; track user.id) {
  <p>{{ user.name }}</p>
}
```

`AsyncPipe` сам управляет подпиской и её завершением.

В таком случае ручной `takeUntilDestroyed()` может вообще не понадобиться.

## Connections

### Supports

- [[Angular Reactive Patterns]] — помогает безопасно связывать RxJS-потоки с lifecycle Angular-компонентов.
    
- [[RxJS Operators Overview]] — `takeUntilDestroyed` является частью Observable pipeline наряду с `map`, `filter`, `switchMap` и другими операторами.
    
- [[Angular Dependency Injection]] — использует `DestroyRef`, который Angular предоставляет через DI.
    
- [[Angular Signals]] — Signals и RxJS могут использоваться вместе, например через `toSignal()` и `toObservable()`.
    

### Contradicts

- [[Manual Unsubscribe via Subject]] — во многих случаях устраняет необходимость создавать `destroy$ = new Subject<void>()` и вручную реализовывать `ngOnDestroy`.
    
- [[Manual Subscription Management]] — уменьшает количество lifecycle-кода, необходимого для управления подписками.
    

### Extends

- [[RxJS takeUntil]] — `takeUntilDestroyed` предоставляет Angular-ориентированный способ завершения Observable на основе `DestroyRef`.
    
- [[Angular Component Lifecycle]] — связывает lifecycle компонента с lifecycle Observable subscription.
    
- [[RxJS Observables]] — добавляет Angular-aware управление временем жизни Observable.
    

## Questions

- Чем `takeUntilDestroyed` отличается от `AsyncPipe` с точки зрения производительности и удобства?
    
- В каких случаях `takeUntilDestroyed()` без аргументов не сможет получить injection context?
    
- Как правильно использовать `takeUntilDestroyed` в `providedIn: 'root'` сервисах?
    
- Что произойдёт, если root-сервис подпишется на долгоживущий Observable через `takeUntilDestroyed()`?
    
- Когда лучше использовать `switchMap`, а когда `takeUntilDestroyed`?
    
- Как правильно комбинировать `takeUntilDestroyed` с `toSignal()`?
    
- Нужно ли использовать `takeUntilDestroyed` для HTTP-запросов, если `HttpClient` Observable обычно завершается самостоятельно?
    
- Как `takeUntilDestroyed` ведёт себя в директивах и embedded views?
    

## Sources

- Angular Official Documentation — `@angular/core/rxjs-interop`
    
- Angular Official Documentation — `takeUntilDestroyed`
    
- Angular Official Documentation — `DestroyRef`
    
- RxJS Documentation — Pipeable Operators
    
- RxJS Documentation — `takeUntil`
    

## Feynman Analogy

### Жизненный цикл абонемента

Представь, что компонент оформил подписку на поток данных:

```text
Component
   ↓
"Я хочу получать данные"
   ↓
Observable
```

Пока компонент существует:

```text
Component alive
      ↓
Получаем данные
```

Но компонент уничтожается:

```text
Component destroyed
      ↓
"Мне больше не нужны данные"
      ↓
Subscription completed
```

`takeUntilDestroyed` — это как автоматическая отмена подписки:

> **«Пока я жив — продолжай. Как только меня уничтожили — закончи поток».**

А `switchMap` решает совершенно другую задачу:

> **«Если появился новый запрос — переключись на него».**

Поэтому:

```text
takeUntilDestroyed()
→ когда закончить subscription?

switchMap()
→ какой Observable сейчас должен быть активным?
```

---

**Review Status**: Draft

- Last reviewed: 2026-09-02
    
- Next review: 2026-10-02