# ccstatusline

> Красивый status bar для Claude Code -- модель, git branch, токены, стоимость, контекст. ([GitHub](https://github.com/sirmalloc/ccstatusline))

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

Status bar, который отображает полезную информацию прямо в интерфейсе Claude Code: текущую модель, git branch, расход токенов, стоимость и заполненность контекстного окна. Работает через нативный statusLine hook Claude Code.

---

## Нативный statusLine ([docs](https://code.claude.com/docs/en/statusline))

Claude Code поддерживает встроенный механизм status line -- внешний процесс, который возвращает JSON для отображения в нижней панели.

### JSON schema

```json
{
  "items": [
    { "label": "Model", "value": "opus-4-6" },
    { "label": "Tokens", "value": "12.3k" }
  ]
}
```

### Команды
- `/statusline` -- просмотр и управление status line из Claude Code

---

## Установка

Zero install через bunx:

```bash
bunx ccstatusline@latest
```

При первом запуске открывается TUI-конфигуратор для выбора отображаемых элементов.

### Настройка в settings.json

```json
{
  "statusLine": "bunx ccstatusline@latest"
}
```

Settings.json расположен в `~/.claude/settings.json` (глобально) или `.claude/settings.json` (проект).

---

## Отображаемая информация

| Элемент | Описание |
|---|---|
| Модель | Текущая Claude модель (opus, sonnet, haiku) |
| Git branch | Активная ветка репозитория |
| Токены | Расход токенов в текущей сессии |
| Стоимость | Стоимость текущей сессии в $ |
| Контекст | Заполненность context window в % |

---

## Кастомизация

TUI-конфигуратор позволяет выбрать, какие элементы отображать и в каком порядке. Запусти `bunx ccstatusline@latest` повторно для перенастройки.

---

Связанные: [[Claude Code]], [[CCM (Claude Code Usage Monitor)]]
