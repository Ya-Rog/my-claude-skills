---
name: postgres-design
description: Проектирование PostgreSQL схем — типы данных, индексы, ограничения, паттерны производительности и продвинутые возможности БД. Используй при создании таблиц, выборе типов данных, настройке индексов, партиционировании или оптимизации схемы PostgreSQL.
source_repo: local
source_sha: local
source_checked: 2026-03-16
category: database
requires_mcp: []
---

# Проектирование таблиц PostgreSQL

## Основные правила

- Определяй **PRIMARY KEY** для справочных таблиц. Предпочитай `BIGINT GENERATED ALWAYS AS IDENTITY`; используй `UUID` только когда нужна глобальная уникальность.
- **Нормализуй до 3НФ** для устранения избыточности; денормализуй **только** для измеренных, высокорентабельных чтений.
- Добавляй **NOT NULL** везде где это семантически требуется; используй **DEFAULT** для частых значений.
- Создавай **индексы для путей доступа которые реально используешь**: PK/unique (авто), **FK колонки (вручную!)**, частые фильтры/сортировки.
- Предпочитай **TIMESTAMPTZ** для времени событий; **NUMERIC** для денег; **TEXT** для строк; **BIGINT** для целых чисел.

## Особенности PostgreSQL

- **Идентификаторы**: без кавычек → в нижний регистр. Используй `snake_case`.
- **Unique + NULL**: UNIQUE допускает несколько NULL. Используй `NULLS NOT DISTINCT` (PG15+) чтобы ограничить одним NULL.
- **Индексы FK**: PostgreSQL **не** создаёт индексы для FK-колонок автоматически. Добавляй вручную.
- **Нет тихих преобразований**: переполнение длины/точности вызывает ошибку (не обрезку).
- **Пробелы в последовательностях** — нормально; не пытайся "исправить" это.
- **MVCC**: обновления/удаления оставляют мёртвые tuple; vacuum их обрабатывает.

## Типы данных

| Для | Используй | НЕ используй |
|---|---|---|
| ID | `BIGINT GENERATED ALWAYS AS IDENTITY` | `SERIAL` |
| Время | `TIMESTAMPTZ` | `TIMESTAMP` без timezone |
| Деньги | `NUMERIC(p,s)` | `MONEY`, `FLOAT` |
| Строки | `TEXT` | `CHAR(n)`, `VARCHAR(n)` |
| UUID | `gen_random_uuid()` | — |
| JSON | `JSONB` | `JSON` (только если нужен порядок ключей) |

## Индексы

- **B-tree**: по умолчанию для равенства/диапазона (`=`, `<`, `>`, `ORDER BY`)
- **Составной**: порядок важен — индекс используется если есть равенство на самом левом префиксе
- **Покрывающий**: `CREATE INDEX ON tbl (id) INCLUDE (name, email)` — index-only scan
- **Частичный**: `CREATE INDEX ON tbl (user_id) WHERE status = 'active'` — для горячих подмножеств
- **По выражению**: `CREATE INDEX ON tbl (LOWER(email))` — выражение должно точно совпадать в WHERE
- **GIN**: JSONB, массивы, полнотекстовый поиск
- **GiST**: диапазоны, геометрия, exclusion constraints

## Ограничения

```sql
-- FK: всегда указывай действие при удалении/обновлении
REFERENCES users(user_id) ON DELETE CASCADE

-- CHECK: NULL проходит проверку (трёхзначная логика)
-- Комбинируй с NOT NULL чтобы принудить значение
price NUMERIC NOT NULL CHECK (price > 0)

-- EXCLUDE: предотвращает перекрывающиеся значения
EXCLUDE USING gist (room_id WITH =, booking_period WITH &&)
```

## Партиционирование

Используй для очень больших таблиц (>100M строк) где запросы фильтруют по ключу партиции:

```sql
-- RANGE: типично для time-series
CREATE TABLE logs (created_at TIMESTAMPTZ NOT NULL)
PARTITION BY RANGE (created_at);

CREATE TABLE logs_2026_01 PARTITION OF logs
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

## Безопасная эволюция схемы

- **Транзакционный DDL**: большинство DDL-операций можно откатить — тестируй в `BEGIN; ... ROLLBACK;`
- **Concurrent index**: `CREATE INDEX CONCURRENTLY` — не блокирует запись, но нельзя в транзакции
- **Летучие DEFAULT**: добавление `NOT NULL` колонки с `now()` или `gen_random_uuid()` перезаписывает всю таблицу

## Примеры

```sql
-- Пользователи
CREATE TABLE users (
  user_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE UNIQUE INDEX ON users (LOWER(email));
CREATE INDEX ON users (created_at);

-- Заказы
CREATE TABLE orders (
  order_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(user_id),
  status TEXT NOT NULL DEFAULT 'PENDING' CHECK (status IN ('PENDING','PAID','CANCELED')),
  total NUMERIC(10,2) NOT NULL CHECK (total > 0),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX ON orders (user_id);  -- FK индекс вручную!
CREATE INDEX ON orders (created_at);

-- JSONB
CREATE TABLE profiles (
  user_id BIGINT PRIMARY KEY REFERENCES users(user_id),
  attrs JSONB NOT NULL DEFAULT '{}'
);
CREATE INDEX profiles_attrs_gin ON profiles USING GIN (attrs);
```

## Расширения

- `pg_trgm` — нечёткий текстовый поиск, `LIKE '%pattern%'`
- `timescaledb` — time-series: автоматическое партиционирование, сжатие
- `postgis` — геопространственный поиск
- `pgvector` — векторный поиск для embeddings
- `pgaudit` — аудит-логирование всей активности БД
