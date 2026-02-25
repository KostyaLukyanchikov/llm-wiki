# Superpowers

> Agentic skills framework для структурированного coding workflow. Заставляет AI-агента проходить через фазы design -> planning -> execution -> testing -> review.

Назад: [[README]] | Папка: `Изучить/` | Статус: **Изучить**

---

## Что это

Самый зрелый agentic framework ([GitHub](https://github.com/obra/superpowers), 61.3k stars, v4.1.1). Превращает хаотичное AI-кодирование в дисциплинированный процесс. Альтернатива/дополнение к [[GSD (Get Shit Done)]].

---

## Фазы работы

1. **Design Phase** -- агент задаёт уточняющие вопросы, презентует design spec для approval
2. **Planning Phase** -- детальный план с задачами по 2-5 минут, точные пути файлов, шаги верификации
3. **Execution Phase** -- субагенты работают по задачам + двухэтапное code review
4. **Testing** -- TDD по RED-GREEN-REFACTOR циклам
5. **Git** -- git worktrees для параллельных веток разработки

---

## Особенности

- Composable skills по категориям: testing, debugging, collaboration, code review
- Mandatory workflows автоматически активируют нужные skills
- Работает в Claude Code, Cursor, Codex, OpenCode
- Production-ready (v4.1.1, 286 commits)

---

Связанные: [[GSD (Get Shit Done)]], [[Claude Code]], [[Awesome Claude Code]]
