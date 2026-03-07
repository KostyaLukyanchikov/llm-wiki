# Supervised Agentic Development (SAD)

> Unified framework для структурированной разработки с LLM-агентами: 10-step TDD-first workflow с checkpoint на каждом шаге, cross-cutting behavioral rules, project management backbone. Объединяет лучшее из [[Superpowers]] + [[GSD (Get Shit Done)|GSD]] + [[Spec-Driven Development с LLM-агентами|SDD]].

Назад: [[README]] | Папка: `Практики/`

---

## Откуда выросло

SAD -- результат эволюции трёх подходов:

| Источник | Что взяли |
|---|---|
| [[Superpowers]] ([GitHub](https://github.com/obra/superpowers)) | Behavioral rules: Iron Law TDD, verification-before-completion, systematic debugging, HARD-GATE on design |
| [[GSD (Get Shit Done)]] ([GitHub](https://github.com/gsd-build/get-shit-done)) | Project management backbone: milestones, roadmap, wave execution, state management в `.planning/` |
| [[Spec-Driven Development с LLM-агентами\|SDD]] | Test-first per-module workflow: 10-step цикл, тесты как спецификация, артефакты как state |

---

## Философия -- 5 принципов

### 1. Evidence over claims

Никаких "should work", "probably fixed", "seems correct". Только доказательства: вывод команды, exit code, test results. Каждое утверждение о работоспособности кода подкреплено свежим запуском ([Superpowers: verification-before-completion](https://github.com/obra/superpowers/blob/main/skills/verification-before-completion/SKILL.md)).

### 2. Tests as specification

Тесты -- лучший формат спецификации для LLM-агента. Они однозначные, верифицируемые и machine-readable. Агент не может "галлюцинировать" прохождение теста. Каждый модуль проходит полный цикл discuss-test-implement-review ([New Stack](https://thenewstack.io/claude-code-and-the-art-of-test-driven-development/)).

### 3. Human at every gate

Checkpoint на каждом из 10 шагов. Агент предлагает, человек подтверждает. Для проектов с нетривиальной бизнес-логикой частый checkpoint ценнее скорости автономного выполнения.

### 4. Fresh context per task

Изолированные контексты по 200K, без context rot. Каждый субагент получает чистый контекст с минимально необходимой информацией. Артефакты в `.planning/` -- мост между контекстами ([GSD: wave execution](https://github.com/gsd-build/get-shit-done)).

### 5. Root cause before fixes

Если 3 фикса подряд не решают проблему -- это архитектурная проблема, а не баг. Остановись, обсуди с человеком ([Superpowers: systematic-debugging](https://github.com/obra/superpowers/blob/main/skills/systematic-debugging/SKILL.md)).

---

## 10-step workflow

Каждый GSD-phase (= один модуль) проходит 10 шагов. Четыре -- GSD native commands, шесть -- кастомные slash commands.

| # | Шаг | Команда | Артефакт | Что нового vs SDD |
|---|------|---------|----------|--------------------|
| 1 | Design & Requirements | `/gsd:discuss-phase` | `CONTEXT.md` | **HARD-GATE**: нельзя перейти к шагу 2 без approved design. Socratic questioning, 2-3 подхода с tradeoffs |
| 2 | Discuss tests | `/project:sad-discuss-tests` | `TEST-SPEC.md` | Явное требование: тесты проверяют *поведение*, не детали реализации |
| 3 | Plan tests | `/project:sad-plan-tests` | `TEST-PLAN.md` | **Полный код тестов в плане** (не только describe/it имена) |
| 4 | Review test plan | `/project:sad-review-test-plan` | `TEST-PLAN-REVIEW.md` | **Two-pass review**: покрытие спеки + качество тестов |
| 5 | Execute tests (RED) | `/project:sad-execute-tests` | код (RED) + коммит | **Iron Law**: удалить impl код. **Verify RED**: обязательная проверка причины падения |
| 6 | Review tests | `/project:sad-review-tests` | `TEST-REVIEW.md` | Verify RED confirmation включён в review |
| 7 | Plan implementation | `/gsd:plan-phase` | `XX-PLAN.md` | **Полный код в плане** (Superpowers style), задачи по 2-5 минут, точные команды |
| 8 | Execute implementation | `/gsd:execute-phase` | код (GREEN) + коммит | **3-fix escalation**: 3+ неудачных фикса = стоп. **Verification-before-completion** |
| 9 | Verify | `/gsd:verify-work` | `VERIFICATION.md` | **Goal-backward + evidence**: stub detection, fresh verification evidence |
| 10 | Review implementation | `/project:sad-review-impl` | `IMPL-REVIEW.md` | **Two-stage review**: (a) spec compliance, (b) code quality. Severity levels |

### Пример: папка фазы после завершения

```
.planning/phases/03-auth-module/
  03-CONTEXT.md              -- шаг 1
  03-TEST-SPEC.md            -- шаг 2
  03-TEST-PLAN.md            -- шаг 3
  03-TEST-PLAN-REVIEW.md     -- шаг 4
  03-TEST-REVIEW.md          -- шаг 6
  03-01-PLAN.md              -- шаг 7
  03-01-SUMMARY.md           -- шаг 8
  03-VERIFICATION.md         -- шаг 9
  03-IMPL-REVIEW.md          -- шаг 10
```

---

## Cross-cutting rules

Применяются ко ВСЕМ шагам workflow.

### 1. Iron Law (из Superpowers TDD)

`NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST`

Написал код до теста? Удали его. Не "используй как reference", не "адаптируй" -- удали и начни заново. Это не рекомендация, а обязательное правило ([SKILL.md](https://github.com/obra/superpowers/blob/main/skills/test-driven-development/SKILL.md)).

### 2. Verify RED (из Superpowers)

После написания теста -- запусти его и убедись что он падает *по правильной причине*. Не просто "тест fails", а "fails because function doesn't exist yet" vs "fails because of import error" vs "fails because of syntax error". MANDATORY, never skip.

Формат отчёта: `"{N} tests, all RED, failure reasons verified, confidence {X}/10"`

### 3. Verification-before-completion (из Superpowers)

Перед ЛЮБЫМ утверждением об успехе:
1. Определи команду верификации
2. Запусти её ЗАНОВО (не используй кешированный результат)
3. Прочитай ПОЛНЫЙ вывод
4. Проверь exit code

Только после этого утверждай успех. Слова "should", "probably", "seems to" = нарушение ([SKILL.md](https://github.com/obra/superpowers/blob/main/skills/verification-before-completion/SKILL.md)).

### 4. 3-fix escalation (из Superpowers systematic-debugging)

Если 3+ попыток исправления провалились, каждая вскрывая новую проблему в другом месте = архитектурная проблема. STOP. Обсуди с человеком. Не предпринимай Fix #4.

### 5. HARD-GATE on design (из Superpowers brainstorming)

Нельзя пропустить шаг 1. Нельзя перейти к шагу 2+ без утверждённого design/requirements. Даже "простые" задачи требуют подтверждения дизайна ([SKILL.md](https://github.com/obra/superpowers/blob/main/skills/brainstorming/SKILL.md)).

### 6. Artifact-based enforcement (из SDD)

Агент проверяет наличие артефактов в `.planning/phases/{phase}/` перед каждым шагом. Отсутствует prerequisite -- предупреждение + предложение правильной команды.

---

## Quick task workflow

Для `/gsd:quick` -- сокращённый протокол без отдельных артефактов:

```
/gsd:quick --discuss
    |
[агент обсуждает требования + тесты: 2-3 вопроса max]
    |
[агент пишет тесты -> RED -> Verify RED -> имплементация -> GREEN]
    |
[verification-before-completion: прогон тестов, проверка вывода]
    |
/project:sad-review-impl
    |
done
```

Quick tasks не создают отдельные файлы TEST-SPEC.md и т.д. -- всё происходит в одном разговоре. Но три правила сохраняются:
1. Тесты до имплементации (Iron Law)
2. Verify RED на каждом тесте
3. Verification-before-completion перед утверждением успеха

---

## Что нового по сравнению с SDD

| Аспект | SDD | SAD |
|---|---|---|
| Design gate | Обычный discuss | HARD-GATE: нельзя пропустить даже для "простых" задач |
| Test plan | describe/it имена | Полный код тестов в плане |
| RED verification | "Все тесты RED" | Verify RED: проверка *причины* падения каждого теста |
| Test plan review | Один проход | Two-pass: покрытие спеки + качество тестов |
| Impl plan | GSD native PLAN.md | Полный код в плане, задачи по 2-5 минут |
| Debugging | Нет explicit правила | 3-fix escalation: 3+ фиксов = стоп |
| Completion claims | Trust but verify | Verification-before-completion: запусти и докажи |
| Impl review | Один чеклист | Two-stage: spec compliance + code quality. Severity levels: CRITICAL/WARNING/SUGGESTION |
| Cross-cutting rules | Нет | 6 правил, применяемых ко всем шагам |

---

## Шаблон и slash commands

Готовый набор файлов для внедрения SAD: CLAUDE.md template + 6 slash commands -- в [[CLAUDE.md шаблон -- SAD]].

---

Связанные: [[Spec-Driven Development с LLM-агентами]], [[CLAUDE.md шаблон -- SAD]], [[Superpowers]], [[GSD (Get Shit Done)]], [[Claude Code]]