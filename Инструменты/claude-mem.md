# claude-mem

> Persistent memory plugin для [[Claude Code]]. Сохраняет контекст между сессиями через lifecycle hooks и semantic search. ([docs](https://docs.claude-mem.ai/introduction) | [GitHub](https://github.com/thedotmack/claude-mem))

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

Plugin, который автоматически захватывает наблюдения из сессий Claude Code, сжимает их через AI и делает доступными в будущих сессиях. Решает проблему stateless-агента -- Claude Code "забывает" всё после завершения сессии, claude-mem это компенсирует.

---

## Ключевые возможности

- **Кросс-сессионная память** -- контекст сохраняется после завершения сессий
- **Auto-CLAUDE.md** -- генерация файлов с таймлайном активности в папках проекта
- **Web UI** -- визуализация потока памяти на `http://localhost:37777`
- **Hybrid search** -- semantic (Chroma vector DB) + keyword (SQLite FTS5)
- **28 языков** -- мультиязычная поддержка
- **Privacy** -- теги `<private>` для исключения секретов

---

## Как работает (5 стадий)

1. **Session Init** -- инъекция контекста из 10 предыдущих сессий
2. **Prompt Capture** -- запись пользовательского ввода
3. **Tool Observation** -- логирование read/write операций
4. **AI Processing** -- извлечение знаний через worker-процессы
5. **Session Close** -- генерация summary для будущих сессий

---

## Установка

```bash
# В Claude Code:
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
# Перезапустить Claude Code
```

**Требования:** Node.js 18+, Bun (авто-установка), uv для Python (авто-установка)

---

## MCP Search -- 3-layer workflow

Для эффективного поиска по памяти используй трёхуровневый подход (экономия токенов в 10x):

```
1. search(query)           --> получить index с IDs (~50-100 tokens/result)
2. timeline(anchor=ID)     --> контекст вокруг интересных результатов
3. get_observations([IDs]) --> полные данные ТОЛЬКО для отфильтрованных IDs
```

> Никогда не запрашивай полные данные без предварительной фильтрации.

---

## Практические советы

- Настройки: `~/.claude-mem/settings.json` (11 параметров для управления инъекцией)
- Используй `<private>` теги для паролей и ключей
- Поддерживает git worktree contexts
- Встроенные skills: `/claude-mem:mem-search`, `/claude-mem:make-plan`, `/claude-mem:do`

---

Связанные: [[Claude Code]], [[Prompt Engineering для разработки#3. Работа с контекстом]]
