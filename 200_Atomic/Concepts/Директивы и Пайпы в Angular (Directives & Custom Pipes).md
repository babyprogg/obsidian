---
type: atomic
status: seedling
domain:
created: 2026-08-18
updated: 2026-08-18
tags:
  - 
related: []
---
# Директивы и Пайпы в Angular (Directives & Custom Pipes)

### Core Idea
Директивы и пайпы — это декларативные инструменты Angular для управления DOM и форматирования данных прямо в шаблонах. Директивы изменяют внешний вид, поведение или структуру элементов, а пайпы трансформируют входные данные для отображения, не меняя исходного состояния.

### Why It Matters
Разделение ответственности — ключевой принцип Angular. Пайпы выносят логику форматирования (даты, валюты, фильтры) из компонента в шаблон, снижая дублирование. Директивы позволяют переиспользовать UI-поведение (подсветка, дрэг-н-дрэг, права доступа) без раздувания контроллеров. Понимание чистоты (purity) пайпов напрямую влияет на производительность и Change Detection.

---

### Key Points

* **Типы директив (`@Directive`):**
  * **Components:** Директивы с собственным HTML-шаблоном.
  * **Attribute Directives:** Изменяют внешний вид или поведение элемента (`ngClass`, `ngStyle`, кастомные атрибуты).
  * **Structural Directives:** Изменяют структуру DOM, добавляя или удаляя элементы (`@if`, `@for` в Control Flow или `*ngIf`, `*ngFor`).
* **Основы пайпов (`@Pipe` & `PipeTransform`):**
  * Принимают значение и опциональные аргументы (`value | pipeName:arg1:arg2`), возвращают трансформированный результат.
  * Используются исключительно для представления данных.
* **Чистые пайпы (Pure Pipes — `pure: true` по умолчанию):**
  * Пересчитываются **только при изменении ссылки** на передаваемый объект/массив или при изменении значения примитива.
  * Игнорируют внутреннюю мутацию объектов (например, `.push()` в массив без смены ссылки не вызовет пересчет).
  * Обеспечивают высокую производительность благодаря мемоизации.
* **Нечистые пайпы (Impure Pipes — `pure: false`):**
  * Вызываются на **каждом витке Change Detection** (любое событие, клик, таймер, HTTP-ответ).
  * Могут вызывать просадки FPS при тяжелых вычислениях, но необходимы для отслеживания внутренних мутаций или работы с внутренним состоянием (как `AsyncPipe`).

---

### Code Examples

#### 1. Кастомная атрибутная директива (Highlight on Hover)

```typescript
import { Directive, HostBinding, HostListener, Input, signal } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  @Input('appHighlight') highlightColor = 'ghostwhite';

  private isHovered = signal(false);

  @HostBinding('style.backgroundColor') get bgColor() {
    return this.isHovered() ? this.highlightColor : 'transparent';
  }

  @HostListener('mouseenter') onMouseEnter() {
    this.isHovered.set(true);
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.isHovered.set(false);
  }
}
```

2. Кастомный чистый пайп (Относительное время `TimeAgo`)
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'timeAgo',
  standalone: true,
  pure: true
})
export class TimeAgoPipe implements PipeTransform {
  transform(value: string | Date | number | null | undefined): string {
    if (!value) return '';

    const date = new Date(value);
    const now = new Date();
    const seconds = Math.floor((now.getTime() - date.getTime()) / 1000);

    if (seconds < 30) return 'только что';

    const intervals: Record<string, number> = {
      'г.': 31536000,
      'мес.': 2592000,
      'дн.': 86400,
      'ч.': 3600,
      'мин.': 60
    };

    for (const unit in intervals) {
      const counter = Math.floor(seconds / intervals[unit]);
      if (counter > 0) return `${counter} ${unit} назад`;
    }

    return date.toLocaleDateString();
  }
}
Использование в HTML:
<p appHighlight="lightgray">
  Опубликовано: {{ post.createdAt | timeAgo }}
</p>

### How to Apply

1. **Иммутабельность для Pure Pipes:** Всегда обновляйте массивы и объекты через создание новых ссылок (спред-оператор `[...arr, newItem]` или `.map()`), чтобы чистые пайпы своевременно обновляли View.
    
2. **Избегайте вызовов функций в шаблонах:** Вместо `{{ getFormattedData(item) }}` (который будет выполняться при каждом Change Detection) используйте Pure Pipe — он закеширует результат.
    
3. **Изоляция UI-поведения:** Если вам нужно одинаковое интерактивное поведение на разных элементах (маски ввода, тултипы, подсвечивание), создавайте атрибутную директиву, а не копируйте логику в компоненты.
### Connections

- **Supports:**
    
    - `[[Angular Change Detection]]` — чистые пайпы минимизируют лишние вычисления во время цикла проверки изменений.
        
    - `[[Immutability in JavaScript]]` — понимание работы ссылочных типов является обязательным условием для корректного применения Pure Pipes.
        
- **Extends:**
    
    - `[[Angular Templates & Control Flow]]` — дополняет синтаксис шаблонов инструментами трансформации и декларативного управления DOM.

### Questions

- Как устроен механизм отписки от подписок внутри встроенного `AsyncPipe` (который является `pure: false`)?
    
- Как писать юнит-тесты (`TestBed`) для кастомных директив, использующих `HostListener` и `HostBinding`?
    

### Sources

- Angular Official Docs: Directives & Custom Pipes
    
- Angular Change Detection & Pipe Purity Deep Dive
    

**Review Status:** Draft

**Last reviewed:** 2026-08-18

**Next review:** 2026-09-17