---
type: atomic
status: seedling
domain:
created: 2026-09-08
updated: 2026-09-08
tags:
  - 
related: []
---
# Zoneless Change Detection (Angular)

## Core Idea

Механизм обнаружения изменений в Angular, который отказывается от библиотеки `Zone.js` и отслеживания всех асинхронных событий в пользу явных сигналов от реактивных данных (Signals).

## Why It Matters

Убирает накладные расходы на постоянный фоновый обход всего дерева компонентов, делает обновляемость UI точечной и прозрачной, уменьшает размер бандла и существенно повышает производительность приложения.

## Key Points

* **Отказ от `Zone.js`:** Фреймворк больше не перехватывает все асинхронные API браузера (`setTimeout`, `fetch`, `eventListeners`) с помощью Monkey Patching.

* **Статика по умолчанию:** Обычные переменные отрисовываются один раз; если данные не задействуют Сигналы, Angular не тратит ресурсы на их отслеживание.

* **Точечная реактивность:** Изменение сигнала явно сообщает фреймворку, какой именно компонент требует перерисовки, исключая глобальные проверки.

* **Простая отладка:** Стек-трейсы ошибок становятся чистыми и понятными без многослойных прослоек `Zone.js`.

## Examples

### Example 1: Декларативное состояние компонента

Вместо отслеживания обычных полей класса, динамическое состояние оборачивается в `signal()`.

```typescript
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Текущий счет: {{ count() }}</p>
    <button (click)="increment()">+1</button>
  `
})
export class CounterComponent {
  // Статика: отрисуется 1 раз, Angular забывает про неё
  readonly step = 1; 

  // Динамика: при изменении сигнала обновится только этот компонент
  count = signal(0); 

  increment() {
    this.count.update(c => c + this.step);
  }
}
```

### Example 2: Интеграция с внешними асинхронными источниками

При получении данных из внешних API или Web Socket значение записывается в сигнал, явным образом инициируя перерисовку UI.

```typescript
@Component({ ... })
export class UserProfileComponent {
  private userService = inject(UserService);
  
  user = signal<User null |>(null);

  loadData() {
    // Без Zone.js вызываемый fetch не запускает глобальную проверку.
    // Перерисовка произойдет ТОЛЬКО в момент вызова .set()
    fetch('/api/user/1')
      .then(res => res.json())
      .then(data => this.user.set(data));
  }
}
```

## How to Apply

1. **Включить экспериментальный Zoneless-провайдер** в конфигурации приложения:

   ```typescript
   export const appConfig: ApplicationConfig = {
     providers: [
       provideExperimentalZonelessChangeDetection()
     ]
   };
   ```

2. **Удалить или исключить `zone.js`** из `angular.json` (из секции `polyfills`).

3. **Все динамические состояния внутри компонентов** объявлять через `signal()`, `computed()` или переводить из RxJS через `toSignal()`.

## Connections

### Supports

* [[Angular Signals]] — Сигналы служат основным механизмом оповещения о необходимости обновления UI в Zoneless-режиме.

* [[Feature-Sliced Design]] — Облегчает изоляцию слоёв, так как компоненты теряют неявные фоновые зависимости от глобального цикла проверок.

### Contradicts

* [[Zone.js]] — Полностью исключает необходимость использовать monkey patching браузерных асинхронных API.

* [[Default Change Detection Strategy]] — Противоречит концепции регулярного обхода всего дерева компонентов сверху вниз.

### Extends

* [[OnPush Change Detection]] — Доводит идею локального и явного обновления компонентов до абсолюта.

## Questions

* [ ] Как оптимизировать работу старых сторонних библиотек, завязанных на `ChangeDetectorRef.markForCheck()`, при полном переходе на Zoneless?

* [ ] Когда `provideExperimentalZonelessChangeDetection()` перейдет в статус Stable?

## Sources

* Официальная документация Angular: [Angular Zoneless Guide](https://angular.dev/guide/zoneless)

---

**Review Status**: Draft

* Last reviewed: 2026-09-08
* Next review: 2026-10-08
