---
type: atomic
status: seedling
domain:
created: 2026-09-07
updated: 2026-09-07
tags:
  - 
related: []
---
# Signal-based queries (viewChild / contentChild)

## Core Idea

Запросы `viewChild` и `contentChild` — это встроенные реактивные функции Angular, которые возвращают `Signal` с прямой ссылкой на DOM-элемент, дочерний компонент или директиву.

## Why It Matters

Они обеспечивают декларативный и безопасный доступ к элементам UI-поведения (фокус, скролл, измерения, Canvas) без сложных хуков жизненного цикла (`ngAfterViewInit`), ошибок `ExpressionChangedAfterItHasBeenCheckedError` и деклараций с провайдерами `ElementRef`.

## Key Points

- **`viewChild` (View Query):** Ищет элементы внутри личного HTML-шаблона компонента.
    
- **`contentChild` (Content Query):** Ищет элементы, проброшенные снаружи через проекцию контента (`ng-content`).
    
- **Сигналы из коробки:** Запросы возвращают `Signal<T | undefined>` (или `Signal<T>` при использовании `.required`), автоматически отслеживая динамическое появление или удаление элементов из DOM.
    
- **Область применения:** Предназначены для императивного UI-поведения и вызова браузерных API, а **не** для прямого мутирования стилей и классов (для этого используются байндинги шаблона `[class]` / `[style]`).
    

## Examples

### Example 1: Управление фокусом инпута (View Query)

```typescript
import { Component, viewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-search-bar',
  standalone: true,
  template: `
    <input #searchInput type="text" placeholder="Поиск..." />
    <button (click)="focusInput()">Найти</button>
  `
})
export class SearchBarComponent {
  // Получаем ссылку на DOM-узел инпута
  private inputRef = viewChild<ElementRef<HTMLInputElement>>('searchInput');

  focusInput(): void {
    // Безопасно вызываем native-метод фокуса
    this.inputRef()?.nativeElement.focus();
  }
}
```

### Example 2: Вызов метода проброшенного компонента (Content Query)

```typescript
import { Component, contentChild } from '@angular/core';
import { CustomHeaderComponent } from './custom-header.component';

@Component({
  selector: 'app-card',
  standalone: true,
  template: `<ng-content></ng-content>`
})
export class CardComponent {
  // Ищем кастомную шапку, переданную в ng-content
  private header = contentChild(CustomHeaderComponent);

  highlightHeader(): void {
    // Дергаем публичный метод дочернего компонента
    this.header()?.activateHighlight();
  }
}
```

## How to Apply

1. **Определи тип запроса:** используй `viewChild` для элементов своего шаблона или `contentChild` для элементов из `ng-content`.
    
2. **Объяви свойство класса** с помощью функции-запроса:
    
    - `myEl = viewChild<ElementRef>('target')`
        
    - `myEl = viewChild.required(...)`, если элемент гарантированно присутствует.
        
3. **Используй ссылку в коде**, вызывая сигнал:
    
    - `this.myEl()?.nativeElement` — для работы с Native API (фокус, скролл, Canvas).
        
    - `this.myEl()?.methodName()` — для вызова публичных методов компонентов.
        

## Connections

### Supports

- [[Angular Signals]] — расширяет единый реактивный паттерн приложения на DOM-запросы.
    
- [[Angular Content Projection]] — обеспечивает управление проброшенным контентом через `contentChild`.
    

### Contradicts

- [[@ViewChild Decorator]] — заменяет устаревший синтаксис на базе декораторов и обязательных lifecycle-хуков (`ngAfterViewInit`).
    

### Extends

- [[ElementRef in Angular]] — оборачивает работу с `ElementRef` в автоматическую реактивную обертку.
    

## Questions

- Как правильно организовывать множественные запросы через `viewChildren` и `contentChildren` при работе с динамическими списками?
    
- Каковы особенности поведения `viewChild.required` при использовании с асинхронными структурами (`@defer`)?
    

## Sources

- Angular Official Documentation: Signal queries (`viewChild`, `contentChild`)
    
- Angular Developer Guide: Working with DOM references
    

---

**Review Status**: Draft

- Last reviewed: 2026-09-07
    
- Next review: 2026-10-07