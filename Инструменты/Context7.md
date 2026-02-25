# Context7

> MCP-сервер, доставляющий актуальную документацию библиотек прямо в контекст [[Claude Code]]. Решает проблему устаревших API в ответах LLM. ([GitHub](https://github.com/upstash/context7))

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

Вместо того чтобы LLM галлюцинировал несуществующие API по устаревшим training data, Context7 подтягивает реальную, версионно-специфичную документацию прямо из источников.

---

## Установка

```bash
# С API ключом (повышенные rate limits)
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp --api-key YOUR_API_KEY

# Без ключа (бесплатный tier)
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

> `--scope user` делает сервер доступным во всех проектах.

---

## Использование

Добавь `use context7` к промпту:
```
Create a Next.js middleware that checks for a valid JWT. use context7
```

Для конкретной библиотеки:
```
use library /supabase/supabase for API and docs
```

### Два инструмента
- `resolve-library-id` -- преобразование имени библиотеки в ID
- `query-docs` -- получение документации

---

## Практические советы

- Добавь в `CLAUDE.md` правило для авто-вызова Context7 при вопросах о библиотеках -- не придётся писать `use context7` каждый раз
- Бесплатный API ключ: https://context7.com/dashboard
- Версия определяется автоматически из промпта

---

Связанные: [[Claude Code#4. MCP (Model Context Protocol)]], [[MCP серверы -- каталог]], [[llms.txt]]
