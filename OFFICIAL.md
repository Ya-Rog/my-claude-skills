# Официальные скилы — не переносим в my-claude-skills

Эти скилы **намеренно не переводятся и не переносятся** в my-claude-skills.

## Правило одной строкой

> Если скил **от Anthropic** или **не нужно улучшать под себя** — оставляем официальным. Он сам обновится.

Подробная таблица решений — в skill-discover (используется каждый раз при поиске скила).

---

## Официальные скилы Anthropic ⚠️

Всё от Anthropic — не трогаем. Они сами знают как лучше, у них eval-инфраструктура, Python-скрипты, система тестирования. Автообновление = всегда последняя версия без потерь.

### Документы — из маркетплейса `anthropic-skills`

| Скил | Зачем | Как загружается |
|---|---|---|
| `pdf` | Читать, создавать, объединять, разбивать PDF, OCR | Автоматически |
| `docx` | Создавать и редактировать Word-документы | Автоматически |
| `xlsx` | Работа с Excel-таблицами | Автоматически |
| `pptx` | Создавать и редактировать презентации | Автоматически |
| `schedule` | Создавать запланированные задачи | Автоматически |

### Инструменты — из `claude-plugins-official`

| Скил / Плагин | Зачем | Включён в settings.json |
|---|---|---|
| `skill-creator` | Создание и тестирование скилов с eval-системой | Авто (anthropic-skills) |
| `frontend-design` | Production-grade UI с уникальным дизайном | `frontend-design@claude-plugins-official` |
| `agent-sdk-dev` | Разработка агентов на Claude Agent SDK | `agent-sdk-dev@claude-plugins-official` |
| `claude-code-setup` | Анализ кодовой базы и рекомендации по автоматизации | `claude-code-setup@claude-plugins-official` |
| `playwright` | Браузерное тестирование | `playwright@claude-plugins-official` |
| `security-guidance` | Рекомендации по безопасности кода | `security-guidance@claude-plugins-official` |
| `pyright-lsp` | LSP-анализ Python кода (типы, ошибки) | `pyright-lsp@claude-plugins-official` |
| `typescript-lsp` | LSP-анализ TypeScript кода | `typescript-lsp@claude-plugins-official` |
| `commit-commands` | Git коммиты, пуш, PR одной командой | `commit-commands@claude-plugins-official` |
| `hookify` | Настройка хуков — правила поведения Claude | `hookify@claude-plugins-official` |
| `claude-md-management` | Аудит и улучшение CLAUDE.md файлов | `claude-md-management@claude-plugins-official` |
| `code-review` | Ревью pull request'ов | `code-review@claude-plugins-official` |
| `pr-review-toolkit` | Комплексный анализ PR (несколько агентов) | `pr-review-toolkit@claude-plugins-official` |

---

## Доступны в маркетплейсе, не установлены — если понадобятся, ставим официально

| Скил | Откуда | Зачем |
|---|---|---|
| `plugin-dev` (7 скилов) | claude-plugins-official | Разработка плагинов, MCP, хуков |

---

## Как добавлять сюда новые скилы

Если решил оставить скил официальным (не переносить в my-claude-skills) — добавь строку в таблицу выше. Укажи:
- Название
- Откуда (маркетплейс)
- Зачем

Последнее обновление: 2026-03-17
