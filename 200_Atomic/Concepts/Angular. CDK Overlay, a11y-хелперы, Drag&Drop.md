---
type: atomic
status: seedling
domain:
created: 2026-09-14
updated: 2026-09-14
tags:
  - 
related: []
---
# Angular CDK: Overlay, a11y & Drag and Drop

## Core Idea

Angular CDK (Component Development Kit) — это набор бессерверных и бессилевых (headless) утилит, берущих на себя сложную физику и логику взаимодействия элементов (позиционирование вне контекста DOM, доступность с клавиатуры, математику перетаскивания), сохраняя полный контроль над внешним видом за разработчиком.

## Why It Matters

Позволяет не изобретать велосипед при создании сложных UI-компонентов (модалок, дропдаунов, Канбан-досок), изолирует от проблем с `overflow: hidden`, `z-index` и фокусом клавиатуры, а также снижает когнитивную нагрузку при соблюдении стандартов доступности (WCAG/a11y).

## Key Points

- **Overlay (Верхний слой / Top Layer):** Выносит рендер элемента в конец `<body>` (`cdk-overlay-container`), избавляя от ограничений родительских `stacking context`. Математически привязывает оверлей к origin-элементу через `ConnectedPosition Strategy`.
    
- **Accessibility (a11y):** Предоставляет готовую логику удержания фокуса клавиатуры (`FocusTrap`), анонсирования изменений для скринридеров (`LiveAnnouncer`) и разграничения источников ввода (`FocusMonitor`).
    
- **Drag & Drop:** Автоматизирует расчёт координат, генерацию превью/заглушек и межсписковое связывание (`cdkDropListConnectedTo`) для перетаскиваемых элементов.
    

## Examples

### Example 1: Выпадающий список (Overlay + a11y)

Выпадающий список внутри таблицы с `overflow: hidden`. При клике CDK выносит контекстное меню в корень `<body>`, позиционирует ровно под кнопкой, а `FocusTrap` не дает пользователю уйти клавишей `Tab` за пределы открытого меню.

### Example 2: Интерактивная Канбан-доска (Drag & Drop)

Перетаскивание карточек задач между колонками "Backlog", "In Progress" и "Done". Директивы `cdkDrag` и `cdkDropList` автоматически сдвигают соседние карточки при наведении и анимируют возврат, если карточка была сброшена в неактивную зону.

## How to Apply

1. **Подключить нужный модуль CDK:** Импортировать `OverlayModule`, `A11yModule` или `DragDropModule` в компонент.
    
2. **Для Overlay:** Создать оверлей через `Overlay` сервис, задать стратегию позиционирования (`flexibleConnectedTo`) и прикрепить шаблон/компонент через `ComponentPortal` или `TemplatePortal`.
    
3. **Для a11y:** Повесить директиву `cdkFocusTrap` на модальное окно или использовать `FocusMonitor` в коде для отслеживания источника фокуса.
    
4. **Для Drag & Drop:** Обернуть контейнер в `cdkDropList`, а перетаскиваемые элементы в `cdkDrag`. Для связывания списков передать массив ID в `[cdkDropListConnectedTo]`.
    

## Connections

### Supports

- [[Angular Component Architecture]] — разделение ответственности между логикой взаимодействия (CDK) и представлением (Stated/UI Component).
    
- [[Design Systems]] — позволяет строить масштабируемые кастомные киты компонентов с единым поведением.
    

### Contradicts

- [[Inline Modals & Absolutes]] — противостоит прямому верстанию модалок через `position: absolute` и `z-index` внутри обычного дерева DOM.
    

### Extends

- [[DOM APIs]] — надстройка над native DOM API (IntersectionObserver, getBoundingClientRect, ARIA standards).
    

## Questions

-  Как эффективно очищать `OverlayRef` при отмонтировании родительского компонента, чтобы избежать утечек памяти?
    
-  Какое влияние оказывает массовый `Drag & Drop` на `Zoneless` архитектуру в современных версиях Angular?
    

## Sources

- Angular CDK Official Documentation: [https://material.angular.io/cdk/categories](https://material.angular.io/cdk/categories)
    

---

**Review Status:** Draft

- Last reviewed: 2026-09-14
    
- Next review: 2026-10-14