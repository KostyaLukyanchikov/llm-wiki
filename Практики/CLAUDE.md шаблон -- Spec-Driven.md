# CLAUDE.md шаблон -- Spec-Driven

> Готовый набор файлов для TDD-first workflow поверх GSD: CLAUDE.md + agent_docs/ + slash commands.

Назад: [[README]] | Папка: `Практики/`

---

## Что в комплекте

```
CLAUDE.md                          -- правила + ссылки (компактный)
agent_docs/
  tdd-workflow.md                  -- протокол обсуждения и написания тестов
  review-checklist.md              -- чеклист для code review
.claude/commands/
  review.md                        -- /project:review
  pr.md                            -- /project:pr
  validate.md                      -- /project:validate
```

## Внедрение: новый проект

```bash
# 1. Инициализируй проект и GSD
git init && npx get-shit-done-cc@latest
/gsd:new-project

# 2. Создай структуру
mkdir -p agent_docs .claude/commands

# 3. Скопируй файлы из шаблонов ниже
#    CLAUDE.md          -- из секции "1. CLAUDE.md"
#    agent_docs/        -- из секций "2. agent_docs/tdd-workflow.md" и "3. agent_docs/review-checklist.md"
#    .claude/commands/  -- из секции "4. Slash commands"

# 4. Адаптируй плейсхолдеры {..} в CLAUDE.md под свой стек

# 5. Первый milestone
/gsd:discuss-phase 1
# planner автоматически создаст Phase 1 = тесты (правило из CLAUDE.md)
```

**Порядок важен**: сначала GSD (`/gsd:new-project`), потом CLAUDE.md с TDD-правилами. GSD создаёт PROJECT.md и ROADMAP.md, а planner при следующем запуске прочитает CLAUDE.md и начнёт создавать Phase 1 = тесты.

## Внедрение: существующий проект

### Если GSD уже настроен

```bash
# 1. Создай структуру
mkdir -p agent_docs .claude/commands

# 2. Скопируй agent_docs/ и .claude/commands/ из шаблонов ниже

# 3. Добавь TDD-секцию в существующий CLAUDE.md
#    НЕ заменяй весь файл -- вставь секции:
#    - "TDD-first workflow (STRICT)"
#    - "Workflow sequence (STRICT)"
#    - "Post-validation"
#    Остальное (project overview, code style, structure) оставь как есть

# 4. Коммит
git add agent_docs/ .claude/commands/ CLAUDE.md
git commit -m "feat: add TDD-first workflow rules"
```

Следующий `/gsd:plan-phase` уже будет создавать Phase 1 = тесты. Текущий milestone можно доработать без TDD, а новый начать с полным workflow.

### Если GSD ещё не настроен

```bash
# 1. Установи GSD
npx get-shit-done-cc@latest

# 2. Инициализируй на существующей кодовой базе
/gsd:map-codebase        # анализ текущего кода
/gsd:new-project         # создание PROJECT.md, ROADMAP.md

# 3. Добавь TDD-файлы (как выше)
mkdir -p agent_docs .claude/commands
# скопируй шаблоны, адаптируй CLAUDE.md

# 4. Для существующего кода без тестов -- первый milestone = покрытие тестами
# Phase 1: тесты на критичные модули (агент обсудит с тобой что покрывать)
# Phase 2+: новая фича с TDD-first
```

### Если GSD не используется вообще

Шаблон работает и без GSD. Замени GSD-команды в CLAUDE.md workflow sequence на свой процесс:

```markdown
## Workflow sequence (STRICT)
Milestone: discuss → plan → write tests (RED) → /project:review → implement (GREEN) → /project:review → verify → /project:pr
Quick: discuss → tests → implement → /project:review → done
```

`agent_docs/` и `/project:*` команды не зависят от GSD.

---

## 1. CLAUDE.md

````markdown
# {Project Name} -- Agent Instructions

## Project overview
{Одно предложение: что делает проект, основной стек}

## TDD-first workflow (STRICT)

### Milestone planning rules
When creating a milestone roadmap, ALWAYS structure phases as:
- Phase 1: Tests -- write failing tests covering all requirements
- Phase 2..N-1: Implementation -- make tests GREEN, module by module
- Phase N: Review + cleanup

Phase 1 tests MUST be discussed with the developer before writing.
Read agent_docs/tdd-workflow.md for the test discussion protocol.

### Quick task rules
For /gsd:quick, ALWAYS write tests before implementation:
1. Discuss test scenarios with developer (abbreviated -- 2-3 questions max)
2. Write failing tests
3. Implement until GREEN
4. Show diff for review
Read agent_docs/tdd-workflow.md § Quick tasks for details.

### Workflow sequence (STRICT)
Milestone: discuss → plan → execute(tests) → /project:review → execute(impl) → /project:review → verify → /project:pr
Quick: discuss → tests → implement → /project:review → done

If a step is skipped, WARN the developer and require explicit confirmation.
Example: "Review step was skipped. Type 'skip review' to confirm, or run /project:review."

### Post-validation
After generating a plan, tests, or PR description, ALWAYS self-check:
- Reread your output
- Confidence 1-10
- If < 8, rewrite weak points
Or run /project:validate.

## Testing
- Framework: {vitest / jest / pytest / ...}
- Run: `{npm test / pytest / ...}`
- Naming: `describe("{Module}") > it("should {action} when {condition}")`
- NEVER write tests and implementation simultaneously

## Commit discipline
- Atomic commits: one logical unit = one commit
- Format: `{feat|fix|test|refactor}({scope}): description`
- Commit after each passing batch of tests
- `/clear` after each commit

## Code style
{Линтеры, форматтеры, конвенции}

## Project structure
```
{Структура проекта}
```
````

---

## 2. agent_docs/tdd-workflow.md

````markdown
# TDD Workflow Protocol

## Milestone: Phase 1 (Tests)

This document is read by the agent during Phase 1 execution.
Follow these steps IN ORDER. Do not skip.

### Step 1: Analyse requirements
- Read REQUIREMENTS.md and relevant source code
- List all testable behaviors as bullet points
- Group by module/component

### Step 2: Discuss with developer
For each module, ask:
- "Какие сценарии критичны для {module}?"
- "Какие edge cases для {input/state}?"
- "Что должно происходить при {error condition}?"
- "Есть ли бизнес-правила, которые не очевидны из кода?"

Rules:
- Ask concrete questions, not vague "что ещё?"
- Propose scenarios and let developer confirm/adjust
- If developer says "ты решай" -- propose 3 options and explain tradeoffs
- Maximum 2 rounds of questions per module

### Step 3: Agree on test plan
Present the final test plan as a table:

| Module | Scenario | Type | Priority |
|--------|----------|------|----------|
| Auth   | Valid login | Happy path | Must |
| Auth   | Expired token | Edge case | Must |
| Auth   | SQL injection in email | Security | Must |
| Auth   | Empty password | Validation | Should |

Wait for developer confirmation before writing any code.

### Step 4: Write tests
- Follow naming conventions from CLAUDE.md
- All tests MUST be RED (failing) -- no implementation code
- One test file per module
- Use descriptive test names: `should {action} when {condition}`
- Commit: `test(phase-N): add failing tests for {module}`

### Step 5: Validate coverage
Self-check:
1. Every requirement from the test plan is covered?
2. Each test has meaningful assertions (not just "doesn't throw")?
3. Tests check behavior, not implementation details?
4. Confidence score 1-10. If < 8, revisit gaps.

Report to developer: "{N} tests written, {M} modules covered, confidence {X}/10"

## Quick tasks

Abbreviated protocol -- same principles, compressed:

1. **Propose**: "Я предлагаю покрыть: {scenario 1}, {scenario 2}, {edge case}. Что добавить?"
2. **Confirm**: Wait for developer OK or adjustments
3. **Write**: Tests first, all RED
4. **Implement**: Make GREEN
5. **Show diff**: Wait for review

Maximum 2-3 questions. If task is trivial (< 1 test file), propose and write without discussion.

## When to skip TDD

Tests can be skipped (with developer's explicit "skip tests" confirmation) for:
- Pure UI layout/styling changes
- Configuration-only changes
- Documentation updates
- One-line fixes with existing test coverage
````

---

## 3. agent_docs/review-checklist.md

````markdown
# Code Review Checklist

Used by /project:review. Agent reads this and evaluates git diff.

## Format
For each item: CRITICAL / WARNING / SUGGESTION / OK
If any CRITICAL found -- stop and show to developer immediately.

## Checklist

### Correctness
- [ ] Logic matches requirements
- [ ] Edge cases handled
- [ ] No off-by-one errors
- [ ] Error handling for external calls

### Security
- [ ] No hardcoded secrets
- [ ] No SQL injection / XSS / command injection
- [ ] Input validation at system boundaries
- [ ] No sensitive data in logs

### Tests
- [ ] New code has corresponding tests
- [ ] Tests cover happy path + edge cases
- [ ] Tests are not testing implementation details
- [ ] All tests pass

### Code quality
- [ ] No dead code introduced
- [ ] No TODO/FIXME without tracking issue
- [ ] Consistent naming conventions
- [ ] No unnecessary complexity

### Breaking changes
- [ ] API contracts preserved (or migration provided)
- [ ] No breaking changes in public interfaces
- [ ] Database migrations are reversible
````

---

## 4. Slash commands

### .claude/commands/review.md

````markdown
---
description: "Self-review code changes against review checklist"
---
Read agent_docs/review-checklist.md.

Run: git diff main...HEAD (or git diff for unstaged changes if no branch).

For each checklist item, evaluate the diff and report:
- CRITICAL: {file}:{line} -- {description}
- WARNING: {file}:{line} -- {description}
- SUGGESTION: {file}:{line} -- {description}
- OK: {category}

If any CRITICAL issues found, list them first and ask developer how to proceed.
If all OK, report: "Review passed. Ready for next step."
````

### .claude/commands/pr.md

````markdown
---
description: "Create PR with generated description"
---
1. Run: git log main...HEAD --oneline (get commit history)
2. Run: git diff main...HEAD --stat (get changed files summary)

3. Generate PR description:
## What
{1-2 sentences: what changed}

## Why
{1-2 sentences: what problem this solves}

## How to test
{Steps for reviewer to verify}

## Tests
{List of test files added/modified, what they cover}

4. Show the description to developer for approval.

5. After approval, run:
gh pr create --title "{type}: {short description}" --body "{approved description}"

6. Report PR URL.
````

### .claude/commands/validate.md

````markdown
---
description: "Post-validation: critically evaluate your last output"
---
Reread your last output (plan / tests / PR description / any generated content).

Evaluate:
1. All requirements from the prompt are covered?
2. Edge cases accounted for?
3. Hidden dependencies identified?
4. Result is verifiable (by tests, command, or visually)?
5. Any assumptions that should be checked with developer?

Confidence: X/10

If < 8: list weak points and rewrite them.
If >= 8: report "Validation passed, confidence {X}/10" and continue.
````

---

## Hook configs (.claude/settings.json)

Опциональные hooks для автоматизации ([docs](https://code.claude.com/docs/en/hooks)):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "{npm test / pytest / ...}",
        "description": "Auto-run tests after file changes"
      }
    ]
  }
}
```

Для больших test suites используй targeted run (`{npm test -- --related}`) или guard-подобные инструменты ([tdd-guard](https://github.com/nizos/tdd-guard)).

---

Связанные: [[Spec-Driven Development с LLM-агентами]], [[Claude Code]], [[GSD (Get Shit Done)]]
