---
type: atomic
status: seedling
domain:
created: 2026-08-23
updated: 2026-08-23
tags:
  - 
related: []
---
# Change Detection & OnPush Strategy in Angular

## Core Idea
Change Detection — это механизм синхронизации состояния класса с HTML-шаблоном, а OnPush — стратегия, которая отсекает ненужные проверки компонента, дожидаясь только явных триггеров (изменение `@Input` по ссылке, события в шаблоне или Signals).

## Why It Matters
Без понимания Change Detection приложение с большим количеством компонентов начнет "фризить" и медленно работать, так как стратегия `Default` (вместе с Zone.js) при любом клике перепроверяет абсолютно всё дерево. OnPush и Zoneless-реактивность выстраивают хирургически точечное обновление DOM с высокой производительностью.

Key Points
* **Default vs OnPush**: `Default` проверяет всё дерево сверху вниз при любом асинхронном событии. `OnPush` замораживает компонент до тех пор, пока не произойдет точечный триггер.
* **Иммутабельность — фундамент OnPush**: Изменение внутренних свойств объекта (`user.name = 'Max'`) не изменит ссылку на объект, и OnPush-компонент не обновится. Нужно передавать новый объект (`user = { ...user, name: 'Max' }`).
* **Триггеры OnPush в Zone.js**: Изменение ссылки `@Input`, вызов `(click)` или другого события из шаблона компонента, `async` pipe, либо ручной вызов `markForCheck()`.
* **Zoneless & Signals (Angular 18+)**: В Zoneless-режиме отключается глобальная Zone.js. Дерево больше не проверяется целиком; обновления происходят точечно через сигналы (`signal.set()`), которые напрямую уведомляют конкретные узлы DOM.

### Examples
Example 1: Мутация vs Иммутабельность в OnPush
```typescript
@Component({
  selector: 'app-user-card',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div>{{ user.name }}</div>`
})
export class UserCardComponent {
  @Input({ required: true }) user!: { name: string };
}

// ❌ Плохо (НЕ обновит DOM в OnPush):
this.user.name = 'Alex'; 

// ✅ Правильно (создаем новую ссылку, OnPush сработает):
this.user = { ...this.user, name: 'Alex' };```
```
Example 2: Реактивность без Zone.js через Signals
``` typescript
@Component({
  selector: 'app-counter',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>Count: {{ count() }}</p>
    <button (click)="increment()">+1</button>
  `
})
export class CounterComponent {
  count = signal(0);

  increment() {
    // В Zoneless режиме signal.set() напрямую обновляет узел DOM без прохода по всему дереву
    this.count.update(c => c + 1);
  }
}
```

## How to Apply

1. Устанавливайте `changeDetection: ChangeDetectionStrategy.OnPush` для всех создаваемых компонентов по умолчанию.
    
2. Используйте Angular Signals (`signal()`, `computed()`) для хранения локального состояния и пропсов — это обеспечит готовность к Zoneless.
    
3. Избегайте прямых мутаций объектов и массивов; используйте spread-оператор (`...`) или иммутабельные методы (`map`, `filter`).
    
4. При работе с RxJS-потоками в шаблоне всегда используйте `async` pipe или трансформируйте поток через `toSignal()`.
    
5. Не вызывайте методы или тяжелые вычисления прямо в шаблонах `{{ calculateData() }}`, чтобы не загружать Change Detection.
    

## Connections Supports

- [[Angular Signals]] - Сигналы обеспечивают точечную уведомляемость для OnPush и Zoneless архитектуры.
    
- [[Immutability in JS]] - Иммутабельность данных гарантирует корректную работу сравнения ссылок в `@Input`.
    

## Contradicts

- [[Zone.js Directives]] - Идея точечного Zoneless-обновления противоречит глобальному перехвату событий через Zone.js.
    

## Extends

- [[Angular Architecture]] - Расширяет базовые паттерны проектирования высокопроизводительных фронтенд-приложений.
    

## Questions

- Как правильно отлаживать утечки проверок Change Detection через Angular DevTools?
    
- Каково поведение `ChangeDetectorRef.detectChanges()` в сравнении с `markForCheck()` внутри Zoneless-приложений?
    

## Sources

- Официальная документация Angular (Change Detection Strategies & Zoneless)
    
- Angular Signals Guide & RFC
    

#### Review Status: Complete

Last reviewed: 2026-08-23 
Next review: 2026-09-22