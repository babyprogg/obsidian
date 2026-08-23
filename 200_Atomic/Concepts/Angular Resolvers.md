---
type: atomic
status: seedling
domain:
created: 2026-06-11
updated: 2026-06-11
tags:
  - angular
related: []
---
---
tags:
  - angular
  - routing
  - architecture
  - frontend
aliases:
  - Резолверы в Angular
  - Функциональные резолверы
date: 2026-06-11
---

# Angular Resolvers (Резолверы данных)

**Angular Resolver** — это специализированный инструмент маршрутизатора (Router), который блокирует активацию компонента до тех пор, пока не загрузятся связанные с ним данные.

> [!SUCCESS] Суть в одно предложение
> Резолвер переносит загрузку данных из этапа инициализации компонента (`ngOnInit`) на этап **маршрутизации** (между кликом по ссылке и рендером страницы).

---

## 🛠️ Базовый код (Functional Resolver)

Начиная с Angular 15+, резолверы пишутся в виде чистых функций с использованием `inject()`.

### 1. Создание (product.resolver.ts)
```typescript
import { inject } from '@angular/core';
import { ResolveFn, ActivatedRouteSnapshot } from '@angular/router';
import { ProductService } from './product.service';
import { Product } from './product.model';

export const productResolver: ResolveFn<Product> = (route: ActivatedRouteSnapshot) => {
  const productService = inject(ProductService);
  const productId = route.paramMap.get('id')!;
  
  // Роутер сам подпишется на Observable, возьмет 1-е значение и сделает complete
  return productService.getProductById(productId);
};
```
## 2. Регистрация в роутинге (app.routes.ts)

   ``` typescript
   export class ProductDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);
  product!: Product;

  ngOnInit() {
    // snapshot подходит, если ID в URL не меняется внутри этого же компонента
    this.product = this.route.snapshot.data['productData'];
    
    // Если ID может меняться реактивно без пересоздания компонента:
    // this.route.data.subscribe(data => this.product = data['productData']);
  }
}
   ```
   ## Когда использовать? (UX & Architecture Decision)

Резолвер — это не серебряная пуля. Его использование — это всегда компромисс между сложностью кода и User Experience (UX).

### Идеальные кейсы (Data-driven Routing)

- **Критические данные:** Страницы редактирования (`/users/42/edit`), где без объекта формы рендерить вообще нечего.
    
- **Защита от 404:** Если в резолвере бэкенд отдал ошибку, можно сразу внутри функции вызвать `router.navigate(['/404'])`. Компонент страницы даже не начнет инициализироваться.
    
- **Тонкий триггер для Store:** В Enterprise-архитектурах резолвер часто возвращает `true`, но внутри делает `store.dispatch(Action)`, чтобы прогреть кэш перед входом.
    

### Когда НЕЛЬЗЯ использовать (Противопоказания)

- **Тяжелые запросы / Слабый интернет:** Если запрос идет 3 секунды, интерфейс «зависнет» на старой странице. Пользователю покажется, что сайт сломался.
    
- **Компоненты с независимыми блоками:** На Дашборде с 5 графиками резолвер заставит ждать загрузки _всех_ графиков. Лучше зайти на страницу сразу и показать Loader/Skeleton для каждого блока отдельно.
    

## Важные нюансы для памяти

1. **Одноразовость потока:** Роутер ждет от `Observable` статус `complete`. Если твой сервис возвращает бесконечный поток (например, WebSocket или горячий BehaviorSubject), резолвер зависнет навсегда. Используй оператор `take(1)` внутри резолвера, если сервис не завершается сам.
    
2. **Глобальный лоадер:** Чтобы избежать эффекта «зависания» интерфейса, всегда вешай спиннер на глобальные события роутера (`NavigationStart` / `NavigationEnd`).
### См. также (Связанные заметки)

- [[Angular Routing Lifecycle]]
    
- [[State Management: NgRx Router Store]]
    
- [[UX Loading Patterns: Skeletons vs Resolvers]]