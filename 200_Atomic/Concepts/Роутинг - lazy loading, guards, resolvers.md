---
type: atomic
status: seedling
domain:
created: 2026-08-25
updated: 2026-08-25
tags:
  - 
related: []
---
# Angular Routing Architecture

## Core Idea

**Angular Router управляет навигацией между страницами приложения, а Lazy Loading, Guards и Resolvers разделяют ответственность за загрузку кода, контроль доступа и подготовку данных.**

## Why It Matters

Routing — это не просто переключение компонентов по URL.

В большом Angular-приложении Router отвечает сразу за несколько важных задач:

- определяет, какой компонент показывать;
    
- управляет URL;
    
- решает, какой код загружать;
    
- контролирует доступ пользователя;
    
- может загружать необходимые данные до отображения страницы;
    
- позволяет разделять приложение на независимые feature-разделы.
    

Главная идея — **Separation of Concerns**:

```text
Lazy Loading → когда загружать код?
Guards       → можно ли продолжить навигацию?
Resolvers    → какие данные нужны?
Component    → что показать пользователю?
```

Это позволяет не превращать компоненты в огромные классы, которые одновременно занимаются авторизацией, загрузкой данных, роутингом и UI.

## Key Points

### 1. Lazy Loading

Lazy Loading позволяет не загружать весь код приложения сразу.

Вместо:

```text
Initial Load
    ↓
Всё приложение
    ↓
Огромный bundle
```

получаем:

```text
Initial Load
    ↓
Основной код
    ↓
Пользователь открыл /dashboard
    ↓
Dashboard chunk
    ↓
Пользователь открыл /admin
    ↓
Admin chunk
```

В современном Angular для этого используются:

- `loadComponent` — загрузить отдельный Standalone Component;
    
- `loadChildren` — загрузить набор маршрутов.
    

```typescript
export const APP_ROUTES: Routes = [
  {
    path: 'profile',
    loadComponent: () =>
      import('./features/profile/profile.component')
        .then(m => m.ProfileComponent)
  },

  {
    path: 'dashboard',
    loadChildren: () =>
      import('./features/dashboard/dashboard.routes')
        .then(m => m.DASHBOARD_ROUTES)
  }
];
```

### 2. Preloading Strategies

После первоначальной загрузки приложения Angular может начать загружать lazy routes заранее.

Основные варианты:

- `NoPreloading` — загружать только при переходе;
    
- `PreloadAllModules` — загрузить все lazy routes после initial load;
    
- Custom Strategy — самостоятельно определить, что и когда предзагружать.
    

```typescript
provideRouter(
  APP_ROUTES,
  withPreloading(PreloadAllModules)
);
```

### 3. Guards

Guards контролируют навигацию.

Они отвечают на вопрос:

> **«Можно ли пользователю выполнить этот переход?»**

Основные типы:

|Guard|Ответственность|
|---|---|
|`CanActivate`|Можно ли активировать маршрут?|
|`CanActivateChild`|Можно ли открыть дочерний маршрут?|
|`CanDeactivate`|Можно ли покинуть текущий маршрут?|
|`CanMatch`|Подходит ли вообще этот маршрут?|

Например:

```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(
    ['/login'],
    {
      queryParams: {
        returnUrl: state.url
      }
    }
  );
};
```

### 4. `CanMatch` + Lazy Loading

`CanMatch` особенно полезен для lazy-loaded routes.

Условно:

```text
User
 ↓
CanMatch
 ↓
Есть доступ?
 ↙       ↘
Да       Нет
 ↓         ↓
Load      Route
chunk     ignored
 ↓
Admin
```

То есть можно принять решение о том, использовать ли маршрут, ещё до нормальной активации lazy route.

### 5. Functional Guards

Современный Angular предпочитает functional guards.

Вместо отдельного класса:

```typescript
@Injectable()
class AuthGuard {
  ...
}
```

можно написать:

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);

  return auth.isAuthenticated();
};
```

Это уменьшает boilerplate и хорошо сочетается с современным Angular API.

### 6. Resolvers

Resolver подготавливает данные **до активации маршрута**.

Без resolver:

```text
Route
 ↓
Component
 ↓
Loading
 ↓
HTTP request
 ↓
Data
 ↓
UI update
```

С resolver:

```text
Route
 ↓
Resolver
 ↓
HTTP request
 ↓
Data
 ↓
Component
 ↓
UI
```

Пример:

```typescript
export const userResolver: ResolveFn<User> = (route) => {
  const userId = route.paramMap.get('id')!;

  const userService = inject(UserService);

  return userService.getUserById(userId);
};
```

Подключение:

```typescript
{
  path: 'users/:id',

  loadComponent: () =>
    import('./user-detail.component')
      .then(m => m.UserDetailComponent),

  resolve: {
    user: userResolver
  }
}
```

### 7. Component Input Binding

Вместо прямого обращения к `ActivatedRoute` можно передавать route data прямо в component input.

Включаем:

```typescript
provideRouter(
  APP_ROUTES,
  withComponentInputBinding()
);
```

Компонент:

```typescript
export class UserDetailComponent {
  readonly user = input.required<User>();
}
```

При этом:

```typescript
resolve: {
  user: userResolver
}
```

связывается с:

```typescript
user = input.required<User>();
```

Получается:

```text
Resolver
   ↓
route data
   ↓
Component Input
   ↓
user()
```

### 8. Navigation Lifecycle

Навигация проходит примерно через такую цепочку:

```text
Navigation Start
       ↓
URL Matching & Redirects
       ↓
CanMatch
       ↓
Load Async Routes
       ↓
CanDeactivate
       ↓
CanActivateChild
       ↓
CanActivate
       ↓
Resolvers
       ↓
Activate Components
       ↓
Navigation End
```

Каждый этап решает свою задачу.

## Examples

### Example 1

### Защищённая Admin Panel

Представим приложение:

```text
/
├── dashboard
├── profile
└── admin
    ├── users
    ├── roles
    └── settings
```

Admin-раздел не должен быть доступен обычному пользователю.

Можно построить routing так:

```text
/admin
   ↓
CanMatch
   ↓
Пользователь admin?
 ↙          ↘
Да          Нет
 ↓            ↓
Lazy Load    Ignore route
 ↓
Admin Routes
```

Здесь:

- `CanMatch` контролирует доступ;
    
- `loadChildren` обеспечивает Lazy Loading;
    
- admin-код не является частью initial bundle.
    

### Example 2

### User Details Page

Есть маршрут:

```text
/users/:id
```

Страница должна показать информацию о пользователе.

Вместо:

```text
Component
 ↓
получить id
 ↓
HTTP request
 ↓
Loading
 ↓
User
```

можно использовать resolver:

```text
/users/:id
      ↓
userResolver
      ↓
GET /users/:id
      ↓
User
      ↓
UserDetailComponent
```

Компонент получает уже подготовленный объект:

```typescript
readonly user = input.required<User>();
```

Это особенно удобно, когда данные необходимы самому существованию страницы.

## How to Apply

### 1. Разделяй ответственность

При проектировании маршрута сначала спроси:

```text
Мне нужно...

Загрузить код позже?
→ Lazy Loading

Проверить доступ?
→ Guard

Получить обязательные данные?
→ Resolver

Просто отобразить страницу?
→ Component
```

### 2. Используй Lazy Loading для feature-разделов

Например:

```text
features/
├── dashboard/
├── users/
├── reports/
└── admin/
```

Каждый крупный feature может иметь собственный route configuration.

```typescript
{
  path: 'reports',
  loadChildren: () =>
    import('./features/reports/reports.routes')
      .then(m => m.REPORTS_ROUTES)
}
```

### 3. Защищай маршруты Guards

Для authentication:

```typescript
{
  path: 'dashboard',
  canActivate: [authGuard],
  loadComponent: ...
}
```

Для lazy routes можно рассмотреть `CanMatch`.

### 4. Используй Resolver только для действительно необходимых данных

Хороший кандидат:

```text
/users/:id
```

если без пользователя страница вообще не имеет смысла.

Плохой кандидат:

```text
/dashboard
```

если dashboard может нормально показать skeleton и постепенно загрузить несколько независимых блоков.

### 5. Для современного Angular используй Functional API

Предпочтительные инструменты:

```text
Functional Guards
Functional Resolvers
Standalone Components
loadComponent
loadChildren
withComponentInputBinding
```

## Connections

### Supports

- [[Angular Standalone Components]] — routing напрямую поддерживает lazy loading Standalone Components через `loadComponent`.
    
- [[Angular Dependency Injection]] — Guards и Resolvers используют `inject()` для получения зависимостей.
    
- [[RxJS]] — HTTP-запросы в Resolver обычно возвращают `Observable`.
    
- [[Angular Signals]] — route data может преобразовываться в Signals или передаваться через `input()`.
    

### Contradicts

- [[God Component]] — routing architecture помогает не помещать navigation, authorization и data fetching непосредственно в компонент.
    
- [[Eager Loading Everything]] — Lazy Loading противопоставляется загрузке всего приложения сразу.
    

### Extends

- [[Angular Routing]] — эта заметка расширяет базовое понимание Angular Router архитектурными паттернами.
    
- [[Angular Lazy Loading]] — подробнее рассматривает загрузку feature-разделов.
    
- [[Angular Guards]] — подробнее рассматривает контроль доступа.
    
- [[Angular Resolvers]] — подробнее рассматривает получение данных до активации маршрута.
    

## Questions

- В каком порядке Angular реально выполняет `CanMatch`, `CanDeactivate` и `CanActivate` в сложном nested routing?
    
- Когда `CanMatch` действительно предотвращает загрузку lazy chunk?
    
- Как Angular обрабатывает несколько Resolvers одновременно?
    
- Что происходит, если один Resolver возвращает ошибку?
    
- Когда Resolver лучше заменить на обычный HTTP-запрос внутри компонента?
    
- Как работает `runGuardsAndResolvers`?
    
- Как работают `RouteReuseStrategy` и кеширование компонентов?
    
- Как SSR и hydration влияют на Angular Router?
    
- Как лучше организовать routing в большом Nx workspace?
    

## Sources

- Angular Router documentation
    
- Angular Lazy Loading documentation
    
- Angular Route Guards documentation
    
- Angular Route Resolvers documentation
    
- Angular Standalone Components documentation
    

---

**Review Status**:

- Last reviewed: 2026-08-25
    
- Next review: 2026-09-24