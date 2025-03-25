# Лекция: Интеграция tRPC с React-приложением

## 1. Введение в tRPC
### Что такое tRPC?
- **tRPC** – инструмент для синхронизации типов между бэкендом и фронтендом.
- Не зависит от React, но имеет специальный адаптер для удобной интеграции.
- **Преимущества**:
  - Автоматическая типизация запросов и ответов.
  - Упрощение работы с API (автодополнение, валидация типов).
  - Интеграция с React Query для управления состоянием запросов.

---

## 2. Настройка проекта
### Установка зависимостей
```bash
cd frontend
pnpm add @trpc/client @trpc/react-query @tanstack/react-query
```

### Структура проекта
- Создайте файл для клиента tRPC: `src/lib/trpc.ts`.
- Используйте расширение `.tsx` для файлов с JSX-компонентами.

---

## 3. Настройка монорепозитория с PNPM Workspaces
### Шаги:
1. Создайте корневой файл `pnpm-workspace.yaml`:
```yaml
packages:
  - "backend"
  - "frontend"
```
2. Переименуйте папки в соответствии со стандартами:
   - Бэкенд: `@your-project/backend`
   - Фронтенд: `@your-project/frontend`
3. Добавьте зависимость фронтенда на бэкенд в `frontend/package.json`:
```json
"dependencies": {
  "@your-project/backend": "workspace:*"
}
```
4. Выполните установку зависимостей в корне проекта:
```bash
pnpm install
```

---

## 4. Импорт типов с бэкенда
- Импортируйте тип роутера из бэкенда:
```ts
import type { AppRouter } from "@your-project/backend";
```
- Создайте клиент tRPC с указанием типа роутера:
```ts
export const trpc = createTRPCReact<AppRouter>();
```

---

## 5. Создание клиентов tRPC и React Query
### Настройка клиентов
```tsx
// src/lib/trpc.ts
const queryClient = new QueryClient({
  defaultOptions: { queries: { refetchOnWindowFocus: false } }
});

const trpcClient = createTRPCClient({
  url: "http://localhost:3000/trpc",
  transformer: superjson,
});
```

---

## 6. Интеграция провайдеров
### Оберните приложение в провайдеры
```tsx
// App.tsx
<QueryClientProvider client={queryClient}>
  <trpc.Provider client={trpcClient} queryClient={queryClient}>
    <AppRouter />
  </trpc.Provider>
</QueryClientProvider>
```

---

## 7. Выполнение запросов
### Пример использования
```tsx
// pages/AllIdeasPage.tsx
const { data, isLoading, error } = trpc.idea.getAll.useQuery();

if (isLoading) return <div>Loading...</div>;
if (error) return <div>Error: {error.message}</div>;

return (
  <div>
    {data?.ids.map((id) => (
      <div key={id}>{id}</div>
    ))}
  </div>
);
```

---

## 8. Решение проблем с CORS
### Настройка бэкенда
1. Установите пакет:
```bash
cd backend
pnpm add cors @types/cors
```
2. Добавьте middleware:
```ts
import cors from "cors";
app.use(cors({ origin: "http://localhost:5173" })); // Укажите адрес фронтенда
```

---

## 9. Тестирование и отладка
### Рекомендации:
- Используйте `isLoading` для индикации загрузки.
- Обрабатывайте ошибки через `error.message`.
- Проверяйте Network-запросы в DevTools:
  - **Batching**: tRPC объединяет несколько запросов в один HTTP-запрос.
  - Типизация ответов автоматически соответствует бэкенду.

---

## 10. Частые ошибки
1. **Опечатки в workspace-конфигах**:
   - Проверяйте названия папок в `pnpm-workspace.yaml`.
2. **Проблемы с импортом типов**:
   - Убедитесь, что версии пакетов на фронтенде и бэкенде совместимы.
3. **CORS-ошибки**:
   - Всегда настраивайте `cors()` на бэкенде для разработки.

---

## Заключение
- **tRPC + React Query** – мощная связка для типизированного взаимодействия с API.
- Монорепозиторий упрощает синхронизацию типов между частями проекта.
- Дальнейшие шаги: 
  - Реализация мутаций
  - Оптимизация запросов
  - Интеграция аутентификации

**Важно**: Ошибки – нормальная часть разработки. Используйте их как возможность глубже понять систему!