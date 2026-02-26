# Claude Code

> CLI-интерфейс от Anthropic для работы с Claude, расширяемый через sub-agents, skills, hooks и MCP.

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

Claude Code -- AI-агент в терминале, который может читать/редактировать файлы, запускать команды, работать с git, и всё это в контексте твоего проекта. Расширяется через 4 системы.

---

## 1. Sub-agents ([docs](https://code.claude.com/docs/ru/sub-agents) | [Habr](https://habr.com/ru/articles/946748/))

Специализированные AI-помощники, работающие в собственном context window.

### Встроенные agents
- **Explore** -- быстрый поиск по коду (Haiku)
- **Plan** -- read-only исследование
- **general-purpose** -- все инструменты

### Создание кастомных agents

Файл `.claude/agents/code-reviewer.md`:
```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
memory: user
---
You are a code reviewer...
```

### Параметры
- **model**: `sonnet`, `opus`, `haiku`, `inherit`
- **Permission modes**: `default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan`
- **memory**: `user` (глобальная), `project`, `local`
- **Background execution** -- параллельная работа в фоне
- **Isolation** -- git worktree для отдельной копии репо

### Scope файлов

| Расположение | Доступность |
|---|---|
| `--agents` CLI flag | Текущая сессия |
| `.claude/agents/` | Текущий проект |
| `~/.claude/agents/` | Все проекты |

---

## 2. Skills ([docs](https://code.claude.com/docs/ru/skills))

Переиспользуемые инструкции в формате Markdown, загружаемые по требованию.

### Возможности
- Автоматический вызов по описанию или ручной через `/skill-name`
- Поддержка аргументов: `$ARGUMENTS`, `$0`, `$1`
- Динамический контекст через `` !`command` ``
- Изоляция через `context: fork`

### Создание
```bash
mkdir -p ~/.claude/skills/my-skill
# Создать ~/.claude/skills/my-skill/SKILL.md
```

### Пример
```markdown
# Skill: prisma-migration
---
description: "Create and apply Prisma migrations"
---
1. Обнови schema.prisma
2. pnpm prisma migrate dev --name описание
3. Проверь сгенерированный SQL
4. Протестируй rollback
```

---

## 3. Hooks ([docs](https://code.claude.com/docs/ru/hooks))

Автоматизация на основе событий жизненного цикла.

### События
| Событие | Когда срабатывает |
|---|---|
| `PreToolUse` / `PostToolUse` | До/после использования инструмента |
| `UserPromptSubmit` | При отправке промпта |
| `Stop` / `SubagentStop` | При завершении работы агента |
| `SessionStart` / `SessionEnd` | Начало/конец сессии |
| `PreCompact` | Перед сжатием контекста |

### Типы hooks
- `type: "command"` -- bash-скрипт
- `type: "prompt"` -- LLM-оценка (Haiku)

### Пример
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "./scripts/run-linter.sh"
      }]
    }]
  }
}
```

> `PreToolUse` с exit code 2 блокирует вызов инструмента.

---

## 4. MCP (Model Context Protocol) ([docs](https://code.claude.com/docs/ru/mcp))

Открытый стандарт для подключения к внешним инструментам и данным. Подробнее о конкретных серверах --> [[MCP серверы -- каталог]]

### Добавление серверов
```bash
# Remote HTTP
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Local stdio
claude mcp add --transport stdio myserver -- npx -y @some/package

# User scope (все проекты)
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

### Scope
- `local` -- текущий проект, не в git
- `project` -- текущий проект, шарится через git
- `user` -- все проекты

---

## Практические советы

- `/mcp` -- проверка статуса серверов
- `/agents` -- управление и создание sub-agents
- При большом количестве MCP tools: `ENABLE_TOOL_SEARCH=auto`
- Для agents с побочными эффектами ограничивай tools через allowlist
- Claude Code сам может быть MCP server: `claude mcp serve`

---

## Ключевые команды

| Команда | Назначение |
|---|---|
| `/clear` | Очистка контекста |
| `/compact` | Сжатие без потери |
| `/mcp` | Статус MCP |
| `/agents` | Sub-agents |
| Ctrl+T | Todo list |
| `/cost` | Потраченные токены |

---

Связанные: [[claude-mem]], [[Context7]], [[GSD (Get Shit Done)]], [[agent-browser]], [[CCM (Claude Code Usage Monitor)]], [[ccstatusline]], [[Spokenly]], [[Ghostty]]
