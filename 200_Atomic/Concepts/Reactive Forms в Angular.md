---
type: atomic
status: seedling
domain:
created: 2026-08-24
updated: 2026-08-24
tags:
  - angular
  - frontend
  - architecture
related: []
---


# Reactive Forms в Angular

### Core Idea
Декларативный подход к управлению формами в Angular, при котором структура, логика, состояние и валидация формы полностью описываются в TypeScript-классе компонента как иммутабельный поток данных.

### Why It Matters
Исключает хаос при работе со сложным пользовательским вводом, делая состояние формы полностью предсказуемым, изолированным от DOM-дерева и легко тестируемым без необходимости рендеринга интерфейса.

### Key Points
* **Явное состояние:** Логика и правила валидации определяются в TS-файле (`FormGroup`, `FormControl`, `FormArray`), а не размазываются по HTML-шаблону.
* **Синхронность и иммутабельность:** Каждое изменение создаёт новое состояние формы, исключая побочные эффекты двустороннего связывания.
* **Реактивность через RxJS:** Потоки `valueChanges` и `statusChanges` позволяют использовать операторы `debounceTime`, `distinctUntilChanged` и `switchMap` прямо "из коробки".
* **Строгая типизация:** Поддержка Strict Typed Forms гарантирует совпадение типов формы с DTO на этапе сборки.

### Examples

#### Example 1: Фильтрация таблицы с задержкой ввода (Debounce)
```typescript
readonly searchControl = new FormControl('', { nonNullable: true });

ngOnInit(): void {
  this.searchControl.valueChanges.pipe(
    debounceTime(300),
    distinctUntilChanged(),
    switchMap(query => this.apiService.search(query))
  ).subscribe(results => this.data.set(results));
}
```
Example 2: Динамический список элементов (FormArray)
``` ts 
readonly profileForm = this.fb.group({
  user: ['', Validators.required],
  skills: this.fb.array([this.fb.control('')])
});

get skillsArray(): FormArray {
  return this.profileForm.controls.skills as FormArray;
}

addSkill(): void {
  this.skillsArray.push(this.fb.control(''));
}
```
### How to Apply

1. Подключить `ReactiveFormsModule` или импортировать его напрямую в `standalone`-компонент.
    
2. В TS-классе сконфигурировать дерево контролов через `FormBuilder` или напрямую инстанцируя `FormGroup` / `FormControl`.
    
3. Задать необходимые валидаторы (`Validators.required`, кастомные синхронные/асинхронные функции).
    
4. Связать форму с HTML-шаблоном через директивы `[formGroup]` и `formControlName`.
    
5. Обработать отправку данных через событие `(ngSubmit)` с проверкой `form.valid`.
    

### Connections

- **Supports**
    
    - [[Clean Architecture]] — Изолирует бизнес-логику валидации и управления вводом от UI-слоя.
        
    - [[Unit Testing in Angular]] — Позволяет тестировать сценарии ввода и валидации без создания DOM-элементов.
        
- **Contradicts**
    
    - [[Template-driven Forms]] — Отказывается от двустороннего связывания `[(ngModel)]` и описания правил в шаблоне.
        
- **Extends**
    
    - [[RxJS Observables]] — Использует архитектуру Observer для передачи изменений состояния через `valueChanges`.
        
    - [[Angular Signals]] — Служит основой для конвертации потоков формы в сигналы через `toSignal()`.
        

### Questions

- Как оптимально интегрировать Zod/Valibot для валидации `Reactive Forms` без использования стандартных `Validators`?
    
- Каким будет нативный API `Signal-driven Forms` при полном отказе от `@angular/forms` в будущих релизах Angular?
    

### Sources

- Official Angular Documentation: Reactive Forms Guide
    
- Angular RFC: Typed Forms & Signal Integration
    

**Review Status:** Draft

**Last reviewed:** 2026-08-24

**Next review:** 2026-09-23