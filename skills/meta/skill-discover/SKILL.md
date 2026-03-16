---
name: skill-discover
description: Используй когда пользователь хочет найти, установить или оценить плагины и скилы для Claude Code. Триггеры: "найди скил для X", "есть ли плагин для Y", "установи что-нибудь для Z", "хочу уметь делать X", "какие есть инструменты для W", "добавь возможность X в Claude Code".
---

# Skill Discover — поиск и установка скилов/плагинов

## Обзор

Находим, оцениваем и устанавливаем скилы и плагины для Claude Code из всех известных маркетплейсов. После установки переводим весь скил на русский и добавляем в личный маркетплейс `Ya-Rog/my-claude-skills`.

## Архитектура личного маркетплейса

Все скилы хранятся в одном месте: `github.com/Ya-Rog/my-claude-skills`.

```
my-claude-skills/
  skills/
    skill-discover/SKILL.md   ← этот файл
    mermaid-tools/SKILL.md    ← следующие скилы сюда
    ...
```

**Правило:** никогда не устанавливать скилы из чужих маркетплейсов напрямую — только найти, перевести, добавить к себе. Это предотвращает дубли и перезапись переводов при обновлениях.

## Известные маркетплейсы

### Официальные от Anthropic

| Маркетплейс | Команда добавления | Что внутри |
|---|---|---|
| `claude-plugins-official` | встроен автоматически | LSP-плагины, code intelligence, проверенные партнёры |
| `anthropics/claude-code` | `/plugin marketplace add anthropics/claude-code` | Демо: agent-sdk-app, pr-review-toolkit, code-review |
| `anthropics/skills` | `/plugin marketplace add anthropics/skills` | Официальные скилы: docx, pdf, pptx, xlsx, canvas-design и др. |

### Community-маркетплейсы

| Маркетплейс | Команда добавления | Масштаб |
|---|---|---|
| `jeremylongshore/claude-code-plugins-plus-skills` | `/plugin marketplace add jeremylongshore/claude-code-plugins-plus-skills` | 347 плагинов, 1900+ скилов, 22 категории |
| `alirezarezvani/claude-skills` | `/plugin marketplace add alirezarezvani/claude-skills` | 180+ скилов: engineering, marketing, product, compliance |
| `davepoon/buildwithclaude` | `/plugin marketplace add davepoon/buildwithclaude` | 489+ расширений: MCP-интеграции, frontend, AI/LLM |
| `kivilaid/plugin-marketplace` | `/plugin marketplace add kivilaid/plugin-marketplace` | 87 плагинов из 10+ источников, 100+ агентов |
| `daymade/claude-code-skills` | `/plugin marketplace add daymade/claude-code-skills` | github-ops, mermaid-tools, ppt-creator, ui-designer |
| `travisvn/awesome-claude-skills` | `/plugin marketplace add travisvn/awesome-claude-skills` | Curated-список: superpowers (/brainstorm, /write-plan), security |
| `jimmc414/claude-code-plugin-marketplace` | `/plugin marketplace add jimmc414/claude-code-plugin-marketplace` | Python-паттерны, Socratic workflows, Ollama |

### Веб-каталоги для ручного поиска

- **claudecodeplugins.io** — каталог к jeremylongshore
- **buildwithclaude.com/marketplaces** — список всех маркетплейсов
- **claudecodecommands.directory** — команды и агенты

---

## Алгоритм работы

### Шаг 1 — Понять задачу

Уточни одним вопросом если задача неясна: что именно нужно делать?

Примеры → ключевые слова поиска:
- «делать SEO-аудит» → seo, audit, marketing
- «генерировать диаграммы» → mermaid, diagram, visual-documentation
- «работать с PDF» → pdf, document
- «автоматизировать git» → git, workflow, commit

### Шаг 2 — Проверить уже установленное

```bash
# Установленные маркетплейсы
cat ~/.claude/settings.json | grep -A 50 "marketplaces"

# Установленные скилы
find ~/.claude/skills -name "SKILL.md" 2>/dev/null | head -30
```

Если нужное уже есть — сообщи и покажи как активировать.

### Шаг 3 — Поиск

**Через web_search:**
```
"Claude Code" plugin {ключевое_слово} site:github.com
"Claude Code" skill {ключевое_слово} SKILL.md
claude plugin marketplace {домен_задачи} 2026
```

Затем `web_fetch` для README найденных репозиториев.

**Если установлен GitHub MCP** — читать содержимое SKILL.md репозиториев напрямую через него вместо web_fetch.

### Шаг 4 — Сформировать топ-3

Для каждого варианта:

```
## [Название]
Откуда: github.com/owner/repo

Зачем нужен: [1-2 предложения — что конкретно умеет]
Подходит потому что: [почему именно это под задачу]
Минус: [честно — чего не умеет]
Требует: [MCP-серверы, API-ключи — если есть]
```

### Шаг 5 — Добавить в личный маркетплейс

После того как пользователь выбрал:

```bash
# 1. Прочитать оригинальный SKILL.md через web_fetch или GitHub MCP
# 2. Перевести весь текст на русский (кроме команд, путей, технических терминов)
# 3. Добавить файл в личный маркетплейс
mkdir -p ~/my-claude-skills/skills/{имя-скила}
# Записать переведённый SKILL.md туда

# 4. Закоммитить и запушить
cd ~/my-claude-skills
git add skills/{имя-скила}/
git commit -m "Добавлен скил {имя-скила} (перевод)"
git push

# 5. Скил автоматически появится в Claude Code при следующем запуске
```

**Не устанавливать через `/plugin install` из чужих маркетплейсов** — это создаёт дубли и теряет переводы при обновлениях.

### Шаг 6 — Обновление скила

Когда автор оригинала выпускает новую версию:

```bash
# Посмотреть что изменилось в оригинале
web_fetch https://raw.githubusercontent.com/owner/repo/main/skills/skill-name/SKILL.md

# Перенести нужные изменения в свою переведённую версию
# Закоммитить
```

Обновления — редкие и осознанные, не автоматические.

---

## Важные нюансы

**Скилы vs MCP — разные вещи:**
- Скил = инструкция КАК делать (SKILL.md с текстом)
- MCP = подключение к внешней системе (GitHub API, база данных, браузер)
- Многие плагины включают оба компонента — скил устанавливается в маркетплейс, MCP — в settings.json

**Когда нужен MCP дополнительно:**
- Скил для работы с GitHub → нужен GitHub MCP
- Скил для браузерного тестирования → нужен Playwright/Chrome MCP
- Всегда сообщай пользователю если скил требует MCP
