# CCM (Claude Code Usage Monitor)

> Real-time dashboard для отслеживания токенов и стоимости Claude Code сессий. ([GitHub](https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor))

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

Терминальный dashboard, который в реальном времени показывает расход токенов и стоимость по 5-часовым billing sessions Claude Code. Работает как отдельный процесс рядом с Claude Code, читает логи сессий. Включает ML-предсказания исчерпания лимита.

---

## Ключевые возможности

- Real-time мониторинг токенов и стоимости текущей billing session
- ML-предсказания исчерпания лимита (когда упрёшься в rate limit)
- Группировка по 5-часовым billing windows
- Daily view -- агрегированная статистика за день
- Поддержка планов Pro / Max / Team
- Настраиваемые темы и timezone

---

## Установка

```bash
# Через uv (рекомендуется)
uv tool install claude-monitor

# Через pip
pip install claude-monitor
```

---

## Использование

```bash
# Запуск с указанием плана
claude-monitor --plan pro

# Daily view -- сводка за день
claude-monitor --plan pro --view daily

# С настройкой timezone и темы
claude-monitor --plan pro --timezone Europe/Moscow --theme dark
```

### Типичный паттерн работы

Запускай CCM в соседнем терминальном сплите рядом с Claude Code. Dashboard обновляется автоматически и показывает:
- Текущий расход в billing session
- Прогноз исчерпания лимита
- Историю расхода за день

---

## Как это работает

CCM читает логи Claude Code сессий из локальной файловой системы. Не требует API-ключей или дополнительных конфигураций -- достаточно указать план подписки для корректного расчёта лимитов.

---

Связанные: [[Claude Code]], [[ccstatusline]]
