---
type: atomic
status: seedling
domain: code
created: 2026-09-16
updated: 2026-09-16
tags:
  - domain/code
  - type/technique
  - angular
related: []
---
# Современные анимации в Angular (вместо `@angular/animations`)

## Core Idea

Пакет `@angular/animations` устарел (deprecated), а современная Angular-разработка использует сочетание Tailwind CSS, нативного View Transitions API и сторонних библиотек вместо громоздкого DSL-синтаксиса.

## Why It Matters

Написание нативных анимаций Angular (`trigger`, `state`, `transition`) увеличивает размер бандла, усложняет поддержку и уступает по производительности чистым CSS-решениям и нативному браузерному API.

## Key Points

- **Deprecated DSL:** `@angular/animations` признан громоздким и избыточным.
    
- **Tailwind & CSS First:** Микро-взаимодействия и UI-компоненты (shadcn, Spartan UI) анимируются через Tailwind CSS классы и CSS Keyframes.
    
- **View Transitions API:** Переходы между роутами осуществляются встроенной функцией `withViewTransitions()` браузера.
    
- **Сложный интерактив:** Для продвинутой графики и сложных цепочек анимаций применяются GSAP, Motion One, [Three.js](https://three.js/) или Angular CDK.
    

## Examples

### Example 1: Анимация роутинга с View Transitions API

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withViewTransitions())
  ]
};
```

### Example 2: Анимация компонента через Tailwind CSS

```typescript
@Component({
  selector: 'app-accordion',
  standalone: true,
  template: `
    <div
      class="transition-all duration-300 ease-in-out overflow-hidden"
      [class.max-h-0]="!isOpen()"
      [class.max-h-96]="isOpen()">
      Контент аккордеона
    </div>
  `
})
export class AccordionComponent {
  isOpen = signal(false);
}
```

## How to Apply

1. Отказаться от импорта `BrowserAnimationsModule` и пакета `@angular/animations` в новых проектах.
    
2. Использовать Tailwind CSS или CSS-переходы для микро-взаимодействий и состояния UI-элементов.
    
3. Включить `withViewTransitions()` в конфигурации роутера для плавных переходов между страницами.
    
4. Подключать специализированные JS-библиотеки (например, GSAP) только при необходимости сложных анимационных последовательностей.
    

## Connections

### Supports

- [[Angular Architecture]]
    
- [[Tailwind CSS]]
    
- [[Web Performance]]
    

### Contradicts

- [[Legacy Angular Animations DSL]]
    

### Extends

- [[View Transitions API]]
    
- [[Angular Signals|Modern Angular Signals]]
    

## Questions

- _Каковы лучшие практики для анимации `:enter` и `:leave` элементов при использовании Angular Signals без DSL-триггеров?_
    
- _Как View Transitions API работает со сложными анимациями совмещенных элементов (shared element transitions)?_
    

## Sources

- Angular Official Docs (Deprecated APIs & View Transitions API)
    
- Spartan UI / shadcn-angular Animation Guidelines
    

---

**Review Status:** New

**Last reviewed:** 2026-09-16

**Next review:** 2026-10-16