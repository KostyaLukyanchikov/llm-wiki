# GSD (Get Shit Done)

> Система meta-prompting и context engineering для [[Claude Code]]. Структурированный workflow от идеи до production, решающий проблему "context rot". ([GitHub](https://github.com/gsd-build/get-shit-done))

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

GSD решает главную проблему LLM-агентов -- деградацию качества по мере заполнения context window ("context rot"). Делает это через:
- Сегментацию задач на атомарные шаги
- Изолированные контексты по 200k tokens для каждой задачи
- Явное планирование и непрерывную верификацию

---

## 5 стадий workflow

```
Initialize --> Discuss --> Plan --> Execute --> Verify
```

1. **Initialize** (`/gsd:new-project`) -- создание PROJECT.md, REQUIREMENTS.md, ROADMAP.md
2. **Discuss** (`/gsd:discuss-phase N`) -- обсуждение подхода перед планированием
3. **Plan** (`/gsd:plan-phase N`) -- XML-structured plans с задачами
4. **Execute** (`/gsd:execute-phase N`) -- wave-based parallel execution
5. **Verify** (`/gsd:verify-work N`) -- user acceptance testing

---

## Ключевые фичи

- **Wave-based parallel execution** -- зависимости маппятся, независимые задачи запускаются параллельно
- **Atomic git commits** -- каждая задача = свой коммит с semantic versioning
- **Multi-agent orchestration** -- 4 параллельных research-агента, planner + verifier loops
- **Quick mode** (`/gsd:quick`) -- для ad-hoc задач без полного цикла
- **Context files**: PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md

---

## Установка

```bash
npx get-shit-done-cc@latest
```

Installer спросит:
1. Runtime: Claude Code, OpenCode, Gemini CLI, Codex или все
2. Location: Global (все проекты) или local (текущий)

---

## Основные команды

| Команда | Назначение |
|---|---|
| `/gsd:new-project` | Инициализация нового проекта |
| `/gsd:discuss-phase N` | Обсуждение фазы перед планированием |
| `/gsd:plan-phase N` | Создание плана задач |
| `/gsd:execute-phase N` | Выполнение с параллельными волнами |
| `/gsd:verify-work N` | User acceptance testing |
| `/gsd:quick` | Быстрый режим для мелких задач |
| `/gsd:map-codebase` | Анализ существующей кодовой базы |
| `/gsd:progress` | Текущий статус |
| `/gsd:new-milestone` | Следующая версия |
| `/gsd:debug` | Структурированный дебаг с persistent state |

---

## Практические советы

- Используй `/gsd:map-codebase` перед стартом на существующей кодовой базе
- `/gsd:quick` идеален для bug fixes -- пропускает research и verification
- STATE.md отслеживает решения и блокеры между сессиями
- Для frictionless работы: настрой granular permissions в `.claude/settings.json`

---

## Когда использовать

| Сценарий | GSD? |
|---|---|
| Новый проект с нуля | `/gsd:new-project` |
| Большая фича (3+ файлов) | `/gsd:plan-phase` + `/gsd:execute-phase` |
| Быстрый баг-фикс | `/gsd:quick` |
| Исследование кодовой базы | `/gsd:map-codebase` |
| Однострочный фикс | Нет, обычный Claude Code |

---

Связанные: [[Claude Code]], [[Рабочие воркфлоу с LLM-агентами]], [[Superpowers]], [[Spec-Driven Development с LLM-агентами]]
