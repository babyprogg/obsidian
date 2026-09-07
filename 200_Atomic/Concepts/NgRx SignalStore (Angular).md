---
type: atomic
status: seedling
domain:
created: 2026-09-04
updated: 2026-09-04
tags:
  - 
related: []
---
# NgRx SignalStore (Angular)

## Core Idea

`signalStore` — это реактивный и декларативный менеджер состояния в Angular, собираемый из функциональных блоков (`withState`, `withComputed`, `withMethods`) на базе Angular Signals.

## Why It Matters

Он убирает избыточный код (boilerplate) традиционного Redux/NgRx, полностью избавляет от необходимости писать RxJS для прямого изменения состояния и обеспечивает простую, лёгкую и строго типизированную работу со стейтом на уровне компонентов и всего приложения.

## Key Points

- **`withState`**: Объявляет базовую структуру состояния и создаёт реактивные сигналы только для чтения (`Readonly Signals`).
    
- **`withComputed`**: Вычисляет производное состояние (`derived state`) на основе базовых сигналов с автоматическим кешированием и ленивым пересчётом.
    
- **`withMethods`**: Предоставляет публичные методы для модификации состояния через иммутабельный `patchState` или выполнения асинхронных операций через `rxMethod`.
    
- **Композиция (`signalStoreFeature`)**: Позволяет переиспользовать логику, например пагинацию и статусы загрузки, между разными сторами как обычные модули.
    
- **Иммутабельность**: `patchState` под капотом создаёт новые объекты состояния, гарантируя предсказуемость реактивных потоков.
    

## Examples

### Example 1: Локальное состояние с вычислениями (Корзина)

```typescript
export const CartStore = signalStore(
  { providedIn: 'root' },
  withState({ items: [] as CartItem[], promo: null as string | null }),
  withComputed(({ items, promo }) => ({
    totalPrice: computed(() => {
      const sum = items().reduce((acc, i) => acc + i.price * i.quantity, 0);
      return promo() === 'SALE10' ? sum * 0.9 : sum;
    })
  })),
  withMethods((store) => ({
    addItem(item: CartItem) {
      patchState(store, (s) => ({ items: [...s.items, item] }));
    }
  }))
);
```

### Example 2: Асинхронные HTTP-запросы и фильтрация (Каталог)

```typescript
export const ProductsStore = signalStore(
  withState({ products: [] as Product[], query: '', loading: false }),
  withComputed(({ products, query }) => ({
    filtered: computed(() => products().filter(p => p.name.includes(query())))
  })),
  withMethods((store, api = inject(ProductsApi)) => ({
    setQuery(query: string) {
      patchState(store, { query });
    },
    loadProducts: rxMethod<void>(
      pipe(
        tap(() => patchState(store, { loading: true })),
        switchMap(() => api.getAll()),
        tap((products) => patchState(store, { products, loading: false }))
      )
    )
  }))
);
```

## How to Apply

1. **Выделите базовые данные** и передайте их в `withState(...)`.
    
2. **Определите зависимые (производные) значения** — фильтры, счётчики и т. д. — и опишите их в `withComputed(...)`.
    
3. **Напишите методы управления** в `withMethods(...)`, меняя стейт через `patchState(store, ...)`.
    
4. **Внедрите стор** в компонент (`providers: [ProductsStore]`) или зарегистрируйте глобально (`{ providedIn: 'root' }`).
    
5. **Вызывайте сигналы в HTML-шаблоне** напрямую: `store.filtered()` или `store.totalPrice()`.
    

## Connections

### Supports

- [[Angular Signals]] — выступает высокоуровневым архитектурным фреймворком над примитивами сигналов (`signal`, `computed`).
    
- [[Component-Driven Architecture]] — изолирует логику состояния внутри компонента или фичи.
    

### Contradicts

- [[Classic NgRx / Redux Boilerplate]] — отказывается от глобального дерева состояния, громоздких `Actions`, `Reducers` и пайплайнов `Effects`.
    
- [[Mutable State Management]] — запрещает прямую мутацию объектов, изменения выполняются только через иммутабельный `patchState`.
    

### Extends

- [[RxJS in Angular]] — заменяет `BehaviorSubject` в управлении состоянием, оставляя RxJS для управления потоками событий и HTTP через `rxMethod`.
    

## Questions

- Как правильно организовывать сквозное тестирование (E2E и Unit) для `signalStore`?
    
- Каковы лучшие практики обработки ошибок внутри `rxMethod` без разрыва потока?
    

## Sources

- Официальная документация: `@ngrx/signals` (ngrx.io)
    

---

**Review Status**: Done

- Last reviewed: 2026-09-04
    
- Next review: 2026-10-04