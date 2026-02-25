# MCP серверы -- каталог

> Справочник полезных MCP серверов для [[Claude Code]], сгруппированных по назначению.

Назад: [[README]] | Папка: `Инструменты/`

---

## Базы данных и backend

| Сервер | Назначение |
|---|---|
| PostgreSQL MCP | Запросы к PostgreSQL на natural language |
| SQLite MCP | Управление SQLite базами |
| Supabase MCP | Database, auth, edge functions через Supabase |

## Web и поиск

| Сервер | Назначение |
|---|---|
| [[Context7]] | Актуальная документация библиотек |
| Brave Search MCP | Web-поиск общего назначения |
| Firecrawl MCP | Конвертация URL в чистый Markdown |
| Jina Reader MCP | Конвертация URL в Markdown |
| Perplexity MCP | Семантический поиск (платный) |
| Exa MCP | Семантический поиск (платный) |

## Дизайн и frontend

| Сервер | Назначение |
|---|---|
| Figma MCP | Доступ к live design layer, генерация кода |
| Magic UI MCP | React/Tailwind production-ready компоненты |

## Автоматизация и интеграции

| Сервер | Назначение |
|---|---|
| Notion MCP | Документация, project databases |
| Slack MCP | Сообщения, поиск, нотификации |
| Zapier MCP | 500+ SaaS интеграций |
| GitHub MCP | Issues, PRs, repos |

## DevTools

| Сервер | Назначение |
|---|---|
| Desktop Commander | Terminal access, process management |
| chrome-devtools-mcp | Управление Chrome из агента |
| E2B MCP | Sandboxed cloud environments |
| Filesystem MCP (official) | Directory-restricted доступ к файлам |

---

## Управление MCP

```bash
# Добавить
claude mcp add --scope user --transport stdio server-name -- command args

# Удалить
claude mcp remove server-name

# Проверить статус
/mcp  # внутри Claude Code
```

### Scope
- `user` -- доступен во всех проектах
- `project` -- текущий проект, шарится через git
- `local` -- текущий проект, локально

---

## Важно: контекст

Каждый подключённый MCP server занимает контекст (описания инструментов). Держи активными **только нужные** для текущей задачи.

Подробнее --> [[Prompt Engineering для разработки#3.3 MCP-серверы: скрытый потребитель контекста]]

---

## Каталоги MCP серверов

- [mcpservers.org](https://mcpservers.org) -- общий каталог MCP серверов
- [mcp.so](https://mcp.so) -- ещё один каталог с поиском
- [claude-code-plugins-plus-skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills) -- 270+ plugins, 739 skills, CCPI package manager
- [claudecodeplugins.io](https://claudecodeplugins.io) -- Claude Code Skills Hub

Связанные: [[Claude Code#4. MCP (Model Context Protocol)]], [[Context7]]
