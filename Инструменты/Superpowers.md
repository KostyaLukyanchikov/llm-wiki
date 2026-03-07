# Superpowers

> Composable skills framework для структурированного coding workflow -- превращает хаотичное AI-кодирование в дисциплинированный процесс с фазами design, planning, execution, testing, review. ([GitHub](https://github.com/obra/superpowers) | [Blog](https://blog.fsck.com/2025/10/09/superpowers/))

Назад: [[README]] | Папка: `Инструменты/`

Используется как источник best practices для [[Supervised Agentic Development]].

---

## Что это

Самый популярный agentic skills framework (73k stars, v4.1.1). Набор composable skills + mandatory workflows, которые автоматически активируются при работе с coding-агентом. Работает как plugin для [[Claude Code]], Cursor, Codex, OpenCode.

Ключевое отличие от обычных CLAUDE.md-инструкций: skills -- это не советы, а mandatory workflows. Агент проверяет релевантные skills перед каждой задачей и активирует их автоматически.

---

## Как работает -- 7 фаз

| Фаза | Skill | Что делает |
|---|---|---|
| 1. Brainstorming | `brainstorming` | Уточняющие вопросы, Socratic design refinement, презентация спеки по частям для approval |
| 2. Worktree | `using-git-worktrees` | Создание изолированного workspace на новой ветке, проверка чистого test baseline |
| 3. Planning | `writing-plans` | Детальный план с задачами по 2-5 минут, точные пути файлов, полный код в плане |
| 4. Execution | `subagent-driven-development` или `executing-plans` | Субагент на каждую задачу + двухэтапное review (spec compliance, затем code quality) |
| 5. TDD | `test-driven-development` | RED-GREEN-REFACTOR на каждом шаге, Iron Law |
| 6. Code Review | `requesting-code-review` | Review между задачами, critical issues блокируют прогресс |
| 7. Finish | `finishing-a-development-branch` | Верификация тестов, merge/PR/keep/discard, cleanup worktree |

---

## Ключевые skills

### TDD и Iron Law ([SKILL.md](https://github.com/obra/superpowers/blob/main/skills/test-driven-development/SKILL.md))

Центральный принцип: `NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST`. Написал код до теста -- удали его. Не "адаптируй", не "используй как reference" -- удали и начни заново.

RED-GREEN-REFACTOR цикл с обязательной верификацией каждого шага:
- **RED** -- один минимальный тест, показывающий ожидаемое поведение
- **Verify RED** -- запустить, убедиться что падает правильно (MANDATORY, never skip)
- **GREEN** -- минимальный код для прохождения теста
- **Verify GREEN** -- все тесты зелёные
- **REFACTOR** -- cleanup, убедиться что остаётся зелёным

### Subagent-Driven Development ([SKILL.md](https://github.com/obra/superpowers/blob/main/skills/subagent-driven-development/SKILL.md))

Свежий субагент на каждую задачу (нет загрязнения контекста) + трёхступенчатое review:
1. Implementer субагент -- имплементация, тесты, self-review, коммит
2. Spec reviewer субагент -- проверяет соответствие спецификации
3. Code quality reviewer субагент -- проверяет качество кода

При провале review -- implementer фиксит и проходит review заново.

### Writing Plans ([SKILL.md](https://github.com/obra/superpowers/blob/main/skills/writing-plans/SKILL.md))

Планы пишутся в расчёте на "enthusiastic junior engineer with poor taste, no judgement, no project context". Каждая задача -- 2-5 минут. Включает:
- Точные пути файлов (create/modify/test)
- Полный код (не "add validation", а реальный код)
- Точные команды запуска с ожидаемым результатом
- Шаги коммита

### Другие skills

| Skill | Назначение |
|---|---|
| `systematic-debugging` | 4-фазный root cause анализ: reproduce, isolate, fix, verify |
| `brainstorming` | Socratic design -- вопросы, альтернативы, секционная валидация |
| `dispatching-parallel-agents` | Параллельные субагенты для независимых задач |
| `verification-before-completion` | Проверка что баг реально пофикшен, а не замаскирован |
| `writing-skills` | Meta-skill для создания новых skills |

---

## Сравнение с SDD + GSD

### Общее

| Аспект | Superpowers | [[Spec-Driven Development с LLM-агентами\|SDD]] + [[GSD (Get Shit Done)\|GSD]] |
|---|---|---|
| TDD-first | Iron Law: код без теста = удалить | 10-step цикл, тесты как спецификация |
| Фазы design -> plan -> execute | 7 фаз с mandatory skills | discuss -> test -> implement -> review |
| Субагенты для execution | subagent-driven-development | GSD executor agents |
| Code review gates | Двухэтапное review (spec + quality) | sdd-review-tests + sdd-review-impl |
| Планы с точными файлами | Пути + полный код в плане | PLAN.md с UAT criteria |
| Git isolation | Worktrees | GSD worktree support |
| Верификация завершённости | verification-before-completion | /gsd:verify-work |

### Различия

| Аспект | Superpowers | SDD + GSD |
|---|---|---|
| Архитектура | Composable skills в CLAUDE.md, plugin | Slash commands + PROJECT.md + .planning/ |
| Гранулярность планов | 2-5 мин задачи с полным кодом | Phase-level планы с UAT |
| Тесты | "Написал код до теста = удали" | Отдельные фазы discuss-tests и plan-tests |
| Review модель | Субагент-reviewer автоматически | Человек ревьюит артефакты на каждом шаге |
| Контроль человека | Approve design, затем автономия часами | Checkpoint на каждом из 10 шагов |
| Обсуждение требований | Brainstorming skill | /gsd:discuss-phase + /project:sdd-discuss-tests |
| State management | Нет -- stateless skills | .planning/ директория, STATE.md, TODO.md |
| Отладка | systematic-debugging skill | /gsd:debug с persistent state |

---

Связанные: [[Supervised Agentic Development]], [[GSD (Get Shit Done)]], [[Claude Code]], [[Spec-Driven Development с LLM-агентами]], [[Awesome Claude Code]]