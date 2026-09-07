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
# Angular Environments & Configuration

## Core Idea

**Работа с окружениями в Angular позволяет использовать один и тот же код приложения с разными параметрами конфигурации для Dev, Test и Prod.**

Главное различие:

```text
Compile-time
    ↓
Конфигурация выбирается во время сборки

Runtime
    ↓
Конфигурация загружается после сборки,
при запуске приложения
```

## Why It Matters

Конфигурация должна быть отделена от бизнес-логики приложения.

Вместо:

```typescript
this.http.get(
  'https://production-api.example.com/users'
);
```

лучше:

```typescript
this.http.get(
  `${environment.apiUrl}/users`
);
```

Тогда один и тот же код можно запускать в разных окружениях:

```text
Development
apiUrl → dev-api.example.com

Test
apiUrl → test-api.example.com

Production
apiUrl → api.example.com
```

Это помогает:

- избежать hardcoded URL;
    
- уменьшить риск использования неправильного API;
    
- автоматизировать CI/CD;
    
- разделить Dev / Test / Prod;
    
- менять конфигурацию независимо от бизнес-логики.
    

## Key Points

### 1. Один код — разные параметры

Компоненты и сервисы не должны знать, в каком окружении они работают.

Вместо этого они используют объект конфигурации:

```typescript
environment.apiUrl
```

Например:

```typescript
export const environment = {
  apiUrl: 'https://api.example.com'
};
```

Сервис:

```typescript
@Injectable({
  providedIn: 'root'
})
export class UserService {

  private readonly apiUrl = environment.apiUrl;

  getUsers() {
    return this.http.get<User[]>(
      `${this.apiUrl}/users`
    );
  }
}
```

Сам `UserService` не должен знать, Dev это, Test или Production.

---

### 2. Compile-time Configuration

Это стандартный Angular-подход с environment files.

Например:

```text
src/environments/
├── environment.ts
└── environment.development.ts
```

Каждый файл содержит одинаковую структуру, но разные значения:

```typescript
// environment.ts

export const environment = {
  apiUrl: 'https://api.example.com'
};
```

```typescript
// environment.development.ts

export const environment = {
  apiUrl: 'https://dev-api.example.com'
};
```

Angular может заменить один файл другим во время build через `fileReplacements`.

Условно:

```text
ng build
    ↓
Angular CLI
    ↓
fileReplacements
    ↓
environment.ts
    ↓
Production bundle
```

То есть конфигурация определяется **до того, как приложение попадёт в браузер**.

---

### 3. Runtime Configuration

Другой подход — не зашивать конфигурацию в JavaScript bundle.

Вместо этого приложение получает конфигурацию при запуске:

```text
Docker Image
     ↓
Один и тот же build
     ↓
Deploy
   ↙      ↘
 Test     Prod
   ↓        ↓
config.json
   ↓        ↓
Test API  Prod API
```

Например:

```json
{
  "apiUrl": "https://api.example.com",
  "features": {
    "newDashboard": true
  }
}
```

Приложение загружает этот файл во время старта.

Это позволяет реализовать принцип:

> **Build once, deploy anywhere.**

Один Docker image можно использовать для разных окружений, меняя только runtime configuration.

---

### 4. `provideAppInitializer`

Для загрузки runtime configuration до запуска приложения можно использовать `provideAppInitializer()`.

Условная схема:

```text
Application Start
       ↓
App Initializer
       ↓
GET /assets/config.json
       ↓
Configuration loaded
       ↓
Angular Application
       ↓
Components
```

Например:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideAppInitializer(() => {
      const configService = inject(ConfigService);

      return configService.load();
    })
  ]
};
```

После этого приложение может использовать загруженную конфигурацию.

---

### 5. Compile-time vs Runtime

Главное различие:

||Compile-time|Runtime|
|---|---|---|
|Когда определяется config|При build|При запуске|
|Нужно пересобирать приложение?|Да|Нет|
|Один Docker image для Dev/Test/Prod|Неудобно|Удобно|
|Стандартный Angular подход|Да|Дополнительная архитектура|
|Можно менять config после build|Нет|Да|

Ментальная модель:

```text
COMPILE-TIME

Source Code
     ↓
Angular Build
     ↓
Environment
     ↓
JS Bundle
     ↓
Browser
```

против:

```text
RUNTIME

Source Code
     ↓
Angular Build
     ↓
Same JS Bundle
     ↓
Browser
     ↓
config.json
     ↓
Application
```

---

### 6. Frontend Configuration ≠ Secrets

Очень важный момент:

> **Любая конфигурация, которая попадает во frontend, потенциально доступна пользователю.**

Например:

```typescript
export const environment = {
  apiUrl: 'https://api.example.com',
  apiKey: 'some-key'
};
```

После build значение может оказаться в JS bundle.

Пользователь может открыть:

```text
DevTools
   ↓
Network
   ↓
Sources
   ↓
JS bundle
```

Поэтому нельзя хранить на frontend:

- database passwords;
    
- private API keys;
    
- secret tokens;
    
- credentials;
    
- private encryption keys.
    

Frontend configuration — это **configuration, а не secret storage**.

## Examples

### Example 1

### Compile-time Environment Replacement

Есть:

```text
environment.ts
environment.development.ts
```

Development:

```typescript
export const environment = {
  apiUrl: 'https://dev-api.example.com'
};
```

Production:

```typescript
export const environment = {
  apiUrl: 'https://api.example.com'
};
```

При production build Angular заменяет development configuration на production configuration согласно настройкам `angular.json`.

Получается:

```text
ng build --configuration=production
             ↓
       fileReplacements
             ↓
     production config
             ↓
       JS bundle
```

Приложение при этом не содержит условие:

```typescript
if (environment === 'production') {
  ...
}
```

Оно просто получает нужную конфигурацию.

---

### Example 2

### Build Once, Deploy Anywhere

Допустим, CI/CD создаёт Docker image:

```text
angular-app:1.0.0
```

Этот image используется и для Test, и для Production.

Test:

```json
{
  "apiUrl": "https://test-api.example.com"
}
```

Production:

```json
{
  "apiUrl": "https://api.example.com"
}
```

Сам frontend build одинаковый:

```text
              angular-app:1.0.0
                  │
          ┌───────┴───────┐
          ↓               ↓
        Test             Prod
          ↓               ↓
   test config.json   prod config.json
```

Меняется только runtime configuration.

Это особенно удобно в Docker/Kubernetes environments.

---

### Example 3

### Feature Flags

Configuration может содержать не только API URL.

Например:

```json
{
  "apiUrl": "https://api.example.com",
  "features": {
    "newDashboard": true,
    "experimentalMap": false
  }
}
```

Компонент:

```typescript
if (config.features.newDashboard) {
  // показать новый dashboard
}
```

Таким образом runtime configuration может использоваться как простой механизм feature flags.

## How to Apply

### 1. Создать environment configuration

В Angular CLI можно сгенерировать environment files:

```bash
ng generate environments
```

Получить структуру:

```text
src/environments/
├── environment.ts
└── environment.development.ts
```

---

### 2. Использовать одинаковые ключи

Например:

```typescript
export const environment = {
  apiUrl: 'https://api.example.com'
};
```

и:

```typescript
export const environment = {
  apiUrl: 'https://dev-api.example.com'
};
```

Структура должна оставаться одинаковой.

---

### 3. Использовать configuration в сервисах

```typescript
import { environment } from '../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class UserService {

  private readonly apiUrl = environment.apiUrl;

  getUsers() {
    return this.http.get<User[]>(
      `${this.apiUrl}/users`
    );
  }
}
```

---

### 4. Добавить Test Environment

Если нужен отдельный Test environment:

```text
environment.ts
environment.development.ts
environment.test.ts
```

И добавить соответствующий `fileReplacements` в `angular.json`.

Например:

```json
{
  "fileReplacements": [
    {
      "replace": "src/environments/environment.ts",
      "with": "src/environments/environment.test.ts"
    }
  ]
}
```

---

### 5. Для Docker/Kubernetes рассмотреть Runtime Configuration

Если deployment pipeline требует:

```text
Build once
     ↓
Deploy many times
```

runtime configuration часто оказывается более удобным решением.

Например:

```text
/assets/config.json
```

который создаётся или подменяется на этапе deployment.

## Connections

### Supports

- [[Angular Architecture]] — помогает отделить configuration от бизнес-логики и конкретных компонентов.
    
- [[Continuous Integration and Delivery]] — позволяет автоматически собирать и деплоить приложение в разные environments.
    
- [[Docker]] — runtime configuration позволяет использовать один image для разных окружений.
    
- [[Feature Flags]] — runtime configuration может хранить настройки включения и отключения функциональности.
    

### Contradicts

- [[Hardcoded Configuration]] — конфигурация хранится отдельно от конкретных сервисов и компонентов.
    
- [[Environment-dependent Business Logic]] — бизнес-логика не должна зависеть от того, в каком окружении запущено приложение.
    

### Extends

- [[12 Factor App Methodology]] — соответствует принципу отделения configuration от кода.
    
- [[Angular Application Configuration]] — расширяет базовую конфигурацию приложения до environment-aware architecture.
    
- [[CI/CD]] — runtime configuration позволяет отделить build pipeline от deployment configuration.
    

## Questions

- Как лучше организовать runtime configuration в Angular + Docker?
    
- Как передавать configuration через Kubernetes ConfigMap?
    
- Как безопасно передавать configuration из CI/CD pipeline?
    
- Когда стоит выбирать compile-time configuration, а когда runtime configuration?
    
- Как типизировать `config.json`?
    
- Как обработать ситуацию, когда `config.json` недоступен?
    
- Как реализовать feature flags без превращения configuration в огромный JSON?
    
- Как тестировать сервисы, которые используют runtime configuration?
    
- Как избежать утечки чувствительных данных через frontend configuration?
    

## Sources

- Angular Official Docs — Configuring application environments
    
- Angular Official Docs — Application Configuration
    
- Angular Official Docs — `provideAppInitializer`
    
- 12 Factor App — Config
    

---

**Review Status**: Seed

- Last reviewed: 2026-08-26
    
- Next review: 2026-09-25