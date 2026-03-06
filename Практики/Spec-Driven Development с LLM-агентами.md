# Spec-Driven Development с LLM-агентами

> Расширение GSD-workflow: TDD-first фазы, review gates, auto-PR. Интегрируется через CLAUDE.md + agent_docs/ + slash commands. ([New Stack](https://thenewstack.io/claude-code-and-the-art-of-test-driven-development/) | [GitHub Blog](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/))

Назад: [[README]] | Папка: `Практики/`

---

## Проблема: GSD не покрывает TDD, review и PR

[[GSD (Get Shit Done)]] отлично справляется с планированием и execution, но:
- Тесты пишутся *после* имплементации (`/gsd:add-tests`) или как `type: tdd` внутри execution -- а не как отдельная фаза *до* кода
- Нет code review gate между execution и verification
- Нет auto-PR

Этот подход не заменяет GSD, а **вклинивает дополнительные шаги** между его командами через CLAUDE.md rules, `agent_docs/` и slash commands.

---

## Идея: тесты как спецификация ([New Stack](https://thenewstack.io/claude-code-and-the-art-of-test-driven-development/) | [alexop.dev](https://alexop.dev/posts/custom-tdd-workflow-claude-code-vue/))

Тесты -- лучший формат спецификации для LLM-агента. Они однозначные, верифицируемые и machine-readable. Агент не может "галлюцинировать" прохождение теста.

Ключевой принцип: **тесты пишутся отдельной фазой до имплементации**. Не `type: tdd` внутри execution, а Phase 1 = тесты в roadmap. Это заставляет тесты быть честной спецификацией, а не подгонкой под реализацию ([rchavesferna](https://rchavesferna.medium.com/the-complete-guide-for-tdd-with-llms-1dfea9041998)).

Разработчик и агент вместе продумывают тесты: разработчик описывает сценарии и edge cases, агент задаёт уточняющие вопросы, вместе итерируют до полного покрытия. Только после ревью тестов переходим к имплементации.

---

## Архитектура: как это работает с GSD

### Файловая структура

```
CLAUDE.md                          -- правила workflow + ссылки на agent_docs/
agent_docs/
  tdd-workflow.md                  -- протокол обсуждения тестов, правила фаз
  review-checklist.md              -- чеклист для review gate
.claude/commands/
  review.md                        -- /project:review (self-review diff)
  pr.md                            -- /project:pr (генерация PR)
  validate.md                      -- /project:validate (post-validation)
```

### Принцип: CLAUDE.md компактный, детали в agent_docs/

CLAUDE.md содержит только правила и ссылки. Детальные протоколы -- в `agent_docs/`. GSD-executor и planner читают CLAUDE.md при каждом запуске, поэтому правила применяются автоматически.

```markdown
# В CLAUDE.md:
## TDD-first workflow
Phase 1 of every milestone MUST be tests. See agent_docs/tdd-workflow.md for protocol.
```

Агент читает `tdd-workflow.md` когда начинает планирование, а не загружает его в контекст постоянно.

---

## Workflow: Milestone

```
/gsd:discuss-phase 1
    |
/gsd:plan-phase 1          -- planner создаёт Phase 1 = tests (CLAUDE.md rule)
    |
/gsd:execute-phase 1       -- агент обсуждает с тобой тесты, пишет RED tests
    |
/project:review             -- review тестов перед имплементацией
    |
/gsd:plan-phase 2..N       -- планирование имплементации
    |
/gsd:execute-phase 2..N    -- имплементация, тесты GREEN
    |
/project:review             -- review diff перед verification
    |
/gsd:verify-work            -- UAT
    |
/project:pr                 -- генерация PR
```

### Как CLAUDE.md заставляет planner создавать Phase 1 = тесты

Правило в CLAUDE.md:

```markdown
## Milestone planning rules
When creating a milestone roadmap, ALWAYS structure phases as:
- Phase 1: Tests — write failing tests covering all requirements
- Phase 2..N-1: Implementation — make tests GREEN, module by module
- Phase N: Review + cleanup

Phase 1 tests MUST be discussed with the developer before writing.
Read agent_docs/tdd-workflow.md for the test discussion protocol.
```

GSD planner читает CLAUDE.md и следует этому правилу при создании ROADMAP.md.

### Фаза тестов: протокол обсуждения

Во время `/gsd:execute-phase 1` (тестовая фаза) агент следует протоколу из `agent_docs/tdd-workflow.md`:

1. **Analyse** -- агент читает requirements и код, предлагает список сценариев
2. **Discuss** -- агент задаёт вопросы: "Какие edge cases для X?", "Что должно происходить при Y?"
3. **Agree** -- разработчик подтверждает список сценариев
4. **Write** -- агент пишет тесты, все RED
5. **Validate** -- агент перечитывает тесты, проверяет покрытие, confidence score

Разработчик направляет *что* тестировать, агент реализует *как*.

---

## Workflow: Quick task

```
/gsd:quick --discuss
    |
[агент обсуждает тесты по сокращённому протоколу]
    |
[агент пишет тесты → RED → имплементация → GREEN]
    |
/project:review
    |
done (или /project:pr если нужно)
```

CLAUDE.md rule для quick tasks:

```markdown
## Quick task rules
For /gsd:quick, ALWAYS write tests before implementation:
1. Discuss test scenarios with developer (abbreviated — 2-3 questions max)
2. Write failing tests
3. Implement until GREEN
4. Show diff for review
Read agent_docs/tdd-workflow.md § Quick tasks for details.
```

---

## Workflow enforcement ([Reflexion](https://www.promptingguide.ai/techniques/reflexion))

CLAUDE.md содержит strict sequence rule:

```markdown
## Workflow sequence (STRICT)
Milestone: discuss → plan → execute(tests) → /project:review → execute(impl) → /project:review → verify → /project:pr
Quick: discuss → tests → implement → /project:review → done

If a step is skipped, WARN the developer and require explicit confirmation.
Example: "Review step was skipped. Type 'skip review' to confirm, or run /project:review."
```

Агент видит это правило при каждом запуске. Если ты запускаешь `/gsd:execute-phase 2` без `/project:review` после Phase 1, агент предупреждает.

---

## Slash commands: что вклинивается между GSD

### /project:review

Self-review diff по чеклисту из `agent_docs/review-checklist.md`. Запускается после execute-phase (тестов или имплементации).

```
Проведи review: git diff main...HEAD (или последняя фаза).
Используй чеклист из agent_docs/review-checklist.md.
Формат: CRITICAL / WARNING / SUGGESTION / OK.
Если есть CRITICAL -- не продолжай, покажи мне.
```

### /project:pr

Генерация PR через `gh pr create`. Запускается после verify-work.

```
Создай PR: gh pr create.
Description: What / Why / How to test / Tests added.
Post-validate description перед отправкой.
```

### /project:validate

Post-validation в любой точке. Можно вызвать после планирования, после тестов, после PR description.

```
Перечитай свой последний output. Confidence 1-10.
Если < 8, перепиши слабые пункты.
```

---

## Что меняется при смене GSD на другой инструмент

| Компонент | При миграции |
|---|---|
| CLAUDE.md rules | Заменить имена GSD-команд на аналоги нового инструмента |
| agent_docs/ | Не меняется -- протоколы тестов и review не зависят от GSD |
| /project:review, /project:pr, /project:validate | Не меняется |
| Workflow enforcement | Обновить sequence rule в CLAUDE.md |

Вся GSD-специфика сосредоточена в CLAUDE.md rules. `agent_docs/` и slash commands -- tool-agnostic.

---

## Post-validation: защита от mirage plans ([Reflexion](https://www.promptingguide.ai/techniques/reflexion))

Агент генерирует планы, которые выглядят убедительно, но содержат пробелы. Паттерн Reflexion: агент критикует собственный output.

**Когда применять:**
- После планирования (GSD plan-checker делает часть, но не всё)
- После написания тестов (перед имплементацией) -- покрытие полное?
- После генерации PR description -- scope не преувеличен?

**Автоматизация:** `/project:validate` + правило в CLAUDE.md "после каждого планирования запусти post-validation".

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
| Hooks reference | [Claude Code docs](https://code.claude.com/docs/en/hooks) |
| LLM coding workflow | [Addy Osmani](https://medium.com/@addyosmani/my-llm-coding-workflow-going-into-2026-52fe1681325e) |

---

Связанные: [[Рабочие воркфлоу с LLM-агентами]], [[GSD (Get Shit Done)]], [[Claude Code]], [[Prompt Engineering для разработки]]
