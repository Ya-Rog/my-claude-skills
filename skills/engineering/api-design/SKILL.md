---
name: api-design
description: Мастерство REST и GraphQL API дизайна — создание интуитивных, масштабируемых и поддерживаемых API. Используй когда проектируешь новые API, рефакторишь существующие, устанавливаешь стандарты дизайна или проверяешь спецификации перед реализацией.
source_repo: local
source_sha: local
source_checked: 2026-03-16
category: engineering
requires_mcp: []
---

# Проектирование API

Освой принципы проектирования REST и GraphQL API для создания интуитивных, масштабируемых и поддерживаемых интерфейсов.

## Когда использовать

- Проектирование новых REST или GraphQL API
- Рефакторинг существующих API для улучшения удобства
- Установка стандартов проектирования API для команды
- Проверка спецификаций API перед реализацией
- Миграция между парадигмами API (REST → GraphQL и т.д.)
- Создание документации API, удобной для разработчиков

## Основные концепции

### 1. Принципы RESTful дизайна

**Ресурсно-ориентированная архитектура:**
- Ресурсы — это существительные (`users`, `orders`, `products`), не глаголы
- HTTP методы для действий (GET, POST, PUT, PATCH, DELETE)
- URL представляют иерархию ресурсов
- Согласованные соглашения об именовании

**Семантика HTTP методов:**
- `GET` — получить ресурс (идемпотентный, безопасный)
- `POST` — создать новый ресурс
- `PUT` — заменить весь ресурс (идемпотентный)
- `PATCH` — частичное обновление ресурса
- `DELETE` — удалить ресурс (идемпотентный)

### 2. Принципы GraphQL дизайна

**Schema-First разработка:**
- Типы определяют доменную модель
- Queries для чтения данных
- Mutations для изменения данных
- Subscriptions для обновлений в реальном времени

### 3. Стратегии версионирования API

```
# URL версионирование
/api/v1/users
/api/v2/users

# Header версионирование
Accept: application/vnd.api+json; version=1
```

## REST паттерны

### Паттерн 1: Дизайн коллекций ресурсов

```python
# Хорошо: ресурсно-ориентированные эндпоинты
GET    /api/users              # Список пользователей (с пагинацией)
POST   /api/users              # Создать пользователя
GET    /api/users/{id}         # Получить конкретного пользователя
PUT    /api/users/{id}         # Заменить пользователя
PATCH  /api/users/{id}         # Обновить поля пользователя
DELETE /api/users/{id}         # Удалить пользователя

# Вложенные ресурсы
GET    /api/users/{id}/orders  # Заказы пользователя
POST   /api/users/{id}/orders  # Создать заказ для пользователя

# Плохо: глагольно-ориентированные эндпоинты (избегать)
POST   /api/createUser
POST   /api/getUserById
```

### Паттерн 2: Пагинация и фильтрация

```python
@app.get("/api/users", response_model=PaginatedResponse)
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    status: Optional[str] = Query(None),
    search: Optional[str] = Query(None)
):
    # Применить фильтры, подсчитать total, вернуть страницу
    ...
```

### Паттерн 3: Коды ошибок и статусов

```python
# Согласованные ответы с ошибками
STATUS_CODES = {
    "success": 200,
    "created": 201,
    "no_content": 204,
    "bad_request": 400,
    "unauthorized": 401,
    "forbidden": 403,
    "not_found": 404,
    "conflict": 409,
    "unprocessable": 422,
    "internal_error": 500
}
```

## GraphQL паттерны

### Паттерн 1: Дизайн схемы

```graphql
type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!
  orders(first: Int = 20, after: String): OrderConnection!
}

# Пагинация в стиле Relay
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

# Payload для mutations — возвращаем данные И ошибки
type CreateUserPayload {
  user: User
  errors: [Error!]
}
```

### Паттерн 2: DataLoader (предотвращение N+1)

```python
class OrdersByUserLoader(DataLoader):
    async def batch_load_fn(self, user_ids):
        # Один запрос для всех пользователей вместо N запросов
        orders = await fetch_orders_by_user_ids(user_ids)
        orders_by_user = {}
        for order in orders:
            orders_by_user.setdefault(order["user_id"], []).append(order)
        return [orders_by_user.get(uid, []) for uid in user_ids]
```

## Лучшие практики

### REST API
1. **Согласованность именования** — множественное число для коллекций (`/users`, не `/user`)
2. **Stateless** — каждый запрос содержит всю необходимую информацию
3. **Правильные HTTP статусы** — 2xx успех, 4xx ошибки клиента, 5xx ошибки сервера
4. **Версионирование** — планируй breaking changes с первого дня
5. **Пагинация** — всегда пагинируй большие коллекции
6. **Rate Limiting** — защити API от злоупотреблений
7. **Документация** — используй OpenAPI/Swagger для интерактивной документации

### GraphQL API
1. **Schema First** — проектируй схему перед написанием resolver'ов
2. **DataLoaders** — предотвращай N+1 проблему
3. **Cursor-based пагинация** — используй Relay spec
4. **Структурированные ошибки** — возвращай ошибки в payload mutations

## Типичные ошибки

- **Over-fetching/Under-fetching (REST)** — решается в GraphQL через DataLoaders
- **Breaking Changes** — версионируй API или используй deprecation
- **Несогласованный формат ошибок** — стандартизируй ответы с ошибками
- **Отсутствие Rate Limits** — API без лимитов уязвим для злоупотреблений
- **Плохая документация** — недокументированные API раздражают разработчиков
- **Тесная связь с БД** — структура API не должна копировать структуру базы данных
