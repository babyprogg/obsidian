---
type: atomic
status: seedling
domain:
created: 2026-09-09
updated: 2026-09-09
tags:
  - 
related: []
---
# Content Projection и Кастомные Структурные Директивы в Angular

## Core Idea

Content Projection (`<ng-content>`) подставляет внешнюю HTML-разметку в заданные слоты компонента, а кастомные структурные директивы (`*directive`) динамически создают или удаляют элементы DOM с помощью `TemplateRef` и `ViewContainerRef`.

## Why It Matters

Позволяет проектировать гибкую UI-библиотеку (Atomic Design) без раздувания API инпутов (`@Input`), разделяя визуальное оформление компонента от бизнес-логики и управления состоянием DOM.

## Key Points

- **Content Projection:** Компонент сдает в аренду места (`<ng-content>` или `<ng-content select="...">`), а родитель сам решает, чем их заполнить.
    
- **Structural Directives:** Превращают шаблон в `TemplateRef` (чертеж) и управляют его внедрением в `ViewContainerRef` (контейнер в DOM).
    
- **Разделение ответственности:** Атомы и Молекулы используют проекцию контента для гибкой верстки, а Организмы и кастомные директивы управляют логикой и условиями отображения.
    

## Examples

### Example 1

**Multi-slot Projection в Молекуле (FormField):**

Передача лейбла, кастомного поля ввода и зоны ошибок через селекторы:

```html
<label><ng-content select="label" /></label>
<div class="control"><ng-content select="input, select" /></div>
<div class="errors"><ng-content select="ui-error" /></div>
```

### Example 2

**Структурная директива контроля доступа (`*appHasRole`):**

Директива внедряет кнопкам/блокам `TemplateRef` только в том случае, если текущий пользователь имеет необходимые права доступа:

```typescript
if (this.auth.hasRole(role)) {
  this.vcr.createEmbeddedView(this.templateRef);
} else {
  this.vcr.clear();
}
```

## How to Apply

1. **Atoms & Molecules:** Для создания переиспользуемых базовых компонентов UI-кита применяй single-slot и multi-slot `<ng-content>`.
    
2. **Custom Directives:** Когда нужно скрыть, размножить или подменить элемент по условию (авторизация, скелетоны, A/B тесты), создавай директиву с инжекцией `TemplateRef` и `ViewContainerRef`.
    
3. **Organisms:** Объединяй сложную верстку (проекцию) с кастомными директивами состояния в организмах (таблицы, карточки, модалки).
    

## Connections

### Supports

- [[Atomic Design]] — помогает реализовывать гибкую композицию UI-компонентов.
    
- [[Single Responsibility Principle]] — разделяет визуальную структуру и логику управления DOM.
    
- [[Clean Architecture]] — способствует разделению ответственности между слоями.
    

### Contradicts

- Избыточное использование `@Input()` для передачи большого количества флагов отображения и верстки.
    

### Extends

- [[Angular Component Architecture]]
    
- [[Native Control Flow (@if, @for)]]
    

## Questions

- Как оптимально сочетать кастомные структурные директивы с новым синтаксисом Control Flow (`@if` / `@for`)?
    
- Какой оверхед создают многослотовые `<ng-content>` при изменении Change Detection в стратегии `OnPush`?
    

## Sources

- Официальная документация Angular (Content Projection & Structural Directives)
    
- Atomic Design by Brad Frost
    

---

**Review Status:** Draft

- Last reviewed: 2026-09-09
    
- Next review: 2026-10-09