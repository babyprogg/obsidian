---
type: atomic
status: seedling
domain:
created: 2026-03-18
updated: 2026-03-18
tags:
  - 
related: []
---

# Nx Module Boundaries

## Core Idea
[One sentence: What is this concept?]
Это механизм принудительного соблюдения архитектурных правил в монорепозитории через ограничение импортов между программными модулями на основе метаданных (тегов).

## Why It Matters
[Why should I care about this?]
Без жестких границ любой монорепозиторий со временем превращается в "большой ком грязи" (Big Ball of Mud), где изменение в одном сервисе неожиданно ломает фронтенд, а время сборки в CI растет в геометрической прогрессии из-за запутанных зависимостей.

## Key Points

- **Тегирование:** Каждому проекту в `project.json` назначаются `tags` (например, `scope:orders`, `type:ui`).
- **Линтинг:** Правило `@nx/enforce-module-boundaries` проверяет соответствие импортов в реальном времени.
- **Иерархия слоев:** Позволяет реализовать направленный граф зависимостей (например, "сверху вниз").
- **Изоляция доменов:** Гарантирует, что бизнес-логика одного домена не "протекает" в другой без явного разрешения.

## Examples

### Example 1
[Concrete example of concept in action]
#### Архитектурные слои (FSD/DDD)

Запрет на импорт из `features` в `shared`. Если разработчик попытается импортировать логику корзины (feature) в общую кнопку (shared ui), линтер выдаст ошибку: _"A library tagged with type:shared may only depend on libraries tagged with type:shared"_.
### Example 2
[Another example, ideally different domain]
#### Кросс-доменные ограничения

В приложении для доставки еды библиотека `libs/courier/maps` не должна импортировать `libs/customer/reviews`. Это разные бизнес-контексты. Границы модулей заставят вынести общие данные в `shared` или использовать события вместо прямых зависимостей.
## How to Apply

[Practical steps to use this concept]

1. **Разметить проекты:** Добавить теги `scope` и `type` в файлы `project.json` всех библиотек.
2. **Настроить правила:** В корневом `.eslintrc.json` в секции `overrides` найти правило `nx/enforce-module-boundaries`.
3. **Определить зависимости:** В массиве `depConstraints` прописать, какой тег может зависеть от какого (например, `sourceTag: "type:feature"`, `onlyDependOnLibsWithTags: ["type:ui", "type:data-access"]`).
4. 1. **Проверить в IDE:** Убедиться, что плагин ESLint подсвечивает запрещенные импорты красным.

## Connections

### Supports
- [[]] - How this note supports another concept

### Contradicts
- [[]] - What this disagrees with

### Extends
- [[]] - What this builds upon

## Questions

- [What remains unclear?]
- [What needs research?]

## Sources

- Nx Documentation: [Enforce Module Boundaries](https://nx.dev/core-features/enforce-module-boundaries)
- Enterprise Monorepo Patterns by Nrwl.

---

**Review Status**: Active
- Last reviewed: 2026-03-18
- Next review: 2026-04-17