---
type: project
status: active
created: 2026-01-15
updated: 2026-01-15
deadline:
next-action:
tags:
  - project/active
  - dev
---

# Project: Gerer Construire

## Vision

[What will this be when complete? Paint the picture.]
Это рабочий проект, но когда я закончу это у меня будет сильный проект в портфолио и бабки. А также много опыта и новых штучек который используются в индустрии, работа с командой уже плотных типов и кооп с фронтенд лидом


## Why This Project

[Why now? What problem does this solve?]
Бабки

### Connects These Interests
- [[Interest 1]]
- [[Interest 2]]
- [[Interest 3]]

## Success Criteria

[How will I know it's done and successful?]

- [ ] Команда довольно мною
- [ ] Клиент доволен
- [ ] Получил много бабок

## Phases

### Phase 1: Начало 
**Goal**: Просто познакомиться с кодом, разобраться с базой и определить задачи.
**Deadline**: 
**Status**: Not started | <mark style="background: #FFB8EBA6;">In progress</mark> | Complete

- [x] Созвон
- [x] Соло разбор

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

- Работа в команде
- Как мыслит лид фронтенд разраб

## Related

### Projects
- [[Unique Learner]] - Брать оттуда решения проблем которыми я встретился и здесь. 

### Notes
- [[Общий разбор папки libs]]
- 

## Log

### 2026-01-15
[What happened today? Decisions made? Blockers hit?]
### 2026-01-21
Now when you try to register, the mock interceptor will handle the request and return a successful response instead of hitting the broken backend. You can test the frontend flow completely including the success message you just added!
To disable it later, simply remove registrationMockInterceptor from the interceptors array in app.config.ts when your backend is working.

### 2026-07-23
## Почему CREATE — это один тикет, а не три

Шаги 1–7 и альт-сценарии (отмена / повторное открытие) в спеке — это **один атомарный пользовательский путь**, а не отдельные фичи:

- Модалка выбора структуры не имеет смысла сама по себе — её кнопка "Suivant" _и есть_ действие создания страницы. Если бы это были два разных МРа, у вас в какой-то момент существовала бы модалка, которая ничего не создаёт, либо возможность создать страницу без способа туда попасть. Откатить один без другого = сломанная фича, а не рабочая версия минус кусок. Это прямо противоречит цели "откатить фичу не сломав остальное".
- Альт-сценарий "повторное открытие без модалки" — это просто ветвление одной и той же логики (`if (structureAlreadySelected) skip modal`), не отдельная возможность.

Ещё важный момент: этот тикет неизбежно тянет за собой **весь фундамент модуля** — domain-сущности (Titre/SousTitre/Commentaire), infrastructure (store, events, provider), роутинг, пункт в меню Actions PV. Без этого физически нечему рендерить дерево. Это прямая параллель с вашим модулем Tâches: там **LIST** — не просто "показать список", а фундамент (store + страница + роутинг), от которого зависят CREATE/EDIT/ARCHIVE/... Здесь **CREATE** играет ту же роль "ствола".

## Почему ADD TITRE / ADD SOUS-TITRE / ADD COMMENTAIRE — три независимых тикета, а не один "ADD *"

Ключевой факт из спеки (Таблица 2): Kimanage-структура **уже приходит с готовыми titre A/B и их sous-titres** при создании страницы (в рамках тикета CREATE). Это значит:

```
CREATE (ствол — создаёт page + default titre/sous-titre)
   ├── ADD TITRE          — не зависит от ADD SOUS-TITRE / ADD COMMENTAIRE
   ├── ADD SOUS-TITRE     — цепляется к titre, который УЖЕ существует из CREATE
   └── ADD COMMENTAIRE    — цепляется к sous-titre, который УЖЕ существует из CREATE
```

Проверка независимости — по каждой паре:

- **ADD SOUS-TITRE не требует ADD TITRE.** Новый sous-titre добавляется к дефолтному titre A/B, который уже есть. Пользователь может пользоваться "+ Ajouter sous-titre" даже если фича "+ Ajouter titre" ещё не задеплоена.
- **ADD COMMENTAIRE не требует ADD SOUS-TITRE.** Комментарий добавляется к дефолтным sous-titres (Lieu de chantier / Durée des travaux), которые уже есть с самого создания страницы.
- **ADD TITRE не требует ничего от двух других.** Новый titre может существовать пустым (сколлапсированным, без sous-titres) — это валидное состояние UI.

Это подтверждается и на уровне кода: у каждого — свой abstract repository (`IAddTitreRepository`/`IAddSousTitreRepository`/`IAddCommentaireRepository`), свой use-case, своя модалка, своя кнопка. Они пишут в общее дерево, но не читают и не зависят от результатов друг друга — это классический fan-out от одного ствола, а не цепочка. Значит:

- Их можно мёржить в любом порядке, хоть параллельно разными людьми.
- Если, скажем, ADD COMMENTAIRE после мёржа сломает что-то (например, дизайн-системный `gc-textarea`, который для него нужно завести с нуля) — ревертите только этот МР: кнопка, модалка, use-case, repo уходят, а Titre/Sous-titre фичи и сама страница не затрагиваются.

## Что я сознательно НЕ выносил отдельным тикетом

- **Сворачивание/разворачивание (стрелки)** — это часть верстки страницы из CREATE, не отдельная фича: без CREATE рендерить нечего, а без стрелок дерево из CREATE всё равно полностью функционально (просто не сворачивается) — то есть выносить в отдельный МР ради возможности отката не имеет смысла, риск точечный и низкий.
- **Кебаб-меню (⋮) на sous-titre** — по спеке прямо относится к F307.2 (ваш GC-463), не входит в F307.1 вообще.
- **`gc-textarea` (design-system) и хардкод PRO-гейтинга** — технически мелкие подзадачи внутри ADD COMMENTAIRE и CREATE соответственно, не самостоятельная ценность вне контекста своего тикета, поэтому не отдельные МРы.


# Généralités — что и почему (объяснение для ревью без вникания в код)

  

## 1. Структура модуля

  

Новый модуль `libs/modules/generalites/` собран по той же схеме, что и все

остальные фичи в проекте (`meeting`, `task`, `location`...) — 4 слоя, каждый

слой это отдельная nx-библиотека:

  

```

libs/modules/generalites/

  domain/           — сущности + контракты репозиториев (чистый TS, без Angular)

  application/       — юз-кейсы (бизнес-операции)

  infrastructure/     — стор, in-memory репозиторий, DI-провайдеры

  presentation/

    pages/generalites-page/  — сама страница (роутится)

    ui/                       — модалка выбора структуры

```

  

**Зачем такое разделение вообще**: чтобы бизнес-логика (`domain`,

`application`) не знала про Angular и HTTP. Если завтра появится реальный

бэкенд-эндпоинт — меняется только `infrastructure`, ни `domain`, ни

`application`, ни компоненты трогать не нужно.

  

Направление зависимостей — только "внутрь":

`presentation → infrastructure → application → domain`.

Domain не знает вообще ни о чём. Это проверяется автоматически линтером

(ESLint module boundaries) — если кто-то попытается импортировать Angular в

`domain`, сборка упадёт.

  

## 2. Что лежит в каждом слое (по факту, не абстрактно)

  

### domain — `generalites.ts`, `*.repository.ts`

Описывает форму данных:

```

Generalites { meetingId, structureType, titres: Titre[] }

Titre { id, order, name, sousTitres: SousTitre[] }

SousTitre { id, order, name, commentaires: Commentaire[] }

Commentaire { id, text, createdAt }

```

Т.е. дерево: **титр (A, B...) → sous-titre (1, 2...) → коммент**.

  

Плюс два контракта (abstract class, не interface — так принято в проекте):

- `IGetGeneralitesRepository` — загрузить дерево по `meetingId`

- `ISelectGeneralitesStructureRepository` — создать дерево по выбранному типу

  структуры (`kimanage` или `vide`)

  

### application — 2 юз-кейса

Просто тонкие обёртки над репозиторием (`GetGeneralitesUseCase`,

`SelectGeneralitesStructureUseCase`). На этом этапе бизнес-логики почти нет —

она появится в следующих тикетах (add titre/sous-titre/commentaire).

  

### infrastructure — стор + мок-репозиторий

- **`LocalGeneralitesRepository`** — реального API для généralités ещё нет,

  поэтому дерево хранится просто в `Map` в памяти на время сессии. Когда

  бэкенд появится — этот файл меняется на HTTP-версию, всё остальное не

  трогается (тот же паттерн, что уже используется для `LocalSeanceAttachmentsRepository`).

- **`GeneralitesStore`** (`@ngrx/signals`) — держит состояние страницы

  (`generalites`, `meetingId`) и слушает события (`opened`, `mutated`),

  подгружая дерево через юз-кейс.

- **`generalitesPageEvents`** — событийная шина (`opened` при заходе на

  страницу, `mutated` когда дерево поменялось), стор реагирует на них.

  

### presentation — страница + модалка

- `GeneralitesPageComponent` — сама страница. При заходе диспатчит

  `opened(meetingId)`. Если у стора после загрузки `generalites === null`

  (дерева ещё нет) — сама открывает модалку выбора структуры.

- `SelectStructureModalComponent` — радиокнопки "Kimanage" / "Vide" (Vide

  задизейблена — PRO-фича, механизма тарифов в проекте пока нет вообще).

  После выбора вызывает `SelectGeneralitesStructureUseCase`, закрывает

  модалку с результатом.

  

## 3. Как это подключено к роутингу

  

В `shell/lib.routes.ts` добавлен путь `meeting/:meetingId/generalites`,

рядом с уже существующим `task`. Заходит на страницу через новый пункт

"Généralités" в выпадающем меню строки таблицы PV

(`meetings-table.component.html`).

  

## 4. Неочевидный технический момент (главное, что стоит знать)

  

**Почему `LocalGeneralitesRepository` объявлен как `providedIn: 'root'`,

а не просто передан в провайдеры роута.**

  

Модалка выбора структуры открывается через Angular CDK `Dialog.open()`.

Особенность CDK Dialog: он создаёт компонент модалки в инжекторе **приложения

(app-root)**, а не в инжекторе текущего роута. Если бы репозиторий был

объявлен только на уровне роута (`provideGeneralitesDeps()`), то:

- у страницы был бы один инстанс репозитория (с данными),

- у модалки — другой, пустой (т.к. CDK её открыл в другом инжекторе).

  

Результат: модалка бы писала данные "в никуда", а страница их не видела бы.

Решение — сделать сам класс `providedIn: 'root'` (singleton на всё

приложение), а в провайдерах роута и модалки только алиасить интерфейс на

этот singleton (`useExisting`), не пересоздавая класс. Так и страница, и

модалка видят одно и то же дерево.

  

Это задокументировано прямо в коде (doc-комментарий на классе), и стоит

проверять этот же капкан при добавлении модалок add titre/sous-titre/commentaire

в следующих тикетах.

  

## 5. Правки, сделанные при ревью этого МР

  

1. **Добавлено правило границ модулей** в `eslint.base.config.mjs`:

   `scope:generalites` теперь явно ограничен в том, что может импортировать

   (design-system, core, base, meeting) — раньше такого правила не было

   вообще (у всех остальных модулей — client/business/location/meeting/task/...

   — оно есть, у generalites не завели). Без этого границы модуля не

   enforce'ились линтером.

  

2. **Убран вызов метода из шаблона** (`{{ titreLetter(i) }}` →

   `{{ i | titreLetter }}` через pure pipe). Это конкретный антипаттерн,

   который ранее словили в код-ревью на другой задаче (project-list):

   вызов метода прямо в биндинге шаблона пересчитывается на каждом цикле

   change detection Angular, а pure pipe — только когда реально меняется

   аргумент. См. правило `@angular-eslint/template/no-call-expression`.

  

## 6. Что сознательно НЕ сделано в этом МР (следующие тикеты)

  

- Добавление titre / sous-titre / commentaire — кнопки "+ Ajouter..." и

  кебаб-меню (⋮) на странице визуальные, без логики.

- Реальный HTTP-бэкенд — всё в памяти до появления API.

- Структура "Vide" — просто задизейбленная радиокнопка, тарифов/подписок в

  проекте пока не существует как концепции.
### Template for new entries