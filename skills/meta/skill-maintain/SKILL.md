---
name: skill-maintain
description: Проверка обновлений для переведённых скилов из внешних репозиториев. Используй когда хочешь узнать устарели ли твои скилы, что изменилось в оригинале, или нужно ли обновить перевод. Триггеры: "проверь обновления скилов", "есть ли новые версии скилов", "обновился ли skill-X".
source_repo: local
source_sha: local
source_checked: 2026-03-16
category: meta
requires_mcp: []
---

# Skill Maintain — проверка обновлений скилов

## Обзор

Система отслеживания обновлений для личного маркетплейса скилов. Сравнивает текущий SHA оригинального файла с записанным в frontmatter и показывает что изменилось.

## Алгоритм проверки

### Шаг 1: Собрать метаданные всех скилов

```bash
# Найти все SKILL.md в личном маркетплейсе
find ~/my-claude-skills/skills -name "SKILL.md"
```

Для каждого файла извлечь из frontmatter:
- `source_repo` — откуда взят скил
- `source_path` — путь к файлу в оригинальном репо
- `source_sha` — SHA коммита когда брали
- `source_checked` — дата последней проверки

Скилы с `source_repo: local` — пропускаем (нет оригинала для сравнения).

### Шаг 2: Проверить актуальный SHA через GitHub API

```bash
# Получить последний коммит для конкретного файла
curl -s "https://api.github.com/repos/{owner}/{repo}/commits?path={file_path}&per_page=1" \
  -H "Authorization: token $GITHUB_TOKEN"
```

Если GitHub MCP доступен — использовать его напрямую (быстрее и надёжнее).

Через web_fetch (без MCP):
```
URL: https://api.github.com/repos/anthropics/claude-plugins-official/commits?path=plugins/skill-creator/skills/skill-creator/SKILL.md&per_page=1
```

### Шаг 3: Сравнить SHA

| Статус | Условие |
|---|---|
| ✅ Актуален | `source_sha` совпадает с последним коммитом |
| ⚠️ Устарел | `source_sha` отличается от последнего коммита |
| ❓ Неизвестен | `source_repo: local` — нет оригинала |

### Шаг 4: Показать diff для устаревших

Для каждого устаревшего скила:

```bash
# Получить содержимое текущего оригинала
curl -s "https://raw.githubusercontent.com/{owner}/{repo}/main/{path}"
```

Показать пользователю:
1. Что именно изменилось в оригинале (ключевые отличия)
2. Нужно ли обновить перевод (если изменения затрагивают инструкции)
3. Если изменения косметические — можно пропустить

### Шаг 5: Обновить after ревью

После того как пользователь решил принять изменения:

```bash
cd ~/my-claude-skills

# Обновить переведённый файл с новыми инструкциями
# Обновить frontmatter:
#   source_sha: <новый SHA>
#   source_checked: <сегодняшняя дата>

git add skills/<category>/<skill-name>/SKILL.md
git commit -m "Обновлён скил <name> до версии <sha[:8]>"
git push
```

## Отчёт проверки

Показывай в формате:

```
Проверка обновлений скилов: 2026-03-16
════════════════════════════════════════

✅ АКТУАЛЬНЫ (не изменились):
  - postgres-design (local)
  - component-architecture (local)

⚠️ УСТАРЕЛИ (есть изменения в оригинале):
  - skill-creator
    Оригинал: anthropics/claude-plugins-official
    Наш SHA:  2cd88e79
    Новый SHA: a4f92b11
    Изменено: добавлен раздел про Cowork-среду (15 строк)
    → Рекомендация: обновить (новые инструкции)

  - frontend-design
    Оригинал: anthropics/claude-plugins-official
    Наш SHA:  55b58ec6
    Новый SHA: 55b58ec6  ← совпадает! (false alarm)

❓ БЕЗ ИСТОЧНИКА (local):
  - api-design, error-handling, skill-discover, skill-maintain

Итого: 1 устаревший, обновление рекомендуется
```

## Когда запускать

- **Раз в месяц** — стандартный ритм
- **Перед важным проектом** — убедиться что скилы актуальны
- **После новостей об обновлении Claude Code** — Anthropic мог обновить официальные скилы

## Добавление нового скила в систему отслеживания

При добавлении скила из внешнего источника всегда заполняй frontmatter:

```yaml
source_repo: anthropics/claude-plugins-official
source_path: plugins/skill-name/skills/skill-name/SKILL.md
source_sha: <sha текущего коммита>
source_checked: <сегодня>
```

Получить SHA последнего коммита:
```bash
curl -s "https://api.github.com/repos/anthropics/claude-plugins-official/commits?path=<path>&per_page=1" | grep '"sha"' | head -1
```
