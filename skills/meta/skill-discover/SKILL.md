---
name: skill-discover
description: Используй когда пользователь хочет найти, установить или оценить плагины и скилы для Claude Code. Триггеры: "найди скил для X", "есть ли плагин для Y", "установи что-нибудь для Z", "хочу уметь делать X", "какие есть инструменты для W", "добавь возможность X в Claude Code". ТАКЖЕ используй когда ты сам обнаруживаешь что для текущей задачи нет нужного инструмента — предложи найти его прежде чем начинать без правильного инструмента.
---

# Skill Discover — поиск и установка скилов/плагинов

## Обзор

Находим, оцениваем скилы и плагины для Claude Code и помогаем принять решение: брать к себе в my-claude-skills или ставить из внешнего маркетплейса.

## Архитектура личного маркетплейса

```
github.com/Ya-Rog/my-claude-skills
  skills/
    meta/      ← инфраструктурные скилы (skill-discover, skill-maintain)
    engineering/  ← паттерны разработки
    frontend/
    database/
    ...
  CATALOG.md   ← переведённые и кастомизированные скилы
  OFFICIAL.md  ← официальные скилы которые не трогаем
```

---

## Известные маркетплейсы

### Официальные от Anthropic

| Маркетплейс | Команда добавления | Что внутри |
|---|---|---|
| `claude-plugins-official` | встроен автоматически | frontend-design, playwright, agent-sdk-dev, skill-creator и др. |
| `anthropics/skills` | `/plugin marketplace add anthropics/skills` | pdf, docx, pptx, xlsx, schedule |

### Community-маркетплейсы

| Маркетплейс | Команда добавления | Масштаб |
|---|---|---|
| `jeremylongshore/claude-code-plugins-plus-skills` | `/plugin marketplace add jeremylongshore/claude-code-plugins-plus-skills` | 347 плагинов, 1900+ скилов, 22 категории |
| `alirezarezvani/claude-skills` | `/plugin marketplace add alirezarezvani/claude-skills` | 180+ скилов: engineering, marketing, product, compliance |
| `davepoon/buildwithclaude` | `/plugin marketplace add davepoon/buildwithclaude` | 489+ расширений: MCP-интеграции, frontend, AI/LLM |
| `daymade/claude-code-skills` | `/plugin marketplace add daymade/claude-code-skills` | github-ops, mermaid-tools, ppt-creator, ui-designer |
| `travisvn/awesome-claude-skills` | `/plugin marketplace add travisvn/awesome-claude-skills` | superpowers, security |

### Веб-каталоги

- **claudecodeplugins.io** — каталог к jeremylongshore
- **buildwithclaude.com/marketplaces** — список всех маркетплейсов

---

## Алгоритм работы

### Шаг 1 — Понять задачу

Уточни одним вопросом если задача неясна: что именно нужно делать?

Примеры → ключевые слова поиска:
- «делать SEO-аудит» → seo, audit, marketing
- «генерировать диаграммы» → mermaid, diagram
- «работать с PDF» → pdf, document
- «автоматизировать git» → git, workflow, commit

### Шаг 2 — Проверить уже установленное

```bash
cat ~/.claude/settings.json | grep -A 50 "enabledPlugins"
find ~/.claude/plugins -name "SKILL.md" 2>/dev/null | head -20
```

Если нужное уже есть — сообщи и покажи как использовать.

### Шаг 3 — Поиск

**Через web_search:**
```
"Claude Code" plugin {ключевое_слово} site:github.com
"Claude Code" skill {ключевое_слово} SKILL.md
claude plugin marketplace {домен_задачи} 2026
```

Затем `web_fetch` для README найденных репозиториев.

**Если установлен GitHub MCP** — читать SKILL.md репозиториев напрямую через него.

### Шаг 4 — Показать таблицу с анализом

**Обязательный формат — таблица со всеми найденными вариантами:**

**Таблица скилов:**

| Скил | Что делает | Откуда | Стоит кастомизировать? |
|---|---|---|---|
| **[название]** | [2-4 предложения на русском — объясни практически что получит пользователь, какую проблему решает, в каких ситуациях помогает. Никакого необъяснённого англ. жаргона — если термин неизбежен, объясни его в скобках] | [маркетплейс] ⚠️ Anthropic / community | **ДА/НЕТ/⛔ Пропустить** — [конкретная причина понятным языком] |

**Правила колонки "Что делает":**
- Минимум 2 предложения — одного мало чтобы принять решение
- Объясни как пользователь будет это использовать, а не просто что это такое
- Если есть команды (/brainstorm, /handoff) — укажи их, это помогает понять реальное использование
- Термины на английском — либо убрать, либо объяснить в скобках

**Правила колонки "Стоит кастомизировать?":**
- `⚠️ Anthropic` → всегда **НЕТ**, официальный
- Генерирует текст/шаблоны которые пользователь видит постоянно → **ДА если** нужен русский
- Инструмент/команда без видимых шаблонов → **НЕТ**, ставим из маркетплейса
- Хардкод под чужой стек → **ДА** — нужна адаптация
- Нет `/plugin install` команды, но SKILL.md есть на GitHub → **ДА** — берём в my-claude-skills вручную (читаем SKILL.md, переводим, добавляем)
- Установка через `git clone + ./setup` скрипт → **⛔ Пропустить** — скрипт setup может сломать структуру ~/.claude, конфликты с плагинной системой. Если хочется — найти SKILL.md внутри репо и добавить вручную без скрипта
- Совсем не найти SKILL.md нигде → **⛔ Пропустить** — нечего добавлять

**Отдельная таблица для MCP** (это не скилы — это подключения к внешним системам):

| MCP | Что делает | Установка | Брать? |
|---|---|---|---|
| **[название]** | [2-4 предложения — какие внешние системы подключает, что становится возможным] | [npm / pip / docker / API-ключ нужен?] | **ДА / НЕТ / Условно** — [когда нужен] |

MCP добавляются в `settings.json` → `mcpServers`, а не через `/plugin install`. Список установленных MCP — в `~/my-claude-skills/mcp/CATALOG.md`.

После таблиц — сразу итог: что устанавливаем и как.

### Шаг 5а — Если в my-claude-skills

```bash
# 1. Прочитать оригинальный SKILL.md
# 2. Перевести на русский (кроме команд, путей, кода)
# 3. Добавить frontmatter с метаданными источника:
#    source_repo, source_path, source_sha, source_checked, category
mkdir -p ~/my-claude-skills/skills/{категория}/{имя}/
# Записать SKILL.md

# 4. Добавить в marketplace.json
# 5. Добавить в settings.json: "{имя}@my-claude-skills": true
cd ~/my-claude-skills
git add . && git commit -m "Добавлен скил {имя} (перевод)" && git push
```

### Шаг 5б — Если из внешнего маркетплейса

```bash
# 1. Добавить маркетплейс если нужно (один раз):
/plugin marketplace add {owner/repo}

# 2. Установить скил:
/plugin install {skill-id}@{marketplace-id}

# Скил будет автообновляться — ничего не трогать
```

### Шаг 6 — Обновление скилов в my-claude-skills

Используй skill-maintain для проверки устаревших переводов.

---

## Скилы vs MCP

- **Скил** = инструкция КАК делать (SKILL.md)
- **MCP** = подключение к внешней системе (GitHub API, браузер, БД)
- Многие плагины включают оба — скил в маркетплейс, MCP в settings.json

Если скил требует MCP — всегда сообщай об этом пользователю.
