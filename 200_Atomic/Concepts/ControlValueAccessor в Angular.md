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
# ControlValueAccessor в Angular

## Core Idea

Интерфейс-адаптер ("переводчик"), который связывает кастомный компонент Angular с Angular Forms API (`FormControl`, `ngModel`), позволяя компоненту работать как стандартное поле ввода.

## Why It Matters

Без CVA кастомные компоненты (селекторы, рейтинги, слайдеры) не могут нативно участвовать в реактивных или шаблонных формах, получать значение от формы, передавать изменения наружу и управлять статусами валидации/блокировки.

## Key Points

- **`writeValue(val)`** — принимает новое значение из формы и обновляет внутреннее состояние компонента (Form → View).
    
- **`registerOnChange(fn)`** — сохраняет функцию-callback от Angular и вызывает её при изменении значения в компоненте (View → Form).
    
- **`registerOnTouched(fn)`** — сохраняет и вызывает callback, когда пользователь взаимодействовал с компонентом (помечает поле как `touched`).
    
- **`setDisabledState(isDisabled)`** — отвечает за реакцию компонента на блокировку формы (`control.disable()`).
    
- **`NG_VALUE_ACCESSOR` + `forwardRef`** — регистрация компонента в DI-системе Angular как валидного источника данных формы.
    

## Examples

### Example 1: Кастомный рейтинг звездочками (`<app-star-rating>`)

Компонент отрисовывает 5 звёзд. При клике на звезду он вызывает `this.onChange(rating)`, передавая число в `FormControl`.

Когда форма сбрасывается через `formControl.reset()`, Angular вызывает `writeValue(null)`, и звёздочки гаснут.

### Example 2: Кастомный переключатель темы/тумблер (`<app-toggle-switch>`)

Компонент с анимацией тумблера "Вкл/Выкл". При клике меняет визуальное состояние и вызывает `this.onChange(boolean)`.

При `control.disable()` Angular вызывает `setDisabledState(true)`, после чего тумблер становится серым и неинтерактивным.

## How to Apply

1. Не писать с нуля по памяти — копировать проверенный шаблон или использовать сниппет IDE.
    
2. Добавить `implements ControlValueAccessor` к классу компонента.
    
3. Зарегистрировать `NG_VALUE_ACCESSOR` в массиве `providers` декоратора `@Component`.
    
4. Реализовать 4 метода, сохраняя callbacks `onChange` и `onTouched`.
    
5. Вызывать `this.onChange(val)` при любом внутреннем изменении значения компонента.
    

## Connections

### Supports

- [[Reactive Forms в Angular|Angular Reactive Forms]] — позволяет включать сложные UI-компоненты в структуры `FormGroup` и `FormArray`.
    
- [[Иерархия инжекторов и inject() в Angular|Angular Dependency Injection]] — использует мульти-провайдеры (`multi: true`) для связывания с механизмом форм.
    

### Contradicts

- [[Direct @Input/@Output Binding]] — в отличие от дуэта `@Input() value` + `@Output() valueChange`, CVA даёт полную интеграцию с валидацией, статусами `touched`/`dirty`/`disabled` и единым API форм.
    

### Extends

- [[Component-Driven Architecture]] — превращает обычные UI-компоненты в переиспользуемые элементы форм.

- [[Общий разбор папки libs]] — design-system/cdk/forms (Custom Form Control Kit) в Gerer Construire — место, где CVA применяется на практике.
    

## Questions

- Как правильно обрабатывать валидацию внутри CVA через интерфейс `Validator`?
    
- Как упростить CVA с появлением Angular Signals?
    

## Sources

- Официальная документация Angular: `ControlValueAccessor` API
    
- Проектная база кода (поиск по `NG_VALUE_ACCESSOR`)
    

---

**Review Status**: Drafted

**Last reviewed**: 2026-09-16

**Next review**: 2026-10-16