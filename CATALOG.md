# Каталог скилов — Ya-Rog/my-claude-skills

Личный маркетплейс скилов для Claude Code. Все скилы переведены на русский язык и адаптированы под работу. Источники отслеживаются через frontmatter каждого SKILL.md.

> **Есть скилы которые мы намеренно не трогаем** — сложные официальные скилы Anthropic с Python-скриптами и eval-системами. Подробнее: [OFFICIAL.md](./OFFICIAL.md)

## Установка

```bash
# Добавить маркетплейс (один раз)
# В ~/.claude/settings.json добавить:
{
  "extraKnownMarketplaces": {
    "my-claude-skills": {
      "source": { "source": "github", "repo": "Ya-Rog/my-claude-skills" },
      "autoUpdate": true
    }
  }
}

# Затем включить нужные скилы в enabledPlugins:
"skill-discover@my-claude-skills": true
```

---

## 🔧 Meta — управление скилами

| Скил | Что делает | Триггер | Источник |
|---|---|---|---|
| **skill-discover** | Поиск и установка новых скилов из маркетплейсов | "найди скил для X", "есть ли плагин для Y" | local |
| **skill-maintain** | Проверка обновлений переведённых скилов | "проверь обновления скилов" | local |

> `skill-creator` — официальный скил Anthropic, устанавливается автоматически. Подробнее: [OFFICIAL.md](./OFFICIAL.md)

---

## ⚙️ Engineering — разработка

| Скил | Что делает | Триггер | Источник |
|---|---|---|---|
| **api-design** | REST и GraphQL API дизайн, паттерны, версионирование | "спроектируй API", "как назвать эндпоинт" | local |
| **component-architecture** | Архитектура UI-компонентов, presentation/container, custom hooks | "как организовать компоненты" | local |
| **error-handling** | Обработка ошибок, retry, circuit breaker, graceful degradation | "как обрабатывать ошибки", "circuit breaker" | local |

---

## 🗄️ Database

| Скил | Что делает | Триггер | Источник |
|---|---|---|---|
| **postgres-design** | Схемы PostgreSQL, типы данных, индексы, партиционирование | "создай таблицу", "как проиндексировать", "PostgreSQL схема" | local |

---

## Структура репозитория

```
my-claude-skills/
  .claude-plugin/
    marketplace.json          ← список плагинов для Claude Code
  skills/
    meta/
      skill-discover/SKILL.md
      skill-maintain/SKILL.md
    engineering/
      api-design/SKILL.md
      component-architecture/SKILL.md
      error-handling/SKILL.md
    database/
      postgres-design/SKILL.md
  CATALOG.md                  ← этот файл
  OFFICIAL.md                 ← официальные скилы Anthropic (не трогаем)
```

---

## Как добавить новый скил

1. Найди скил через `skill-discover`
2. Переведи SKILL.md на русский
3. Добавь frontmatter с метаданными источника
4. Положи в соответствующую категорию: `skills/<категория>/<имя>/SKILL.md`
5. Добавь запись в `.claude-plugin/marketplace.json`
6. Добавь `"<имя>@my-claude-skills": true` в `~/.claude/settings.json`
7. Закоммить и запушь

```bash
cd ~/my-claude-skills
git add .
git commit -m "Добавлен скил <name>"
git push
```

---

## Как обновить скил

```bash
# 1. Проверить что изменилось в оригинале
# (используй skill-maintain или вручную через GitHub)

# 2. Обновить SKILL.md
# 3. Обновить source_sha и source_checked в frontmatter

git add skills/<category>/<name>/SKILL.md
git commit -m "Обновлён скил <name> до SHA <sha[:8]>"
git push
```
