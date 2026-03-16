---
name: component-architecture
description: Масштабируемые архитектурные паттерны компонентов для создания модульных и переиспользуемых компонентов. Используй когда проектируешь структуру UI-компонентов, выбираешь между паттернами, или организуешь папки компонентов в проекте.
source_repo: local
source_sha: local
source_checked: 2026-03-16
category: engineering
requires_mcp: []
---

# Архитектура компонентов

## Основные принципы

При построении компонентов:

1. **Single Responsibility** — один компонент = одна задача
2. **TypeScript Props** — интерфейс props с типизацией для безопасности
3. **Разделение ответственности** — presentation vs container компоненты
4. **Custom hooks** — для переиспользуемой логики
5. **Композиция вместо наследования**
6. **Чёткая структура папок**: `/components/Feature/index.tsx` + стили + тесты

## Паттерны

### Presentation vs Container

```tsx
// Presentation — только отображение, получает данные через props
const UserCard = ({ name, email, avatar }: UserCardProps) => (
  <div className="user-card">
    <img src={avatar} alt={name} />
    <h3>{name}</h3>
    <p>{email}</p>
  </div>
);

// Container — управляет состоянием и данными
const UserCardContainer = ({ userId }: { userId: string }) => {
  const { user, loading } = useUser(userId);
  if (loading) return <Skeleton />;
  return <UserCard {...user} />;
};
```

### Custom Hook для логики

```tsx
// Логика вынесена в хук — компонент остаётся простым
const useUser = (userId: string) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);

  return { user, loading };
};
```

### Структура папок

```
components/
  UserCard/
    index.tsx        ← основной компонент
    UserCard.test.tsx ← тесты рядом с кодом
    UserCard.module.css ← стили
    types.ts         ← TypeScript типы (если объёмные)
```

## Когда разделять компонент

- Компонент > 150 строк → рассмотри разбивку
- Логика повторяется → вынеси в хук или утилиту
- Разные части меняются по разным причинам → разные компоненты
- Тест требует много setup → компонент делает слишком много
