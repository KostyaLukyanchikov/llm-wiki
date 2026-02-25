# CodeWiki

> Open-source framework для автогенерации иерархической документации по codebase через tree-sitter + LLM clustering.

Назад: [[LLM Wiki]] | Папка: `Изучить/` | Статус: **Изучить**

---

## Что это

Решает проблему навигации coding agents по большим репозиториям ([Habr](https://habr.com/ru/articles/1002424/)). Вместо загрузки всего кода в context window -- "progressive information disclosure": агент читает нужный уровень детализации.

---

## Как работает

1. **AST parsing** через tree-sitter (Python, Java, JS, TS, C, C++, C#)
2. **LLM-based clustering** -- группирует файлы в логические модули
3. **Bottom-up документирование** -- сначала leaf-модули по исходникам, потом parent-модули по документации детей
4. **Static site generation** -- навигируемый HTML wiki с cross-references

---

## Трёхуровневая структура

| Уровень | Что описывает |
|---|---|
| **Components** (leaf) | Отдельные функции/классы |
| **Modules** (container) | Группы компонентов |
| **Overview** (system) | Архитектурный обзор |

---

## Зачем

- Агент читает overview --> понимает архитектуру --> спускается к нужному модулю
- Экономия контекста vs загрузка всех файлов
- Документация автоматически отражает взаимодействие компонентов

---

Связанные: [[Claude Code]], [[llms.txt]], [[GSD (Get Shit Done)]]
