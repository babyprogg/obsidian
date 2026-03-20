---
type: note
status: active
created: 2026-02-25
updated: 2026-02-25
deadline:
next-action:
tags:
  - project/active
  - 
related: []
---

# Челленджы скейлинга проекта

## Context
[Why this note was created / what prompted it]
Это нужно чтобы понимать проблемы проекта при скейлинга в ближайшем будущем и при этом пытаться работать на перед и сейчас уже потихоньку смягчать углы. 
## Content
[Main information, thoughts, or description]
## Challenge 1: Запутанные связи (Module Interdependencies)

**Проблема:** Модули знают друг о друге слишком много.

- **Пример «как плохо»:** Чтобы создать **Бизнес**, тебе нужно выбрать **Клиента**. Ты импортируешь `ClientService` прямо в `BusinessComponent`.
    
- **Что происходит:** Теперь `Business` не может существовать без `Client`. Если ты захочешь вынести `Business` в отдельное микро-приложение или просто протестировать его, тебе придется тянуть за собой весь код Клиентов. Это называется **Tight Coupling** (жесткая связь).
    
- **Как это выглядит в коде:**
    
    TypeScript
    
    ```
    // В Business.component.ts
    import { ClientService } from '@my-app/client'; // ОПАСНО: прямая зависимость между доменами
    ```
    

---

## Challenge 2: Сложные формы (Form State)

**Проблема:** Данные формы разбросаны по кусочкам, и их трудно собрать воедино.

- **Пример:** Представь регистрацию Бизнеса из 5 шагов (Шаг 1: Имя, Шаг 2: Адрес, Шаг 3: Налоги...).
    
- **Что происходит:** Пользователь заполнил 4 шага и нажал «Назад» или случайно обновил страницу. Если данные хранятся только внутри маленьких компонентов, они пропадут.
    
- **Боль:** Как сделать кнопку «Отменить всё» (Undo), если каждый инпут живет своей жизнью? Стандартный Angular не дает «коробку» для управления состоянием таких гигантских форм.
    

---

## Challenge 3: Тормоза (Performance at Scale)

**Проблема:** Приложение «тяжелеет» и начинает лагать.

- **Пример с виртуализацией:** У тебя есть список из 2000 локаций. Без виртуализации Angular отрисует 2000 блоков `<div>`. Если пользователь начнет быстро скроллить, браузер "заикнется", потому что он пытается перерисовать тысячи элементов одновременно.
    
- **Пример с Nx Cache:** У вас в проекте 50 модулей. Ты изменил одну запятую в `UserModule`. Если `buildable: false`, то Nx не поймет, что остальные 49 модулей не изменились, и начнет пересобирать **весь** проект с нуля. Это лишние 5-10 минут ожидания билда.
    

---

## Challenge 4: Сложность тестирования (Testing Strategy)

**Проблема:** Ты проверяешь детали, но не видишь всей картины.

- **Пример:** Ты написал Unit-тест для кнопки (проверил, что она нажимается). Это круто. Но у тебя **нет E2E тестов** (End-to-End).
    
- **Что происходит:** Ты обновил версию библиотеки для графиков. Unit-тесты прошли (кнопки-то работают!), но на реальной странице график перестал отображаться. Без E2E тестов (которые имитируют действия реального юзера в браузере) ты узнаешь о баге только от разгневанного клиента.
    
- **Сложность с Signals:** Сигналы обновляются мгновенно. В старом RxJS можно было сказать тесту: «Подожди 100мс и проверь результат». В Сигналах сложнее «поймать» промежуточное состояние для проверки.

## Key Takeaways
- **Ch 1:** Про архитектуру (чтобы всё не превратилось в спагетти).
- **Ch 2:** Про удобство юзера в длинных формах.
- **Ch 3:** Про скорость работы сайта и твоего компьютера при сборке.
- **Ch 4:** Про уверенность в том, что завтра ничего не сломается.

## Links
- [[]] - 
- [[]] - 

## Sources
- 

## Next Steps
[Optional: what to do with this information]
- 
- 

---
**Last Updated**: 2026-02-25
# Project: Без названия

## Vision

[What will this be when complete? Paint the picture.]

## Why This Project

[Why now? What problem does this solve?]

### Connects These Interests
- [[Interest 1]]
- [[Interest 2]]
- [[Interest 3]]

## Success Criteria

[How will I know it's done and successful?]

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Phases

### Phase 1: [Name]
**Goal**: 
**Deadline**: 
**Status**: Not started | In progress | Complete

- [ ] Task 1
- [ ] Task 2

### Phase 2: [Name]
**Goal**: 
**Deadline**: 

- [ ] Task 1

## Next Actions

**Immediate** (Do this week):
- [ ] 

**Soon** (Do this month):
- [ ] 

**Someday** (Future):
- [ ] 

## Resources Needed

### Knowledge
- Need to learn: [[]]
- Reference: [[]]

### Tools
- 
- 

### Help
- Who could help:

## Challenges & Solutions

### Challenge 1
**Problem**: 
**Solution**: 

## Learning Goals

[What will I learn from this project?]

- 
- 

## Related

### Projects
- [[Related Project]] - How they connect

### Notes
- [[Relevant Concept]]

## Log

### 2026-02-25
[What happened today? Decisions made? Blockers hit?]

### Template for new entries