# LLM Wiki -- использование AI-агентов в работе

> Структурированная база знаний по инструментам, воркфлоу и практикам работы с LLM для разработки и проектной деятельности.

---

## Практики

> `Практики/`

| Тема | Описание | Ключевые источники |
|---|---|---|
| [[Prompt Engineering для разработки]] | Принципы промптинга, CLAUDE.md, контекст | [best practices](https://code.claude.com/docs/en/best-practices) / [Habr: контекст](https://habr.com/ru/articles/971772/) |
| [[Рабочие воркфлоу с LLM-агентами]] | Workflow для фич, дебага, review, anti-patterns | [Habr: субагенты](https://habr.com/ru/articles/946748/) / [docs](https://code.claude.com/docs/ru/sub-agents) |
| [[Сравнение LLM моделей и цены (2026)]] | Claude vs GPT vs Gemini vs DeepSeek, матрица выбора | [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) / [OpenAI pricing](https://openai.com/api/pricing/) |

---

## Инструменты

> `Инструменты/` -- то, что используем в работе

| Инструмент | Что делает | Источник |
|---|---|---|
| [[Claude Code]] | CLI-агент: sub-agents, skills, hooks, MCP | [docs](https://code.claude.com/docs/ru/sub-agents) |
| [[claude-mem]] | Persistent memory между сессиями | [docs](https://docs.claude-mem.ai/introduction) / [GitHub](https://github.com/thedotmack/claude-mem) |
| [[Context7]] | Актуальная документация библиотек в контексте | [GitHub](https://github.com/upstash/context7) |
| [[GSD (Get Shit Done)]] | Структурированный project management workflow | [GitHub](https://github.com/gsd-build/get-shit-done) |
| [[agent-browser]] | Browser automation для AI-агентов | [GitHub](https://github.com/vercel-labs/agent-browser) |
| [[MCP серверы -- каталог]] | Справочник полезных MCP серверов по категориям | [mcpservers.org](https://mcpservers.org) |

---

## Изучить

> `Изучить/` -- интересные инструменты, которые стоит попробовать

| Инструмент | Приоритет | Что делает | Источник |
|---|---|---|---|
| [[Superpowers]] | Высокий | Agentic skills framework | [GitHub](https://github.com/obra/superpowers) |
| [[Awesome Claude Code]] | Высокий | Каталог ресурсов экосистемы | [GitHub](https://github.com/hesreallyhim/awesome-claude-code) |
| [[llms.txt]] | Средний | Стандарт AI-friendly контента | [Habr](https://habr.com/ru/articles/974882/) / [llmstxt.org](https://llmstxt.org) |
| [[CodeWiki]] | Средний | Автодокументация codebase | [Habr](https://habr.com/ru/articles/1002424/) |
| [[UI-UX Pro Max Skill]] | Низкий | AI-генератор design system | [GitHub](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) |

---

## Стек по задачам

### Разработка
```
Claude Code (Opus/Sonnet) + context7 + claude-mem + GSD
```
- [[Claude Code]] как основной агент
- [[Context7]] для актуальной документации
- [[claude-mem]] для памяти между сессиями
- [[GSD (Get Shit Done)]] для структурированных проектов

### Дебаг
```
Claude Code + agent-browser (для UI) + sub-agents
```
- Быстрый и глубокий debug workflow --> [[Рабочие воркфлоу с LLM-агентами#3. Workflow: Debug]]
- [[agent-browser]] для визуального тестирования

### Code Review
```
Claude Code + gh CLI + sub-agents (reviewer)
```
- Self-review и review PR --> [[Рабочие воркфлоу с LLM-агентами#4. Workflow: Code Review]]

### Проектная работа
```
GSD + Claude Code + MCP (Notion/Slack)
```
- [[GSD (Get Shit Done)]] для полного цикла: plan -> execute -> verify
- [[MCP серверы -- каталог]] для интеграций

---

## Quick reference

### Ключевые команды Claude Code
| Команда | Назначение |
|---|---|
| `/clear` | Очистка контекста (делай чаще!) |
| `/mcp` | Статус MCP серверов |
| `/agents` | Управление sub-agents |
| `/compact` | Сжатие контекста |
| Ctrl+T | Todo list |

### Выбор модели
| Задача | Модель |
|---|---|
| Сложная архитектура | Opus 4.6 |
| Повседневный кодинг | Sonnet 4.6 |
| Быстрые задачи | Haiku 4.5 |
| Бюджетные задачи | DeepSeek V3.2 |

Подробнее --> [[Сравнение LLM моделей и цены (2026)#6. Матрица решений]]

---

*Последнее обновление: февраль 2026*
