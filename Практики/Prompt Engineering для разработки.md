# Prompt Engineering для разработки

> Принципы промптинга, настройка CLAUDE.md, управление контекстом и примеры эффективных промптов.

Назад: [[README]] | Папка: `Практики/` | Связанные: [[Рабочие воркфлоу с LLM-агентами]], [[Claude Code]]

---

## 1. Основные принципы промптинга для coding tasks ([Playbook](https://addyo.substack.com/p/the-prompt-engineering-playbook-for))

### 1.1 Контекст решает всё

LLM не знает ничего о твоём проекте. Каждый промпт должен содержать достаточно информации, чтобы ответ был релевантным. Три кита хорошего промпта:

- **WHAT** -- что за проект, стек, структура
- **WHY** -- зачем нужно это изменение, какая бизнес-задача
- **HOW** -- как именно ты хочешь это сделать, какие ограничения

### 1.2 Specificity over brevity

Вместо "почему не работает?" ---> "Функция `processOrder` в `src/services/order.ts` возвращает `undefined` вместо объекта `OrderResult`. Вот код: [...]. Вот error trace: [...]. Ожидаемый результат: [...]."

**Правило**: если твой промпт применим к тысяче других проектов -- он слишком расплывчатый.

### 1.3 Декомпозиция

Одна задача -- один промпт. Не "исправь 5 багов и добавь 3 фичи". Разбивай на шаги:

1. Сначала plan -- обсуди подход
2. Потом implementation -- по одному шагу
3. Потом review -- проверь результат

### 1.4 Few-shot примеры

Показывай модели, чего ты хочешь, через примеры input/output:

```
Преобразуй эти API endpoints в TypeScript интерфейсы:

Пример:
GET /users/:id -> interface GetUserParams { id: string }
POST /users body: {name, email} -> interface CreateUserBody { name: string; email: string }

Теперь сделай для:
GET /orders/:id/items?page=N&limit=N
POST /orders body: {userId, items: [{productId, quantity}]}
```

### 1.5 Role-play для глубины анализа

Назначение роли меняет "глубину" ответа:

- `Выступи как senior code reviewer` -- детальный анализ кода
- `Ты security-аудитор` -- фокус на уязвимостях
- `Ты performance engineer` -- фокус на оптимизации

Это не магия -- это просто способ активировать нужный "режим" генерации.

### 1.6 Итеративность

Промптинг -- это диалог, а не one-shot. Стратегия:
1. Получил результат
2. Определил, что не так
3. Уточнил конкретно: "Важно: используй `async/await` вместо callbacks" или "Note: нужна обратная совместимость с v2 API"
4. Если контекст стал мутным -- `/clear` и начни заново с чистым промптом

---

## 2. CLAUDE.md -- настройка и best practices

### 2.1 Что это ([best practices](https://code.claude.com/docs/en/best-practices) | [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) | [Arize](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/))

`CLAUDE.md` -- единственный файл, который по умолчанию попадает в каждую сессию Claude Code. Это "онбординг" агента в твой проект. Агент stateless -- между сессиями он ничего не помнит, и CLAUDE.md это компенсирует.

### 2.2 Структура

Три обязательных блока:

```markdown
# Project: MyApp

## WHAT (стек и структура)
- Next.js 14, TypeScript, Prisma, PostgreSQL
- Monorepo: apps/web, apps/api, packages/shared
- Тесты: vitest для unit, playwright для e2e

## WHY (контекст проекта)
- SaaS платформа для управления заказами
- apps/web -- клиентский dashboard
- apps/api -- REST API + WebSocket для real-time обновлений
- packages/shared -- общие типы и утилиты

## HOW (как работать)
- Запуск: `pnpm dev` из корня
- Тесты: `pnpm test` (unit), `pnpm test:e2e` (e2e)
- Type check: `pnpm typecheck`
- Линтинг: `pnpm lint` (запускай ПЕРЕД коммитом)
- Ветки: feature/TICKET-123-description
- Коммиты: conventional commits (feat:, fix:, refactor:)
```

### 2.3 Размер и дисциплина

- **Максимум 150-200 инструкций** (60-300 строк). System prompt Claude Code уже содержит ~50 инструкций -- каждая твоя сверху ухудшает следование остальным.
- **Тест на необходимость**: для каждой строки спроси -- "если её убрать, Claude будет делать ошибки?" Нет --> удаляй.
- **Универсальность**: только то, что релевантно для КАЖДОЙ сессии. Специфичные инструкции (миграции БД, деплой) -- в отдельные файлы.

### 2.4 Progressive disclosure ([DEV.to](https://dev.to/cleverhoods/claudemd-best-practices-from-basic-to-adaptive-9lm))

Не пихай всё в один файл. Используй структуру:

```
CLAUDE.md                    # корневой, 60-100 строк
agent_docs/
  ├── architecture.md        # архитектура и решения
  ├── testing_guide.md       # как писать тесты
  ├── api_conventions.md     # конвенции API
  └── deployment.md          # процесс деплоя
```

В `CLAUDE.md` укажи:
```markdown
Перед началом работы прочитай релевантный файл из agent_docs/.
Для работы с тестами: agent_docs/testing_guide.md
Для работы с API: agent_docs/api_conventions.md
```

Можно также использовать синтаксис `@agent_docs/testing_guide.md` для импортов.

### 2.5 Чего НЕ включать

- **Code style** -- это работа линтера/форматтера, не LLM. Настрой ESLint/Biome/Prettier.
- **Автогенерированный контент** (`/init`) -- CLAUDE.md влияет на каждый этап работы, поэтому каждая строка должна быть осознанной.
- **Копии кода** -- используй ссылки `file:line`, а не вставки, которые устареют.
- **Нерелевантные инструкции** -- Claude склонен игнорировать то, что не относится к текущей задаче.

### 2.6 Иерархия файлов

```
~/.claude/CLAUDE.md          # глобальный, для всех проектов
~/project/CLAUDE.md          # проектный, шарится через git
~/project/src/api/CLAUDE.md  # для конкретной директории
```

Каждый уровень добавляет контекст при работе в соответствующей директории.

---

## 3. Работа с контекстом

### 3.1 Контекст -- главный ресурс

Context window -- это рабочая память агента. Чем больше мусора, тем хуже качество. Агент начинает пропускать важные детали, "забывать" инструкции, галлюцинировать.

### 3.2 Агрессивная очистка (`/clear`) ([Habr](https://habr.com/ru/articles/971772/))

Главный инсайт из практики: **очищай контекст чаще, чем кажется нужным**.

- Закончил подзадачу --> `/clear`
- Осталось меньше 50% релевантной информации --> `/clear`
- Агент начал "тупить" или повторяться --> `/clear`
- Переключаешься на другую область кода --> `/clear`

Это контринтуитивно -- кажется, что "потеряешь" наработки. Но агент заново подтянет нужное из файлов, и это будет чище, чем деградирующий контекст.

### 3.3 MCP-серверы: скрытый потребитель контекста ([Habr](https://habr.com/ru/articles/971772/))

Описания инструментов MCP-серверов загружаются в каждый запрос и могут занимать до 50% контекста ещё до начала работы.

**Стратегия**:
- Отключай неиспользуемые MCP-серверы (если Jira нужен раз в месяц -- не держи его всегда)
- Используй фильтрацию инструментов, где MCP это поддерживает
- Выноси шумные операции в sub-agents

### 3.4 Skills как lazy-loading инструкций

Skills загружаются только при активации, в отличие от CLAUDE.md который всегда в контексте. Вместо того чтобы раздувать CLAUDE.md правилами миграций, логирования и API-паттернов -- вынеси это в skills.

```markdown
# Skill: database-migration
---
Правила создания миграций Prisma:
1. Всегда создавай reversible миграции
2. Данные мигрируй отдельным шагом
3. Проверяй через: pnpm prisma migrate diff
...
```

### 3.5 Sub-agents для фильтрации output ([docs](https://code.claude.com/docs/ru/sub-agents) | [Habr](https://habr.com/ru/articles/946748/))

Вместо того чтобы заливать основного агента 200 строками тестового output -- делегируй sub-agent'у, который запустит тесты и вернёт только failures с объяснениями.

```yaml
---
name: test-runner
description: 'Run tests and report only failures with analysis'
model: sonnet
---
```

---

## 4. Примеры промптов для типичных задач

### 4.1 Debug

**Плохо:**
```
Почему падает тест?
```

**Хорошо:**
```
Тест `OrderService.createOrder` в tests/services/order.test.ts
падает с ошибкой:

TypeError: Cannot read property 'id' of undefined
  at OrderService.createOrder (src/services/order.ts:45)

Ожидаемое поведение: метод создаёт заказ и возвращает
OrderResult с полем id.

Фактическое: возвращает undefined.

Код метода: [прочитай src/services/order.ts]

Разберись пошагово:
1. Пройди по коду line-by-line
2. Отследи, где теряется значение
3. Предложи фикс с объяснением причины
```

### 4.2 Refactor

**Плохо:**
```
Отрефактори этот файл
```

**Хорошо:**
```
Отрефактори src/api/handlers/orders.ts:

Цели:
1. Убрать дублирование логики валидации (строки 23-45 и 67-89)
2. Вынести бизнес-логику из handler в OrderService
3. Добавить proper error handling вместо try/catch с console.log

Ограничения:
- Сохрани обратную совместимость API (те же endpoints, те же форматы ответов)
- Используй паттерны из существующего UserService как reference
- Покрой изменения тестами

Объясни каждое изменение.
```

### 4.3 New Feature

**Плохо:**
```
Добавь пагинацию
```

**Хорошо:**
```
Нужно добавить cursor-based пагинацию для GET /api/orders.

Контекст:
- Сейчас endpoint возвращает все заказы без лимита
- Таблица orders растёт, уже 100k+ записей
- Фронтенд использует infinite scroll

Требования:
- Cursor-based (не offset), по полю createdAt + id
- Query params: ?cursor=xxx&limit=20
- Response: { data: Order[], nextCursor: string | null, hasMore: boolean }
- Default limit: 20, max limit: 100

Сначала составь план реализации, потом реализуй.
Посмотри как сделана пагинация в src/api/handlers/users.ts
для reference.
```

### 4.4 Code Review

**Плохо:**
```
Посмотри этот PR
```

**Хорошо:**
```
Проведи code review изменений в текущей ветке
(git diff main...HEAD).

Фокус:
1. Корректность логики -- есть ли баги или edge cases?
2. Безопасность -- SQL injection, XSS, auth bypass?
3. Производительность -- N+1 queries, memory leaks,
   неэффективные алгоритмы?
4. Соответствие конвенциям проекта (см. CLAUDE.md)

Формат ответа:
- CRITICAL: баги/уязвимости, которые нужно исправить
- WARNING: потенциальные проблемы
- SUGGESTION: улучшения, не блокирующие merge
- POSITIVE: что сделано хорошо

Не комментируй стилистику -- для этого есть линтер.
```

---

*Все источники указаны inline рядом с соответствующими разделами.*
