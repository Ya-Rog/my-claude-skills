---
name: error-handling
description: Мастерство обработки ошибок — исключения, Result типы, распространение ошибок и graceful degradation для создания устойчивых приложений. Используй при реализации обработки ошибок, проектировании отказоустойчивых API, отладке production-проблем или построении fault-tolerant систем.
source_repo: local
source_sha: local
source_checked: 2026-03-16
category: engineering
requires_mcp: []
---

# Паттерны обработки ошибок

Строй устойчивые приложения со стратегиями обработки ошибок, которые корректно обрабатывают сбои и обеспечивают отличный опыт отладки.

## Когда использовать

- Реализация обработки ошибок в новых функциях
- Проектирование отказоустойчивых API
- Отладка production-проблем
- Улучшение надёжности приложения
- Реализация паттернов retry и circuit breaker
- Построение fault-tolerant распределённых систем

## Ключевые концепции

### Исключения vs Result Types

| Подход | Когда использовать |
|---|---|
| **Исключения** | Неожиданные ошибки, исключительные условия |
| **Result Types** | Ожидаемые ошибки, ошибки валидации |
| **Panic/Crash** | Неисправимые ошибки, баги программирования |

### Категории ошибок

**Исправимые:** таймауты сети, отсутствующие файлы, невалидный ввод, rate limits API

**Неисправимые:** out of memory, stack overflow, баги программирования

## Python: обработка ошибок

### Иерархия исключений

```python
class ApplicationError(Exception):
    def __init__(self, message: str, code: str = None, details: dict = None):
        super().__init__(message)
        self.code = code
        self.details = details or {}

class ValidationError(ApplicationError):
    pass

class NotFoundError(ApplicationError):
    pass

class ExternalServiceError(ApplicationError):
    def __init__(self, message: str, service: str, **kwargs):
        super().__init__(message, **kwargs)
        self.service = service

# Использование
def get_user(user_id: str) -> User:
    user = db.query(User).filter_by(id=user_id).first()
    if not user:
        raise NotFoundError("Пользователь не найден",
                           code="USER_NOT_FOUND",
                           details={"user_id": user_id})
    return user
```

### Retry с экспоненциальным backoff

```python
def retry(max_attempts=3, backoff_factor=2.0, exceptions=(Exception,)):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt < max_attempts - 1:
                        time.sleep(backoff_factor ** attempt)
                    else:
                        raise
        return wrapper
    return decorator

@retry(max_attempts=3, exceptions=(NetworkError,))
def fetch_data(url: str) -> dict:
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()
```

## TypeScript: обработка ошибок

### Result Type (функциональный подход)

```typescript
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

const Ok = <T>(value: T): Result<T, never> => ({ ok: true, value });
const Err = <E>(error: E): Result<never, E> => ({ ok: false, error });

// Использование
function parseJSON<T>(json: string): Result<T, SyntaxError> {
  try {
    return Ok(JSON.parse(json) as T);
  } catch (error) {
    return Err(error as SyntaxError);
  }
}

const result = parseJSON<User>(userJson);
if (result.ok) {
  console.log(result.value.name);
} else {
  console.error("Ошибка парсинга:", result.error.message);
}
```

## Универсальные паттерны

### Circuit Breaker (предотвращение каскадных сбоев)

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=timedelta(seconds=60)):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.state = CircuitState.CLOSED  # CLOSED / OPEN / HALF_OPEN
        self.last_failure_time = None

    def call(self, func):
        if self.state == CircuitState.OPEN:
            if datetime.now() - self.last_failure_time > self.timeout:
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker ОТКРЫТ — запросы заблокированы")
        try:
            result = func()
            self._on_success()
            return result
        except Exception:
            self._on_failure()
            raise
```

### Graceful Degradation (плавная деградация)

```python
def with_fallback(primary, fallback, log_error=True):
    """Пробуем основную функцию, при ошибке — запасная."""
    try:
        return primary()
    except Exception as e:
        if log_error:
            logger.error(f"Основная функция упала: {e}")
        return fallback()

# Использование
def get_user_profile(user_id):
    return with_fallback(
        primary=lambda: fetch_from_cache(user_id),
        fallback=lambda: fetch_from_database(user_id)
    )
```

### Агрегация ошибок (собираем все ошибки, не падаем на первой)

```typescript
class ErrorCollector {
  private errors: Error[] = [];
  add(error: Error) { this.errors.push(error); }
  hasErrors() { return this.errors.length > 0; }
  throw() { throw new AggregateError(this.errors, `${this.errors.length} ошибок`); }
}

function validateUser(data: any): User {
  const errors = new ErrorCollector();
  if (!data.email) errors.add(new ValidationError("Email обязателен"));
  if (!data.name) errors.add(new ValidationError("Имя обязательно"));
  if (errors.hasErrors()) errors.throw();
  return data as User;
}
```

## Лучшие практики

1. **Fail Fast** — валидируй ввод рано, падай быстро
2. **Сохраняй контекст** — включай stack traces, метаданные, временные метки
3. **Понятные сообщения** — объясняй что случилось и как исправить
4. **Логируй уместно** — ошибка = логируй, ожидаемый сбой = не спамь логи
5. **Обрабатывай на нужном уровне** — лови там, где можешь обработать значимо
6. **Освобождай ресурсы** — используй try-finally, context managers, defer
7. **Не глотай ошибки** — логируй или пробрасывай дальше, не игнорируй молча

## Типичные ошибки

- **Слишком широкий catch** — `except Exception` скрывает баги
- **Пустые catch блоки** — молчаливое поглощение ошибок
- **Двойное логирование** — логировать И пробрасывать создаёт дубли
- **Не освобождать ресурсы** — забыть закрыть файлы, соединения
- **Неинформативные сообщения** — "Произошла ошибка" не помогает
- **Игнорировать async ошибки** — необработанные Promise rejections
