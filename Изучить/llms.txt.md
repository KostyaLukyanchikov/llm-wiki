# llms.txt

> Стандарт для предоставления контента сайтов в machine-readable формате для LLM. Предложен Jeremy Howard (fast.ai), де-факто принят индустрией. ([Habr](https://habr.com/ru/articles/974882/) | [llmstxt.org](https://llmstxt.org))

Назад: [[LLM Wiki]] | Папка: `Изучить/` | Статус: **Изучить**

---

## Что это

Три файла в корне сайта, которые дают LLM чистый контекст вместо парсинга HTML:

1. **Markdown-зеркала страниц** -- чистый Markdown вместо HTML с навигацией/рекламой
2. **llms.txt** -- index файл, AI-оптимизированная карта сайта с приоритизацией
3. **llms-full.txt** -- вся документация в одном файле

---

## Формат llms.txt

```markdown
# Название проекта

> Краткое описание (blockquote summary)

Развёрнутое описание проекта.

## Docs
- [Getting Started](https://example.com/docs/start.md): Основы работы
- [API Reference](https://example.com/docs/api.md): Полный API

## Optional
- [Changelog](https://example.com/changelog.md)
```

---

## Зачем

- Устраняет "token waste" на нерелевантные элементы страниц (навигация, футеры, реклама)
- Предотвращает галлюцинации по устаревшим API
- Даёт LLM curated контекст вместо raw HTML

---

## Кто уже использует

AWS, Docker, Stripe, Vue.js, Anthropic/Claude, Vercel, Supabase и многие другие.

---

## Связь с [[Context7]]

Context7 MCP делает похожую работу, но на стороне запроса -- подтягивает документацию из источников. llms.txt -- это на стороне публикации, чтобы сайты были AI-friendly.

---

Связанные: [[Context7]], [[Claude Code]], [[CodeWiki]]
