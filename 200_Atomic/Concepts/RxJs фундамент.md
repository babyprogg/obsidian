---
type: atomic
status: seedling
domain:
created: 2026-06-08
updated: 2026-06-08
tags:
  - 
related: []
---
# <% tp.file.title %>

## Core Idea
RxJS (Reactive Extensions for JavaScript) — это инструмент для управления асинхронными данными. Он превращает любые события (клики, HTTP-запросы, ввод текста) в управляемые «потоки воды» (Observable), внутри которых данные можно фильтровать, трансформировать и комбинировать до того, как они попадут на экран.

## Why It Matters
Без RxJS сложная асинхронность в Angular превращается в "Callback Hell" и заставляет плодить кучу флагов состояния (isLoading, и т.д.). RxJS позволяет декларативно описать логику приложения как единую схему водопровода, автоматически решая проблемы утечек памяти, отмены старых запросов и гонки данных (Race Conditions).

## Key Points
* Потоки (Observable) «холодные» по умолчанию: Вода не потечет по трубе, пока на нее никто не подписался (.subscribe()).
* Subject — это ретранслятор (Multicast): Позволяет раздавать одни данные множеству подписчиков одновременно. 
  * BehaviorSubject — всегда помнит последнее актуальное значение (идеален для хранения состояния/стейта).
  * Subject — ничего не помнит, просто триггерит событие в момент наступления (как клик).
* Flattening Operators (Выравнивание): Превращают "поток потоков" в один плоский поток.
  * switchMap — отменяет прошлый запрос (нужен для поиска/пагинации).
  * exhaustMap — игнорирует спам-клики, пока первый запрос не завершится (нужен для оплат/сабмита форм).
  * concatMap — ставит запросы в строгую очередь (нужен для сохранения порядка).
* catchError Placement: Если написать catchError во внутреннем пайпе (внутри switchMap), ошибка изолируется, и главный поток останется жив. Если написать во внешнем пайпе — ошибка убьет весь стрим навсегда.

## Examples

### Example 1: Живой поиск (Архитектурный паттерн Typeahead)
Пользователь быстро набирает слово. Нам не нужно отправлять запрос на каждую букву и выводить устаревшие ответы, если они придут позже.

```typescript
this.searchTerms$.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => 
    this.api.searchUsers(term).pipe(
      catchError(() => of([])) // Гасим ошибку ТУТ, чтобы поиск не умер
    )
  )
).subscribe(users => this.users = users);
```
### Example 2: Кнопка безопасной оплаты (Защита от двойного списания)

Пользователь кликает по кнопке «Оплатить» несколько раз подряд. Деньги должны списаться только один раз.
```
this.paymentClick$.pipe(
  exhaustMap(() => this.api.chargeMoney(amount)) // Игнорирует новые клики, пока идет обработка
).subscribe({
  next: (receipt) => this.showSuccess(receipt),
  error: (err) => this.showError(err)
});
```
## How to Apply

1. Не подписывайся в компонентах вручную: По возможности используй async пайп в HTML шаблоне (`data$ | async`) — Angular сам подпишется и сам отпишется.
    
2. Если подписываешься в `.ts`: Всегда используй оператор `takeUntilDestroyed()` в конструкторе, чтобы избежать утечек памяти при уничтожении компонента.
    
3. Прячь Subject: Держи BehaviorSubject приватным внутри сервиса (`private _state$`), а наружу отдавай только `state$ = this._state$.asObservable()`.

## Connections

### Supports(пока не созданы)

- [[Angular Change Detection (OnPush)]] - Потоки идеально поставляют данные для стратегии OnPush через async пайп.
    
- [[State Management patterns]] - Потоки лежат в основе управления состоянием приложения.

### Contradicts

- [[Imperative Programming (Promises)]] - Промисы одноразовые и не умеют отменяться, в отличие от отменяемых потоков RxJS.
    

### Extends

- [[Asynchronous JavaScript]] - Расширяет базовые концепции Event Loop и асинхронных событий до уровня мощных функциональных пайплайнов.
    

## Questions

- Как правильно тестировать RxJS потоки с помощью TestScheduler и мраморного тестирования (Marble Testing)?
    
- В каких редких кейсах mergeMap может вызвать перегрузку памяти (Memory Bloat)?
    

## Sources

- Официальная документация: [rxjs.dev](https://rxjs.dev)
    
- Интерактивные диаграммы операторов: [rxmarbles.com](https://rxmarbles.com)
    

**Review Status:**

- **Last reviewed:** <% tp.date.now("YYYY-MM-DD") %>
    
- **Next review:** <% tp.date.now("YYYY-MM-DD", 30) %>