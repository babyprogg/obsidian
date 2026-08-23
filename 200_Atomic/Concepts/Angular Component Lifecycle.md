---
type: atomic
status: seedling
domain:
created: 2026-08-19
updated: 2026-08-19
tags:
  - 
related: []
---
# Angular Component Lifecycle

## Core Idea
Жизненный цикл компонента в Angular — это строго определенная последовательность этапов (хуков), через которые проходит компонент от момента своего создания до полного удаления из DOM-дерева.

## Why It Matters
Понимание жизненного цикла позволяет правильно организовывать инициализацию данных, безопасно манипулировать DOM-элементами, отслеживать изменения входящих параметров и избегать критических утечек памяти при уничтожении компонента.

Key Points
* **Создание (`constructor` vs `ngOnInit`):** В конструкторе создается экземпляр класса, но `@Input()` свойства еще недоступны; `ngOnInit` вызывается, когда входящие данные уже инициализированы, и служит точкой старта бизнес-логики.
* **Реакция на изменения (`ngOnChanges` & `ngDoCheck`):** `ngOnChanges` отслеживает изменения `@Input()` по ссылкам (`SimpleChanges`), а `ngDoCheck` позволяет перехватывать изменения, которые стандартный Change Detection пропускает.
* **Работа с DOM и дочерними элементами (`ngAfterContentInit/Checked`, `ngAfterViewInit/Checked`):** `ngAfterContentInit` срабатывает после внедрения внешней разметки (`<ng-content>`), а `ngAfterViewInit` — когда полностью готов собственный DOM компонента и сработали `@ViewChild` / `@ViewChildren`.
* **Очистка ресурсов (`ngOnDestroy`):** Финальная стадия, где необходимо отписываться от длительных RxJS-подписок, очищать таймеры и глобальные слушатели событий для предотвращения утечек памяти.

### Examples

``` typescript
Example 1: Загрузка данных пользователя при старте
@Component({ selector: 'app-user-profile', template: `...` })
export class UserProfileComponent implements OnInit, OnDestroy {
  @Input() userId!: string;
  private sub!: Subscription;

  ngOnInit() {
    // Безопасно используем userId для HTTP-запроса
    this.sub = this.userService.getUser(this.userId).subscribe();
  }

  ngOnDestroy() {
    // Гарантированно очищаем подписку перед удалением компонента
    this.sub.unsubscribe();
  }
}

Example 2: Безопасное подключение сторонней библиотеки к DOM
@Component({ selector: 'app-chart', template: `<div #chartRef></div>` })
export class ChartComponent implements AfterViewInit {
  @ViewChild('chartRef') chartRef!: ElementRef;

  ngAfterViewInit() {
    // DOM полностью отрисован, элементы доступны через ViewChild
    new ChartLibrary(this.chartRef.nativeElement);
  }
}
```

## How to Apply
1. **Запросы и подписки:** Делай HTTP-запросы и тяжелую инициализацию в `ngOnInit`, а не в `constructor`.
2. **Защита от утечек:** Каждую подписку, созданную в `ngOnInit`, обязательно завершай в `ngOnDestroy`.
3. **Безопасная работа с DOM:** К `@ViewChild` обращайся строго начиная с `ngAfterViewInit`.
4. **Избегай тяжелой логики в `*Checked` хуках:** Не выполняй ресурсоемкие операции в `ngAfterViewChecked` и `ngDoCheck`, так как они вызываются при каждом микро-событии Change Detection.

## Connections
Supports
* [[Angular Change Detection Mechanics]] — хуки жизненного цикла служат точками входа и контроля в процессе обнаружения изменений.
Contradicts
* [[Imperative DOM Manipulation in Vanilla JS]] — Angular управляет DOM-деревом и состоянием компонентов через декларативный жизненный цикл, запрещая прямое удаление или создание узлов в обход фреймворка.
Extends
* [[Object-Oriented Component Design]] — расширяет концепцию создания и уничтожения объектов интерфейсными контрактами фреймворка.

## Questions
* Как именно вызов `ChangeDetectorRef.detectChanges()` внутри `ngAfterViewInit` влияет на ошибку `ExpressionChangedAfterItHasBeenCheckedError`?
* Каков точный порядок вызова lifecycle-хуков у родительского и дочернего компонентов при их одновременной инициализации?

## Sources
* Angular Official Documentation: Component Lifecycle (`angular.dev`)
* Angular In-Depth: "Everything you need to know about change detection and lifecycle hooks"

## Review Status: Draft

Last reviewed: 2026-08-19
Next review: 2026-09-18