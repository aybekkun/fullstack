### Лекция: Внедрение tRPC в серверное приложение. Основы

---

#### **Цель урока**:  
Настроить tRPC для типизированного взаимодействия между бэкендом и фронтендом. Создать базовый роутер и процедуры.

---

### **1. Что такое tRPC?**
- **Определение**: Библиотека для синхронизации типов между бэкендом и фронтендом.
- **Преимущества**:
  - Автоматическая проверка типов на обоих концах.
  - Упрощение API-взаимодействий.
  - Независимость от фреймворков (работает с Express, Next.js и др.).

---

### **2. Установка зависимостей**
1. Установите `@trpc/server`:
   ```bash
   pnpm add @trpc/server
   ```
2. Для Express потребуется адаптер:
   ```bash
   pnpm add @trpc/server@next @trpc/client@next
   ```

---

### **3. Настройка tRPC**
#### **Шаг 1: Создание файла `src/trpc.ts`**
```typescript
import { initTRPC } from '@trpc/server';

// Инициализация tRPC
const t = initTRPC.create();

// Экспорт роутера и процедур
export const router = t.router;
export const publicProcedure = t.procedure;
```

---

### **4. Создание роутера**
#### **Пример: Роутер для работы с идеями**
1. Создайте файл `src/routers/ideas.ts`:
   ```typescript
   import { publicProcedure, router } from '../trpc';

   // Пример данных (временное хранилище)
   const ideas = [{ id: 1, text: 'Первая идея' }];

   export const ideasRouter = router({
     getIdeas: publicProcedure.query(() => {
       return { ideas };
     }),
   });
   ```

2. Объедините роутеры в корневой роутер (`src/trpc.ts`):
   ```typescript
   import { ideasRouter } from './routers/ideas';

   export const appRouter = router({
     ideas: ideasRouter,
   });

   export type AppRouter = typeof appRouter;
   ```

---

### **5. Интеграция с Express**
#### **Шаг 1: Подключение tRPC к серверу**
```typescript
import { createExpressMiddleware } from '@trpc/server/adapters/express';
import { appRouter } from './trpc';

// Удалите старый эндпоинт /ideas (если был)
app.use(
  '/trpc', // Базовый URL для tRPC
  createExpressMiddleware({
    router: appRouter,
  })
);
```

#### **Шаг 2: Запуск сервера**
```typescript
app.listen(3000, () => {
  console.log('Сервер запущен: http://localhost:3000/trpc/ideas.getIdeas');
});
```

---

### **6. Пример запроса**
- **Структура URL**:  
  `http://localhost:3000/trpc/[роутер].[процедура]`
- **Пример**:  
  ```bash
  curl http://localhost:3000/trpc/ideas.getIdeas
  ```
- **Ответ**:
  ```json
  {
    "result": {
      "data": { "ideas": [{ "id": 1, "text": "Первая идея" }] }
    }
  }
  ```

---

### **7. Особенности tRPC**
- **Процедуры**:
  - `query`: Для GET-запросов (получение данных).
  - `mutation`: Для POST/PUT/DELETE (изменение данных).
- **Типы**: Экспортируйте `AppRouter` для использования на фронтенде:
  ```typescript
  export type AppRouter = typeof appRouter;
  ```

---

### **8. Частые ошибки**
1. **Ошибка "Cannot find module '@trpc/server'"**:
   - Переустановите зависимости:
     ```bash
     pnpm install
     ```
2. **Эндпоинты не работают**:
   - Убедитесь, что базовый URL (`/trpc`) совпадает на клиенте и сервере.
3. **Типы не синхронизируются**:
   - Проверьте экспорт `AppRouter` и его импорт на фронтенде.

---

### **Итоги урока**
- Настроен tRPC для типизированного API.
- Создан роутер с процедурой для получения данных.
- Сервер интегрирован с Express.
- Экспортированы типы для фронтенда.

---

**Следующий шаг**:  
Подключение фронтенда к tRPC, использование типов и автоматическое дополнение кода.