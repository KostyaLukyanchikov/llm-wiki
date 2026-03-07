# CLAUDE.md шаблон -- Spec-Driven

> Готовый набор файлов для 10-step TDD-first workflow поверх GSD: CLAUDE.md + 6 slash commands. Каждый GSD-phase = один модуль, внутри -- полный цикл discuss-test-implement-review.

Назад: [[README]] | Папка: `Практики/`

---

## Что в комплекте

```
CLAUDE.md                                    -- правила + sequence table
.claude/commands/
  sdd-discuss-tests.md                       -- шаг 2: обсуждение тестов
  sdd-plan-tests.md                          -- шаг 3: план тестов
  sdd-review-test-plan.md                    -- шаг 4: review тест-плана
  sdd-execute-tests.md                       -- шаг 5: написание RED тестов
  sdd-review-tests.md                        -- шаг 6: review кода тестов
  sdd-review-impl.md                         -- шаг 10: review реализации
```

Шаги 1, 7, 8, 9 -- GSD native commands (`/gsd:discuss-phase`, `/gsd:plan-phase`, `/gsd:execute-phase`, `/gsd:verify-work`).

---

## Внедрение

### Новый проект

```bash
# 1. Инициализируй GSD
git init && npx get-shit-done-cc@latest
/gsd:new-project

# 2. Создай структуру
mkdir -p .claude/commands

# 3. Скопируй файлы из шаблонов ниже
#    CLAUDE.md          -- из секции "1. CLAUDE.md"
#    .claude/commands/  -- из секции "2. Slash commands"

# 4. Адаптируй плейсхолдеры {..} в CLAUDE.md под свой стек

# 5. Первый milestone
/gsd:new-milestone
/gsd:discuss-phase 1
# после discuss агент предложит /project:sdd-discuss-tests
```

### Существующий проект с GSD

```bash
# 1. Создай .claude/commands/ и скопируй 6 slash commands
mkdir -p .claude/commands

# 2. Добавь SDD-секцию в существующий CLAUDE.md
#    НЕ заменяй весь файл -- вставь секции:
#    - "SDD Workflow (STRICT)"
#    - "Testing"
#    - "Commit discipline"

# 3. Коммит
git add .claude/commands/ CLAUDE.md
git commit -m "feat: add SDD 10-step workflow"
```

Следующий `/gsd:discuss-phase` уже запустит 10-step цикл.

### Без GSD

Замени GSD-команды (шаги 1, 7, 8, 9) на свой процесс. Кастомные `/project:sdd-*` commands не зависят от GSD.

---

## 1. CLAUDE.md

````markdown
# {Project Name} -- Agent Instructions

## Project overview
{Одно предложение: что делает проект, основной стек}

## SDD Workflow (STRICT)

Each GSD phase = one module. Every phase follows a 10-step sequence.

### Sequence

| # | Step | Command | Artifact | Requires |
|---|------|---------|----------|----------|
| 1 | Discuss requirements | /gsd:discuss-phase | CONTEXT.md | -- |
| 2 | Discuss tests | /project:sdd-discuss-tests | TEST-SPEC.md | CONTEXT.md |
| 3 | Plan tests | /project:sdd-plan-tests | TEST-PLAN.md | TEST-SPEC.md |
| 4 | Review test plan | /project:sdd-review-test-plan | TEST-PLAN-REVIEW.md | TEST-PLAN.md |
| 5 | Execute tests | /project:sdd-execute-tests | code (RED) + commit | TEST-PLAN-REVIEW.md |
| 6 | Review tests | /project:sdd-review-tests | TEST-REVIEW.md | test files |
| 7 | Plan implementation | /gsd:plan-phase | XX-PLAN.md | TEST-REVIEW.md |
| 8 | Execute implementation | /gsd:execute-phase | code (GREEN) + commit | PLAN.md |
| 9 | Verify | /gsd:verify-work | VERIFICATION.md | implementation |
| 10 | Review implementation | /project:sdd-review-impl | IMPL-REVIEW.md | VERIFICATION.md |

### Enforcement rule

Before executing any command in the sequence, check which artifacts exist in `.planning/phases/{phase}/`.
If the previous step's artifact is missing:
1. WARN: "{step name} not completed -- {artifact} not found."
2. SUGGEST: "Run {correct command} first, or type 'skip' to proceed anyway."

After completing any step, suggest the next command from the sequence.

### Quick task rules
For /gsd:quick, ALWAYS write tests before implementation:
1. Discuss requirements + test scenarios with developer (2-3 questions max)
2. Write failing tests
3. Implement until GREEN
4. Run /project:sdd-review-impl
No separate artifacts for quick tasks.

## Testing
- Framework: {vitest / jest / pytest / ...}
- Run: `{npm test / pytest / ...}`
- Naming: `describe("{Module}") > it("should {action} when {condition}")`
- NEVER write tests and implementation in the same step

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

## 2. Slash commands

### .claude/commands/sdd-discuss-tests.md

````markdown
---
description: "SDD Step 2: Discuss test scenarios based on requirements"
---
Phase: $ARGUMENTS

## Instructions

1. Read `.planning/phases/{phase}/XX-CONTEXT.md` to understand requirements and decisions.

2. Propose a list of testable behaviors grouped by component:
   - Happy path scenarios
   - Edge cases
   - Error handling
   - Security concerns
   - Non-functional requirements (if applicable)

3. For each component, ask the developer:
   - "Какие сценарии критичны для {component}?"
   - "Какие edge cases для {input/state}?"
   - "Что должно происходить при {error condition}?"
   - "Есть ли бизнес-правила, которые не очевидны из кода?"

4. Rules:
   - Ask concrete questions, not vague "что ещё?"
   - Propose scenarios and let developer confirm/adjust
   - If developer says "ты решай" -- propose 3 options and explain tradeoffs
   - Maximum 2 rounds of questions per component

5. After agreement, write `TEST-SPEC.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Specification

**Agreed:** {date}

## Scenarios

### {Component A}
- [ ] {scenario 1} -- {type: happy path / edge case / error / security} -- {priority: must / should / could}
- [ ] {scenario 2} ...

### {Component B}
...

## Out of scope
- {What we explicitly decided NOT to test and why}

## Open questions
- {Any unresolved questions to revisit during implementation}
```

6. Suggest next step: "Запустите `/project:sdd-plan-tests {phase}` для создания плана тестов."
````

### .claude/commands/sdd-plan-tests.md

````markdown
---
description: "SDD Step 3: Create concrete test plan from test spec"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `TEST-SPEC.md` and `CONTEXT.md`.

2. Transform scenarios into a concrete test plan:
   - Map each scenario to a test file and describe/it block
   - Define required mocks, fixtures, and test data
   - Identify shared setup (beforeEach, factories, helpers)
   - Order tests by dependency (independent first)

3. Write `TEST-PLAN.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Plan

**Created:** {date}
**Based on:** TEST-SPEC.md

## Test structure

### {test-file-1.test.ts}
| describe | it | Scenario from spec | Priority |
|----------|----|--------------------|----------|
| {Component} | should {action} when {condition} | {ref to TEST-SPEC scenario} | must |
| ... | ... | ... | ... |

### {test-file-2.test.ts}
...

## Shared setup
- {Mocks, fixtures, factories needed}
- {Test data descriptions}

## Dependencies
- {External services to mock}
- {Database state requirements}

## Estimated test count: {N}
```

4. Suggest next step: "Запустите `/project:sdd-review-test-plan {phase}` для ревью плана."
````

### .claude/commands/sdd-review-test-plan.md

````markdown
---
description: "SDD Step 4: Review test plan with developer"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `TEST-PLAN.md` and `TEST-SPEC.md`.

2. Self-review the plan:
   - Every scenario from TEST-SPEC.md is covered?
   - Tests check behavior, not implementation details?
   - Assertions will be meaningful (not just "doesn't throw")?
   - No redundant or overlapping tests?
   - Priorities are correct?
   - Confidence score 1-10

3. Present findings to developer:
   - Coverage summary: "{N} tests across {M} files, covering {K} scenarios from spec"
   - Gaps found (if any)
   - Confidence score
   - Questions or concerns

4. Discuss with developer. Incorporate feedback.

5. Write `TEST-PLAN-REVIEW.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Plan Review

**Reviewed:** {date}
**Confidence:** {X}/10

## Coverage
- Scenarios from spec: {total}
- Covered by plan: {covered}
- Gaps: {list or "none"}

## Review notes
- {Feedback item 1 -- resolved/deferred}
- {Feedback item 2 -- resolved/deferred}

## Changes made
- {What was changed in TEST-PLAN.md after review}

## Decision: APPROVED / APPROVED WITH CHANGES / NEEDS REWORK
```

6. If approved, suggest: "Запустите `/project:sdd-execute-tests {phase}` для написания тестов."
   If needs rework, suggest: "Обновите TEST-PLAN.md и запустите `/project:sdd-review-test-plan {phase}` повторно."
````

### .claude/commands/sdd-execute-tests.md

````markdown
---
description: "SDD Step 5: Write failing (RED) tests from approved plan"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `TEST-PLAN.md` and `TEST-PLAN-REVIEW.md`.
   - If TEST-PLAN-REVIEW.md status is not APPROVED -- stop and suggest review first.

2. Write tests following the plan:
   - Follow naming conventions from CLAUDE.md
   - All tests MUST be RED (failing) -- no implementation code
   - One test file per component (as defined in plan)
   - Use descriptive test names: `should {action} when {condition}`

3. Run the test suite to confirm all tests FAIL as expected.
   - If any test passes -- it's testing something that already exists or is trivial. Flag it.

4. Commit: `test({module}): add failing tests for {component}`

5. Self-validate:
   - Every test from the plan is written?
   - Each test has meaningful assertions?
   - Tests are independent (no order dependency)?
   - Report: "{N} tests written, all RED, confidence {X}/10"

6. Suggest next step: "Запустите `/project:sdd-review-tests {phase}` для ревью кода тестов."
````

### .claude/commands/sdd-review-tests.md

````markdown
---
description: "SDD Step 6: Review test code with developer"
---
Phase: $ARGUMENTS

## Instructions

1. Show the test diff: `git diff` for the test files created in step 5.

2. Review the test code:
   - Tests match the approved plan?
   - Assertions are meaningful and specific?
   - No tests on implementation details (internal state, private methods)?
   - Tests are readable and well-structured?
   - Mocks are minimal and appropriate?
   - Test names clearly describe the scenario?

3. Present review to developer with format:
   - OK: {what looks good}
   - WARNING: {file}:{line} -- {concern}
   - SUGGESTION: {file}:{line} -- {improvement}

4. Discuss with developer. Apply agreed changes. Re-commit if needed.

5. Write `TEST-REVIEW.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Review

**Reviewed:** {date}

## Summary
- Test files: {list}
- Total tests: {N}
- All RED: yes/no

## Review results
- {Item 1 -- OK / fixed / deferred}
- {Item 2 -- OK / fixed / deferred}

## Changes after review
- {What was changed and why}

## Decision: APPROVED
```

6. Suggest next step: "Запустите `/gsd:plan-phase {phase}` для планирования имплементации."
````

### .claude/commands/sdd-review-impl.md

````markdown
---
description: "SDD Step 10: Review implementation code with developer"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `VERIFICATION.md` to understand test results.

2. Show the implementation diff: `git diff` for implementation files (exclude test files).

3. Review against checklist:

### Correctness
- [ ] Logic matches requirements from CONTEXT.md
- [ ] Edge cases from TEST-SPEC.md are handled
- [ ] No off-by-one errors
- [ ] Error handling for external calls

### Security
- [ ] No hardcoded secrets
- [ ] No SQL injection / XSS / command injection
- [ ] Input validation at system boundaries
- [ ] No sensitive data in logs

### Code quality
- [ ] No dead code introduced
- [ ] No TODO/FIXME without tracking issue
- [ ] Consistent naming conventions
- [ ] No unnecessary complexity

### Breaking changes
- [ ] API contracts preserved (or migration provided)
- [ ] No breaking changes in public interfaces
- [ ] Database migrations are reversible

4. Present findings with format:
   - CRITICAL: {file}:{line} -- {description}
   - WARNING: {file}:{line} -- {description}
   - SUGGESTION: {file}:{line} -- {description}
   - OK: {category}

5. If CRITICAL found -- stop, discuss, fix before proceeding.

6. Discuss with developer. Apply agreed changes. Re-commit if needed.

7. Write `IMPL-REVIEW.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Implementation Review

**Reviewed:** {date}

## Test results
- Total: {N}, Passing: {M}, Failing: {F}

## Review results
- {CRITICAL / WARNING / SUGGESTION / OK items}

## Changes after review
- {What was changed and why}

## Decision: APPROVED / NEEDS FIXES
```

8. If approved, report: "Phase {N} complete. Все 10 шагов пройдены."
   Suggest next phase: "Запустите `/gsd:discuss-phase {next}` для следующего модуля."
````

---

## Hook configs (.claude/settings.json)

Опциональные hooks для автоматического прогона тестов ([docs](https://code.claude.com/docs/en/hooks)):

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

Для больших test suites используй targeted run (`{npm test -- --related}`) или guard-подобные инструменты.

---

Связанные: [[Spec-Driven Development с LLM-агентами]], [[Claude Code]], [[GSD (Get Shit Done)]]