# agent-browser

> Headless browser automation CLI для AI-агентов. Позволяет [[Claude Code]] взаимодействовать с веб-интерфейсами. ([GitHub](https://github.com/vercel-labs/agent-browser))

Назад: [[README]] | Папка: `Инструменты/`

---

## Что это

CLI для автоматизации браузера с Rust-реализацией (быстрая) и Node.js fallback. Semantic locators вместо хрупких CSS-селекторов.

---

## Возможности

- **Навигация**: open, goto, navigate
- **Взаимодействие**: click, fill, type, hover, scroll, drag, upload
- **Инспекция**: text, html, attributes, bounding boxes
- **Screenshots и PDF** генерация
- **Accessibility tree snapshots** с element references
- **Semantic locators** -- поиск по ARIA role, text, labels, placeholders, data-testid
- **Network interception** -- перехват и мокирование запросов
- **Cookie и storage management**
- **Multiple tabs/windows**
- **Visual diffing**

---

## Установка

```bash
# npm (рекомендуется)
npm install -g agent-browser
agent-browser install  # Скачать Chromium

# Homebrew (macOS)
brew install agent-browser
agent-browser install

# Quick start
npx agent-browser install
npx agent-browser open example.com
```

---

## Использование

```bash
agent-browser open example.com
agent-browser click "Login button"
agent-browser fill "Email input" "test@example.com"
agent-browser screenshot page.png
```

Интегрируется с Claude Code как skill через `.claude-plugin` конфигурацию.

---

## Практические советы

- Semantic locators устойчивее CSS-селекторов -- предпочитай их
- Используй accessibility tree snapshots для понимания структуры перед взаимодействием
- Network interception полезен для мокирования API в тестах
- Headless mode по умолчанию -- подходит для CI/CD

---

Связанные: [[Claude Code]], [[MCP серверы -- каталог]]
