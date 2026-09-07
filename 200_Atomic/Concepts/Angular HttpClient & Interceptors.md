---
type: atomic
status: seedling
domain:
created: 2026-08-26
updated: 2026-08-26
tags:
  - 
related: []
---
# Angular HttpClient & Interceptors

## Core Idea

**`HttpClient` — встроенный HTTP-клиент Angular на базе RxJS Observables, а Interceptors — промежуточный слой, который позволяет централизованно обрабатывать исходящие запросы и входящие ответы.**

## Why It Matters

Без Interceptors сетевую логику легко начать дублировать во всех сервисах:

```text
UserService
   ├── добавляет JWT
   ├── обрабатывает 401
   └── логирует запрос

OrderService
   ├── добавляет JWT
   ├── обрабатывает 401
   └── логирует запрос

ProductService
   ├── добавляет JWT
   ├── обрабатывает 401
   └── логирует запрос
```

Interceptors позволяют вынести общую HTTP-логику в одно место:

```text
                    HttpClient
                       ↓
              ┌─────────────────┐
              │  Interceptors   │
              ├─────────────────┤
              │ Base URL        │
              │ Authentication  │
              │ Logging         │
              │ Error handling  │
              └─────────────────┘
                       ↓
                     API
```

Понимание RxJS-природы `HttpClient` также важно для:

- отмены запросов;
    
- предотвращения memory leaks;
    
- управления конкурентными запросами;
    
- предотвращения race conditions;
    
- retry и error handling.
    

## Key Points

### 1. HttpClient построен на RxJS

HTTP-запросы Angular возвращают `Observable`.

```typescript
const users$ = http.get<User[]>('/api/users');
```

Важный момент:

> `Observable` от `HttpClient` является **cold / ленивым** — запрос не выполняется до подписки.

```typescript
const users$ = http.get<User[]>('/api/users');

// HTTP-запрос ещё не отправлен

users$.subscribe(users => {
  // Здесь запрос выполняется
});
```

Благодаря RxJS запросы можно комбинировать с операторами вроде:

```text
switchMap
catchError
retry
debounceTime
takeUntil
```

---

### 2. HttpRequest иммутабелен

`HttpRequest` нельзя изменять напрямую.

Нельзя делать условно:

```typescript
req.headers = ...
```

Вместо этого используется:

```typescript
req.clone(...)
```

Например:

```typescript
const authReq = req.clone({
  headers: req.headers.set(
    'Authorization',
    `Bearer ${token}`
  )
});
```

Ментальная модель:

```text
Original Request
      ↓
   clone()
      ↓
Modified Request
```

Оригинальный request остаётся неизменным.

---

### 3. Interceptors используют Chain of Responsibility

Interceptors образуют цепочку.

Например:

```text
Request
   ↓
BaseUrlInterceptor
   ↓
AuthInterceptor
   ↓
LoggingInterceptor
   ↓
ErrorInterceptor
   ↓
API
```

Для исходящего **Request** они проходят в порядке объявления.

Для входящего **Response** цепочка проходит обратно:

```text
API
 ↓
ErrorInterceptor
 ↓
LoggingInterceptor
 ↓
AuthInterceptor
 ↓
BaseUrlInterceptor
 ↓
Application
```

Это важно при проектировании порядка Interceptors.

---

### 4. Functional Interceptors

В современном Angular предпочтительнее использовать функциональные интерцепторы:

```typescript
HttpInterceptorFn
```

Они хорошо сочетаются с:

```typescript
inject()
```

и регистрируются через:

```typescript
provideHttpClient(
  withInterceptors([...])
)
```

Пример:

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    const authReq = req.clone({
      headers: req.headers.set(
        'Authorization',
        `Bearer ${token}`
      )
    });

    return next(authReq);
  }

  return next(req);
};
```

---

### 5. `HttpContext` позволяет управлять поведением конкретного запроса

Не каждый HTTP-запрос должен проходить через одинаковую обработку.

Например, глобальный Loader может быть нужен почти везде, кроме background-запроса.

Для этого можно использовать `HttpContextToken`.

Ментальная модель:

```text
Global Interceptor
       ↓
"Нужно ли обрабатывать этот request?"
       ↓
HttpContext
    ↙       ↘
  true      false
   ↓          ↓
Process     Skip
```

Это позволяет задавать поведение **на уровне конкретного HTTP-запроса**, не создавая отдельные сервисы или обходные пути.

---

### 6. Interceptor не должен превращаться в God Object

Interceptor хорошо подходит для **cross-cutting concerns**:

- authentication;
    
- logging;
    
- error handling;
    
- loading indicators;
    
- retry;
    
- headers;
    
- базовая трансформация request/response.
    

Но бизнес-логику конкретной фичи лучше оставлять в соответствующем сервисе.

Плохо:

```text
AuthInterceptor
 ├── JWT
 ├── Users business logic
 ├── Orders business logic
 ├── Payments business logic
 └── Analytics business logic
```

Хорошо:

```text
AuthInterceptor
 └── Authentication

UserService
 └── Users business logic

OrderService
 └── Orders business logic
```

## Examples

### Example 1

### Автоматическая подстановка JWT

Допустим, каждый защищённый API endpoint требует:

```http
Authorization: Bearer <token>
```

Без interceptor пришлось бы писать это в каждом сервисе:

```typescript
http.get('/users', {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

С interceptor:

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (!token) {
    return next(req);
  }

  const authReq = req.clone({
    headers: req.headers.set(
      'Authorization',
      `Bearer ${token}`
    )
  });

  return next(authReq);
};
```

Теперь любой запрос автоматически получает токен:

```text
http.get('/users')
       ↓
AuthInterceptor
       ↓
Authorization header
       ↓
GET /users
```

---

### Example 2

### Глобальная обработка ошибок и Token Refresh

Предположим, access token истёк.

API отвечает:

```text
401 Unauthorized
```

Interceptor может попытаться обновить токен и повторить оригинальный запрос.

```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {

      if (error.status === 401) {
        return authService.refreshToken().pipe(
          switchMap(() => {

            const retryReq = req.clone({
              headers: req.headers.set(
                'Authorization',
                `Bearer ${authService.getToken()}`
              )
            });

            return next(retryReq);
          })
        );
      }

      return throwError(() => error);
    })
  );
};
```

Логика:

```text
Request
   ↓
API
   ↓
401
   ↓
Interceptor
   ↓
refreshToken()
   ↓
New token
   ↓
Retry original request
   ↓
API
   ↓
Response
```

> ⚠️ В реальном приложении token refresh требует дополнительной защиты от ситуации, когда одновременно несколько запросов получили `401`. Иначе можно случайно запустить несколько refresh-запросов одновременно.

---

### Example 3

### Отключение глобального Loader

Допустим, есть interceptor:

```text
Request
   ↓
LoadingInterceptor
   ↓
showLoader()
   ↓
API
```

Но для background autosave loader показывать не хочется.

Можно передать специальный `HttpContext`:

```typescript
const request = http.post(
  '/api/autosave',
  data,
  {
    context: new HttpContext().set(
      SKIP_LOADER,
      true
    )
  }
);
```

Interceptor проверяет:

```typescript
if (req.context.get(SKIP_LOADER)) {
  return next(req);
}
```

Получается:

```text
Обычный request
     ↓
Loader

Autosave request
     ↓
HttpContext
     ↓
Skip Loader
```

## How to Apply

### 1. Зарегистрировать HttpClient

В `app.config.ts`:

```typescript
provideHttpClient(
  withInterceptors([
    baseUrlInterceptor,
    authInterceptor,
    loggingInterceptor,
    errorInterceptor
  ])
)
```

---

### 2. Продумать порядок Interceptors

Например:

```text
BaseUrlInterceptor
        ↓
AuthInterceptor
        ↓
LoggingInterceptor
        ↓
ErrorInterceptor
        ↓
API
```

Порядок важен, потому что Interceptors образуют цепочку.

---

### 3. Использовать `HttpContext` для исключений

Если конкретному request нужно изменить глобальное поведение, не создавай отдельную копию interceptor.

Используй:

```typescript
HttpContextToken
```

Например:

```text
SKIP_AUTH
SKIP_LOADER
SKIP_ERROR_HANDLER
SKIP_CACHE
```

---

### 4. Возвращать Observable из сервисов

Сервис обычно должен возвращать Observable, а не подписываться внутри себя.

Хорошо:

```typescript
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

Компонент:

```typescript
users$ = this.userService.getUsers();
```

или:

```typescript
this.userService.getUsers()
  .subscribe(users => {
    // ...
  });
```

Плохо:

```typescript
getUsers() {
  this.http.get<User[]>('/api/users')
    .subscribe(users => {
      // ...
    });
}
```

Почему?

Потому что сервис, который сам делает `subscribe()`, забирает у вызывающего кода контроль над Observable.

---

### 5. Использовать явную типизацию

Вместо:

```typescript
this.http.get(url);
```

предпочтительно:

```typescript
this.http.get<User[]>(url);
```

Так TypeScript знает форму данных, которые возвращает API.

## Connections

### Supports

- [[Angular Architecture]] — помогает отделить HTTP/network layer от UI и business logic.
    
- [[RxJS Design Patterns]] — `HttpClient` использует Observables и реактивные операторы для управления потоками данных.
    
- [[Angular Dependency Injection]] — Functional Interceptors получают зависимости через `inject()`.
    
- [[TypeScript Generics]] — generic-типы вроде `http.get<User[]>()` обеспечивают типизацию API responses.
    

### Contradicts

- [[Promise-based HTTP Clients]] — `HttpClient` использует Observable-based подход вместо модели, построенной вокруг `Promise` и `async/await`.
    
- [[Service-level HTTP duplication]] — Interceptors позволяют вынести повторяющуюся HTTP-логику из отдельных сервисов.
    

### Extends

- [[Middleware Pattern]] — Interceptors реализуют идею middleware.
    
- [[Chain of Responsibility]] — каждый interceptor передаёт request следующему элементу цепочки.
    
- [[Angular Services]] — `HttpClient` обычно используется внутри сервисов для взаимодействия с API.
    

## Questions

- Как корректно предотвращать несколько одновременных `refreshToken()` при множественных `401`?
    
- Как лучше реализовывать отмену дублирующихся HTTP-запросов?
    
- Когда использовать `switchMap`, `exhaustMap` и `concatMap` для HTTP-запросов?
    
- Как лучше интегрировать `HttpClient` с Angular Signals через `toSignal()`?
    
- Как правильно реализовать глобальный retry для HTTP-запросов?
    
- Как организовать interceptor для caching?
    
- Как обрабатывать ошибки refresh token и предотвращать бесконечный цикл `401 → refresh → 401`?
    

## Sources

- Angular Official Docs — `HttpClient`
    
- Angular Official Docs — `HttpInterceptorFn`
    
- Angular Official Docs — `HttpContext` / `HttpContextToken`
    
- RxJS Documentation — Observable streams
    
- RxJS Documentation — Stream transformation and error handling
    

---

**Review Status**: Draft

- Last reviewed: 2026-08-26
    
- Next review: 2026-09-25