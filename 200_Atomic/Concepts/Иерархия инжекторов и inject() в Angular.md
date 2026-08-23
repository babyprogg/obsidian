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
# Иерархия инжекторов и inject() в Angular

## Core Idea
Внедрение зависимостей (DI) в Angular — это иерархическая система поиска и предоставления экземпляров сервисов по дереву компонентов, реализуемая сегодня через гибкую функцию `inject()`.

## Why It Matters
Понимание иерархии инжекторов и `inject()` позволяет четко разделять ответственность (UI, бизнес-логика, данные), управлять жизненным циклом сервисов, избегать утечек памяти и легко переиспользовать код без громоздкого наследования через конструктор.

## Key Points
- **Две иерархии инжекторов**:
  - `EnvironmentInjector`: Отвечает за глобальные синглтоны (`providedIn: 'root'`) и ленивые модули/маршруты.
  - `ElementInjector`: Создается на уровне компонента или директивы через массив `providers: [...]`.
- **Алгоритм поиска**:
  - Angular ищет сервис снизу вверх: сначала в `ElementInjector` самого компонента, затем поднимается по родительским компонентам, а после переходит в `EnvironmentInjector` (до самого `Root Injector`).
  - Если сервис не найден нигде — выбрасывается `NullInjectorError`.
- **Функция `inject()`**:
  - Заменяет внедрение через конструктор (`constructor(private svc: MyService)`).
  - Работает строго в **Injection Context** (объявление свойств класса, `constructor`, функции вроде `runInInjectionContext`, функциональные `guards` и `interceptors`).
  - Убирает необходимость передавать сервисы через `super()` при наследовании классов.

## Examples

### Example 1: Финансовый компонент (Изоляция фичи)
Использование `inject()` для получения сервиса данных и сервиса расчетов внутри компонента.

```typescript
@Component({
  selector: 'app-revenue-calculator',
  standalone: true,
  providers: [FinanceCalculatorService], // Локальный экземпляр в ElementInjector
  template: `<div>Чистый доход: {{ netRevenue() }}</div>`
})
export class RevenueCalculatorComponent {
  // Внедрение через функциональный контекст
  private dataService = inject(IncomeDataService);
  private calcService = inject(FinanceCalculatorService);

  readonly netRevenue = computed(() => {
    const rawData = this.dataService.getTransactions();
    return this.calcService.calculateNetProfit(rawData, { taxRate: 0.12 });
  });
}

```
### Example 2: Кастомная утилита (Композиция вместо наследования)

Создание переиспользуемой функции с `inject()` вне классов компонентов.
``` ts
// Вспомогательная функция с использованием DI
export function injectParam(paramKey: string) {
  const route = inject(ActivatedRoute);
  return toSignal(
    route.paramMap.pipe(map(params => params.get(paramKey)))
  );
}

// Использование в любом компоненте без конструктора
@Component({ ... })
export class UserProfileComponent {
  readonly userId = injectParam('id');
}
```
## How to Apply

1. **По умолчанию — Root**: Регистрируй общие сервисы без состояния через `@Injectable({ providedIn: 'root' })`.
    
2. **Локализуй контекст**: Добавляй сервисы в `providers: [...]` компонента только тогда, когда нужен отдельный экземпляр, привязанный к жизненному циклу этого UI-узла.
    
3. **Используй `inject()` для свойств**: Пиши `private service = inject(MyService);` при объявлении полей класса, чтобы не раздувать конструктор.
    
4. **Применяй в функциональном коде**: Используй `inject()` внутри функциональных гардов, интерцепторов и утилит вместо создания лишних классов-оберток.
## Connections

### Supports

- [[Разделение обязанностей]] — Оставляет компоненты "тонкими", делегируя расчеты и запросы в сервисы.
    
- [[Feature-Sliced Design]] — Помогает изолировать сервисы в рамках конкретных слоев и слайсов.
    

### Contradicts

- [[Жесткая связанность]] — Предотвращает прямое создание экземпляров через `new Service()` внутри компонентов.
    

### Extends

- [[Паттерн Dependency Injection]] — Дополняет классический ООП DI функциональными примитивами и иерархическим резолвингом Angular.
    

## Questions

- Как устроена работа `runInInjectionContext` "под капотом" при вызове DI вне момента инициализации класса?
    
- Как глубина дерева `ElementInjector` влияет на перформанс в очень крупных приложениях?
    

## Sources

- Официальная документация Angular: Dependency Injection System
    
- Angular `inject()` API Reference
    

Review Status: #unreviewed Last reviewed: 2026-08-23 Next review: 2026-09-22