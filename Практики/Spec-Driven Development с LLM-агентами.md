# Spec-Driven Development с LLM-агентами

> Расширение GSD-workflow: 10-step TDD-first цикл на каждый модуль, review gates, артефакты в `.planning/`. Интегрируется через CLAUDE.md + slash commands + GSD native commands. ([New Stack](https://thenewstack.io/claude-code-and-the-art-of-test-driven-development/) | [GitHub Blog](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/))

Назад: [[README]] | Папка: `Практики/`

> **Эволюция:** Этот подход развился в [[Supervised Agentic Development]] -- unified framework на базе SDD + [[Superpowers]] + [[GSD (Get Shit Done)|GSD]]. Данная страница сохранена как reference.

---

## Проблема: почему "все тесты вперёд" не работает

Наивный TDD-подход с LLM-агентами: Phase 1 = все тесты для milestone, Phase 2..N = имплементация. Это waterfall-тесты:

- Вся бизнес-логика должна быть известна заранее -- нереалистично для сложных фич
- Реализация модуля A влияет на интерфейс модуля B -- тесты для B устаревают до написания кода
- Edge cases всплывают при имплементации, а не при планировании
- Агент может нагаллюцинировать несуществующие сценарии или пропустить критичные бизнес-правила

Решение: **TDD per module** -- каждый модуль проходит полный цикл discuss-test-implement-review. Знания от предыдущего модуля переносятся в следующий.

---

## Идея: тесты как спецификация, модуль за модулем

Тесты -- лучший формат спецификации для LLM-агента. Они однозначные, верифицируемые и machine-readable. Агент не может "галлюцинировать" прохождение теста ([New Stack](https://thenewstack.io/claude-code-and-the-art-of-test-driven-development/) | [rchavesferna](https://rchavesferna.medium.com/the-complete-guide-for-tdd-with-llms-1dfea9041998)).

Ключевой принцип: **один GSD-phase = один модуль, внутри -- 10-step цикл** с явными точками обсуждения и review. Тесты пишутся только после того, как бизнес-логика модуля полностью обсуждена. Имплементация начинается только после review тестов.

Разработчик направляет *что* тестировать и *какие* бизнес-правила, агент реализует *как*.

---

## 10-step workflow

Каждый GSD-phase (= один модуль) проходит 10 шагов. Четыре из них -- GSD native commands, шесть -- кастомные slash commands.

| # | Шаг | Команда | Артефакт | Требует |
|---|------|---------|----------|---------|
| 1 | Discuss requirements | `/gsd:discuss-phase` | `CONTEXT.md` | -- |
| 2 | Discuss tests | `/sdd:discuss-tests` | `TEST-SPEC.md` | CONTEXT.md |
| 3 | Plan tests | `/sdd:plan-tests` | `TEST-PLAN.md` | TEST-SPEC.md |
| 4 | Review test plan | `/sdd:review-test-plan` | `TEST-PLAN-REVIEW.md` | TEST-PLAN.md |
| 5 | Execute tests | `/sdd:execute-tests` | код (RED) + коммит | TEST-PLAN-REVIEW.md |
| 6 | Review tests | `/sdd:review-tests` | `TEST-REVIEW.md` | test files |
| 7 | Plan implementation | `/gsd:plan-phase` | `XX-PLAN.md` | TEST-REVIEW.md |
| 8 | Execute implementation | `/gsd:execute-phase` | код (GREEN) + коммит | PLAN.md |
| 9 | Verify | `/gsd:verify-work` | `VERIFICATION.md` | implementation |
| 10 | Review implementation | `/sdd:review-impl` | `IMPL-REVIEW.md` | VERIFICATION.md |

### Как это работает

**CLAUDE.md как контроллер.** Содержит таблицу sequence и одно правило: перед выполнением любой команды агент проверяет какие артефакты уже существуют в `.planning/phases/{phase}/`. Если предыдущий шаг не завершён -- предупреждает и предлагает правильную команду.

**Артефакты как state.** Нет отдельного STATUS.md или чеклиста. Наличие файлов в папке фазы -- единственный источник правды о текущем шаге. Агент определяет позицию по файлам.

**GSD + custom.** GSD native commands (discuss, plan, execute, verify) работают как обычно. Кастомные commands вклиниваются между ними и создают дополнительные артефакты в той же папке фазы.

### Пример: папка фазы после завершения

```
.planning/phases/03-auth-module/
  03-CONTEXT.md              -- шаг 1: discuss requirements (GSD)
  03-TEST-SPEC.md            -- шаг 2: discuss tests
  03-TEST-PLAN.md            -- шаг 3: plan tests
  03-TEST-PLAN-REVIEW.md     -- шаг 4: review test plan
  03-TEST-REVIEW.md          -- шаг 6: review tests
  03-RESEARCH.md             -- GSD research (между шагами 6-7)
  03-01-PLAN.md              -- шаг 7: plan implementation (GSD)
  03-01-SUMMARY.md           -- шаг 8: execute implementation (GSD)
  03-VERIFICATION.md         -- шаг 9: verify (GSD)
  03-IMPL-REVIEW.md          -- шаг 10: review implementation
```

---

## Что происходит на каждом шаге

### Шаг 1: Discuss requirements (`/gsd:discuss-phase`)

GSD native. Обсуждение бизнес-логики, функциональных и нефункциональных требований, границ модуля, архитектурных решений. Результат -- `CONTEXT.md` с секциями decisions, specifics, deferred.

### Шаг 2: Discuss tests (`/sdd:discuss-tests`)

Агент читает CONTEXT.md и предлагает что покрыть тестами. Обсуждение с разработчиком:
- Какие сценарии критичны?
- Какие edge cases для каждого входа/состояния?
- Что должно происходить при ошибках?
- Есть ли бизнес-правила, которые не очевидны из кода?

Результат -- `TEST-SPEC.md`: список сценариев, edge cases, приоритеты. Это спецификация *что* тестировать, не *как*.

### Шаг 3: Plan tests (`/sdd:plan-tests`)

Агент преобразует TEST-SPEC.md в конкретный план: файлы тестов, структура describe/it, моки, фикстуры. Это план *как* тестировать.

Результат -- `TEST-PLAN.md`: таблица тестов с модулем, сценарием, типом, приоритетом + структура файлов.

### Шаг 4: Review test plan (`/sdd:review-test-plan`)

Разработчик и агент ревьюят план тестов. Агент проверяет:
- Покрытие всех сценариев из TEST-SPEC.md
- Тесты проверяют поведение, а не детали реализации
- Нет пропущенных edge cases
- Приоритеты корректны

Обсуждение, правки. Результат -- `TEST-PLAN-REVIEW.md`: замечания, решения, финальный OK.

### Шаг 5: Execute tests (`/sdd:execute-tests`)

Агент пишет тесты по утверждённому плану. Все тесты RED (failing). Коммит: `test({module}): add failing tests`.

### Шаг 6: Review tests (`/sdd:review-tests`)

Ревью кода тестов (не плана, а написанного кода). Разработчик и агент проверяют:
- Код соответствует плану
- Assertions содержательные
- Нет тестов на детали реализации
- Тесты читаемые и поддерживаемые

Обсуждение, правки. Результат -- `TEST-REVIEW.md`.

### Шаг 7: Plan implementation (`/gsd:plan-phase`)

GSD native. Агент планирует имплементацию: архитектура, порядок, зависимости, waves. Результат -- `XX-PLAN.md`.

### Шаг 8: Execute implementation (`/gsd:execute-phase`)

GSD native. Агент реализует код. Тесты переходят из RED в GREEN. Коммит после каждого wave.

### Шаг 9: Verify (`/gsd:verify-work`)

GSD native. Автоматическая верификация: прогон тестов, проверка coverage, доработки если что-то RED. Результат -- `VERIFICATION.md`.

### Шаг 10: Review implementation (`/sdd:review-impl`)

Ревью кода реализации. Агент проверяет diff по чеклисту (correctness, security, code quality, breaking changes). Обсуждение, правки. Результат -- `IMPL-REVIEW.md`.

---

## Workflow enforcement

CLAUDE.md содержит strict sequence rule. Агент при каждом вызове команды:

1. Читает список файлов в `.planning/phases/{phase}/`
2. Определяет текущий шаг по наличию артефактов
3. Если вызванная команда не соответствует текущему шагу -- предупреждает
4. Предлагает правильную следующую команду

Пример: разработчик вызывает `/gsd:plan-phase` (шаг 7), но `TEST-REVIEW.md` не существует. Агент:

> "Шаг 6 (review тестов) не завершён -- TEST-REVIEW.md отсутствует. Запустите `/sdd:review-tests` или введите 'skip' для пропуска."

---

## Quick task workflow

Для `/gsd:quick` -- сокращённый протокол без отдельных артефактов:

```
/gsd:quick --discuss
    |
[агент обсуждает требования + тесты: 2-3 вопроса max]
    |
[агент пишет тесты -> RED -> имплементация -> GREEN]
    |
/sdd:review-impl
    |
done
```

Quick tasks не создают отдельные файлы TEST-SPEC.md и т.д. -- всё происходит в одном разговоре. Но правило "тесты до имплементации" сохраняется.

---

## Что меняется при смене GSD на другой инструмент

| Компонент | При миграции |
|---|---|
| CLAUDE.md sequence table | Заменить GSD-команды (шаги 1, 7, 8, 9) на аналоги нового инструмента |
| `/sdd:*` commands | Не меняется -- не зависят от GSD |
| Артефакты | Путь к папке фазы может измениться |
| Enforcement logic | Обновить правило проверки артефактов в CLAUDE.md |

Вся GSD-специфика -- в 4 шагах из 10. Остальные 6 шагов tool-agnostic.

---

## Источники по теме

| Тема | Источник |
|---|---|
| TDD + Claude Code | [New Stack](https://thenewstack.io/claude-code-and-the-art-of-test-driven-development/) |
| Red-Green-Refactor | [alexop.dev](https://alexop.dev/posts/custom-tdd-workflow-claude-code-vue/) |
| TDD guide for LLMs | [rchavesferna](https://rchavesferna.medium.com/the-complete-guide-for-tdd-with-llms-1dfea9041998) |
| Spec-driven dev | [GitHub Blog](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/) |
| Reflexion pattern | [promptingguide.ai](https://www.promptingguide.ai/techniques/reflexion) |
| Agentic coding patterns | [CodeScene](https://codescene.com/blog/agentic-ai-coding-best-practice-patterns-for-speed-with-quality) |
| LLM coding workflow | [Addy Osmani](https://medium.com/@addyosmani/my-llm-coding-workflow-going-into-2026-52fe1681325e) |

---

Связанные: [[Рабочие воркфлоу с LLM-агентами]], [[GSD (Get Shit Done)]], [[Claude Code]], [[Prompt Engineering для разработки]], [[CLAUDE.md шаблон -- Spec-Driven]]