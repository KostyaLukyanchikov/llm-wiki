# CLAUDE.md шаблон -- SAD

> Готовый набор файлов для Supervised Agentic Development: CLAUDE.md + 6 slash commands. Каждый GSD-phase = один модуль, внутри -- 10-step цикл с cross-cutting behavioral rules из [[Superpowers]] (Iron Law, Verify RED, verification-before-completion, 3-fix escalation).

Назад: [[README]] | Папка: `Практики/`

---

## Что в комплекте

```
CLAUDE.md                                    -- правила + sequence table + cross-cutting rules
.claude/commands/
  sad-discuss-tests.md                       -- шаг 2: обсуждение тестов
  sad-plan-tests.md                          -- шаг 3: план тестов
  sad-review-test-plan.md                    -- шаг 4: review тест-плана
  sad-execute-tests.md                       -- шаг 5: написание RED тестов + Verify RED
  sad-review-tests.md                        -- шаг 6: review кода тестов
  sad-review-impl.md                         -- шаг 10: two-stage review реализации
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
# после discuss агент предложит /project:sad-discuss-tests
```

### Существующий проект с GSD

```bash
# 1. Создай .claude/commands/ и скопируй 6 slash commands
mkdir -p .claude/commands

# 2. Добавь SAD-секцию в существующий CLAUDE.md
#    НЕ заменяй весь файл -- вставь секции:
#    - "SAD Workflow (STRICT)"
#    - "Cross-cutting rules"
#    - "Testing"
#    - "Commit discipline"

# 3. Коммит
git add .claude/commands/ CLAUDE.md
git commit -m "feat: add SAD 10-step workflow"
```

### Без GSD

Замени GSD-команды (шаги 1, 7, 8, 9) на свой процесс. Кастомные `/project:sad-*` commands не зависят от GSD.

---

## 1. CLAUDE.md

````markdown
# {Project Name} -- Agent Instructions

## Project overview
{Одно предложение: что делает проект, основной стек}

## SAD Workflow (STRICT)

Each GSD phase = one module. Every phase follows a 10-step sequence.

### Sequence

| # | Step | Command | Artifact | Requires |
|---|------|---------|----------|----------|
| 1 | Design & Requirements | /gsd:discuss-phase | CONTEXT.md | -- |
| 2 | Discuss tests | /project:sad-discuss-tests | TEST-SPEC.md | CONTEXT.md |
| 3 | Plan tests | /project:sad-plan-tests | TEST-PLAN.md | TEST-SPEC.md |
| 4 | Review test plan | /project:sad-review-test-plan | TEST-PLAN-REVIEW.md | TEST-PLAN.md |
| 5 | Execute tests | /project:sad-execute-tests | code (RED) + commit | TEST-PLAN-REVIEW.md |
| 6 | Review tests | /project:sad-review-tests | TEST-REVIEW.md | test files |
| 7 | Plan implementation | /gsd:plan-phase | XX-PLAN.md | TEST-REVIEW.md |
| 8 | Execute implementation | /gsd:execute-phase | code (GREEN) + commit | PLAN.md |
| 9 | Verify | /gsd:verify-work | VERIFICATION.md | implementation |
| 10 | Review implementation | /project:sad-review-impl | IMPL-REVIEW.md | VERIFICATION.md |

### HARD-GATE on design

Step 1 CANNOT be skipped. Even "simple" tasks need design confirmation before proceeding. If developer tries to jump to step 2+, WARN and route back to /gsd:discuss-phase.

### Enforcement rule

Before executing any command in the sequence, check which artifacts exist in `.planning/phases/{phase}/`.
If the previous step's artifact is missing:
1. WARN: "{step name} not completed -- {artifact} not found."
2. SUGGEST: "Run {correct command} first, or type 'skip' to proceed anyway."

After completing any step, suggest the next command from the sequence.

### Quick task rules
For /gsd:quick, ALWAYS follow these rules:
1. Discuss requirements + test scenarios with developer (2-3 questions max)
2. Write failing tests
3. Verify RED: each test fails for the CORRECT reason (not import/syntax error)
4. Implement until GREEN
5. Verification-before-completion: run tests FRESH, read FULL output, check exit code
6. Run /project:sad-review-impl
No separate artifacts for quick tasks.

## Cross-cutting rules (apply to ALL steps)

### Iron Law
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.
Wrote code before test? DELETE it. No "reference", no "adapting" -- delete and start fresh.

### Verify RED
After writing any test, run it and verify it fails for the CORRECT reason.
- "function not found" = correct (behavior gap)
- "import error" = wrong (infrastructure problem, fix first)
- "syntax error" = wrong (fix the test)
Report format: "{N} tests, all RED, failure reasons verified, confidence {X}/10"

### Verification-before-completion
Before ANY success claim:
1. Identify verification command
2. Run it FRESH (no cached results)
3. Read FULL output
4. Check exit code
Only THEN claim success. Words "should", "probably", "seems to" = VIOLATION.

### 3-fix escalation
If 3+ fixes fail, each revealing new problem in different place = architectural problem.
STOP. Discuss with human. Do NOT attempt Fix #4.

## Testing
- Framework: {vitest / jest / pytest / ...}
- Run: `{npm test / pytest / ...}`
- Naming: `describe("{Module}") > it("should {action} when {condition}")`
- NEVER write tests and implementation in the same step
- Tests MUST verify behavior, not implementation details

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

### .claude/commands/sad-discuss-tests.md

````markdown
---
description: "SAD Step 2: Discuss test scenarios based on requirements"
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
   - Tests MUST verify behavior, not implementation details. If a proposed test checks internal state or private methods -- reframe it as behavior test.

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

6. Suggest next step: "Запустите `/project:sad-plan-tests {phase}` для создания плана тестов."
````

### .claude/commands/sad-plan-tests.md

````markdown
---
description: "SAD Step 3: Create concrete test plan with FULL test code"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `TEST-SPEC.md` and `CONTEXT.md`.

2. Transform scenarios into a concrete test plan with FULL test code:
   - Map each scenario to a test file and describe/it block
   - Write complete test code: imports, setup, assertions, teardown
   - Define required mocks, fixtures, and test data
   - Identify shared setup (beforeEach, factories, helpers)
   - Order tests by dependency (independent first)

3. The plan must be readable by "an enthusiastic junior engineer with no project context" -- every detail explicit, no implicit knowledge assumed.

4. Write `TEST-PLAN.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Plan

**Created:** {date}
**Based on:** TEST-SPEC.md

## Test structure

### {test-file-1.test.ts}

| describe | it | Scenario from spec | Priority |
|----------|----|--------------------|----------|
| {Component} | should {action} when {condition} | {ref to TEST-SPEC scenario} | must |

Full test code:

\`\`\`typescript
import { ... } from '...';

describe('{Component}', () => {
  // setup
  beforeEach(() => { ... });

  it('should {action} when {condition}', () => {
    // arrange
    // act
    // assert
  });
});
\`\`\`

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

5. Suggest next step: "Запустите `/project:sad-review-test-plan {phase}` для ревью плана."
````

### .claude/commands/sad-review-test-plan.md

````markdown
---
description: "SAD Step 4: Two-pass review of test plan"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `TEST-PLAN.md` and `TEST-SPEC.md`.

2. **Pass 1 -- Spec coverage:**
   - Every scenario from TEST-SPEC.md has a corresponding test?
   - Every edge case covered?
   - Out-of-scope items correctly excluded?
   - No phantom tests (testing scenarios NOT in spec)?

3. **Pass 2 -- Test quality:**
   - Tests verify behavior, not implementation details?
   - Assertions are meaningful (not just "doesn't throw")?
   - No excessive mocking that hides real behavior?
   - Test code is correct and will compile?
   - No redundant or overlapping tests?

4. Present findings to developer:
   - **Pass 1 results:** "{N} scenarios, {M} covered, {K} gaps"
   - **Pass 2 results:** quality issues found
   - Confidence score 1-10
   - Questions or concerns

5. Discuss with developer. Incorporate feedback.

6. Write `TEST-PLAN-REVIEW.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Plan Review

**Reviewed:** {date}
**Confidence:** {X}/10

## Pass 1: Spec coverage
- Scenarios from spec: {total}
- Covered by plan: {covered}
- Gaps: {list or "none"}

## Pass 2: Test quality
- Behavior tests: {count} / Implementation tests: {count}
- Mock quality: {assessment}
- Issues found: {list or "none"}

## Changes made
- {What was changed in TEST-PLAN.md after review}

## Decision: APPROVED / APPROVED WITH CHANGES / NEEDS REWORK
```

7. If approved, suggest: "Запустите `/project:sad-execute-tests {phase}` для написания тестов."
   If needs rework, suggest: "Обновите TEST-PLAN.md и запустите `/project:sad-review-test-plan {phase}` повторно."
````

### .claude/commands/sad-execute-tests.md

````markdown
---
description: "SAD Step 5: Write failing (RED) tests with Iron Law check and Verify RED"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `TEST-PLAN.md` and `TEST-PLAN-REVIEW.md`.
   - If TEST-PLAN-REVIEW.md status is not APPROVED -- stop and suggest review first.

2. **Iron Law check:** Before writing tests, verify NO implementation code exists for this module. If found:
   - WARN: "Implementation code found before tests. Iron Law requires tests first."
   - List the files found
   - Ask developer: "Delete implementation and proceed with tests, or discuss?"

3. Write tests following the plan:
   - Follow naming conventions from CLAUDE.md
   - All tests MUST be RED (failing) -- no implementation code
   - One test file per component (as defined in plan)
   - Use descriptive test names: `should {action} when {condition}`

4. **Verify RED:** Run the test suite and for EACH failing test verify the failure reason:
   - "function/module not found" = CORRECT (behavior gap)
   - "import error" = WRONG (fix infrastructure first, then re-verify)
   - "syntax error" = WRONG (fix the test, then re-verify)
   - "test passes" = WRONG (testing existing behavior or trivial assertion -- flag it)

5. Commit: `test({module}): add failing tests for {component}`

6. Report:
   - "{N} tests written, all RED"
   - "Failure reasons verified: {summary}"
   - "Confidence: {X}/10"

7. Suggest next step: "Запустите `/project:sad-review-tests {phase}` для ревью кода тестов."
````

### .claude/commands/sad-review-tests.md

````markdown
---
description: "SAD Step 6: Review test code with Verify RED confirmation"
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

3. **Verify RED confirmation:** Check the test execution output from step 5:
   - Each test's failure message confirms it tests the right thing?
   - No infrastructure failures (import errors, missing config)?
   - Failure messages are descriptive enough to guide implementation?

4. Present review to developer with format:
   - OK: {what looks good}
   - WARNING: {file}:{line} -- {concern}
   - SUGGESTION: {file}:{line} -- {improvement}
   - VERIFY RED: {confirmation or issues}

5. Discuss with developer. Apply agreed changes. Re-commit if needed.
   If tests were changed -- re-run and re-verify RED.

6. Write `TEST-REVIEW.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Test Review

**Reviewed:** {date}

## Summary
- Test files: {list}
- Total tests: {N}
- All RED: yes/no
- Failure reasons verified: yes/no

## Review results
- {Item 1 -- OK / fixed / deferred}
- {Item 2 -- OK / fixed / deferred}

## Verify RED results
- {Each test failure reason confirmed correct}

## Changes after review
- {What was changed and why}

## Decision: APPROVED
```

7. Suggest next step: "Запустите `/gsd:plan-phase {phase}` для планирования имплементации."
````

### .claude/commands/sad-review-impl.md

````markdown
---
description: "SAD Step 10: Two-stage implementation review"
---
Phase: $ARGUMENTS

## Instructions

1. Read the phase's `VERIFICATION.md` to understand test results.

2. Show the implementation diff: `git diff` for implementation files (exclude test files).

3. **Stage 1 -- Spec compliance:**
   - Logic matches requirements from CONTEXT.md?
   - All scenarios from TEST-SPEC.md are handled?
   - Edge cases from spec are covered in code (not just tests)?
   - No stub or placeholder implementations?
   - All tests GREEN?

4. **Stage 2 -- Code quality:**

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

5. Present findings with severity levels:
   - **CRITICAL:** {file}:{line} -- {description} (blocks approval)
   - **WARNING:** {file}:{line} -- {description} (discuss before proceeding)
   - **SUGGESTION:** {file}:{line} -- {description} (optional improvement)
   - **OK:** {category}

6. If CRITICAL found -- stop, discuss, fix before proceeding.

7. Discuss with developer. Apply agreed changes. Re-commit if needed.

8. Write `IMPL-REVIEW.md` to the phase directory:

```markdown
# Phase {N}: {Module} -- Implementation Review

**Reviewed:** {date}

## Test results
- Total: {N}, Passing: {M}, Failing: {F}

## Stage 1: Spec compliance
- Requirements covered: {X}/{Y}
- Stubs detected: {list or "none"}
- Verdict: PASS / FAIL

## Stage 2: Code quality
- CRITICAL: {count}
- WARNING: {count}
- SUGGESTION: {count}

## Review items
- {CRITICAL / WARNING / SUGGESTION / OK items with details}

## Changes after review
- {What was changed and why}

## Decision: APPROVED / NEEDS FIXES
```

9. If approved, report: "Phase {N} complete. Все 10 шагов пройдены."
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

---

Связанные: [[Supervised Agentic Development]], [[Superpowers]], [[Spec-Driven Development с LLM-агентами]], [[CLAUDE.md шаблон -- Spec-Driven]], [[Claude Code]], [[GSD (Get Shit Done)]]