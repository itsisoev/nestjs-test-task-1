# NestJS Test Task 1

## ЗАДАЧА

Развернуть сервис на **NestJS**, PostgreSQL, ClickHouse, NATS, Redis, Docker.

---

### ⚙️ Требования:

- Использовать **NestJS** в качестве backend-фреймворка
- Использовать **PostgreSQL** в качестве основной БД
- Использовать **ClickHouse** для логирования
- Использовать **NATS** для очередей
- Использовать **Redis** для кеширования
- Использовать **Docker** для запуска всей системы

---

### 📦 Модели данных и миграции

- Создать таблицу `goods` и `projects` в PostgreSQL
- Проставить **primary key** и **индексы** на указанные поля
- В миграциях PostgreSQL:
    - Для `goods`:
        - `id = serial`
        - `priority` — автоинкрементируемый от max + 1
        - Индекс на `priority`
    - Для `projects`:
        - `id = serial`
        - `name = string`
        - При накатке миграций — добавить одну запись в таблицу `projects`:
            - `id = 1`
            - `name = 'Первая запись'`

---

### ✏️ CRUD методы (таблица GOODS)

Реализовать REST API:

| Метод  | Описание                         |
|--------|----------------------------------|
| GET    | Получить данные по ID            |
| POST   | Создать запись                   |
| PATCH  | Обновить запись по ID            |
| DELETE | Удалить запись по ID             |

---

### 🧠 Особенности

- При **редактировании данных**:
    - Использовать **транзакции**
    - Использовать `SELECT FOR UPDATE`
    - Валидировать поля
    - Инвалидировать кеш в Redis
- При **удалении или редактировании** несуществующей записи:
    - Вернуть статус **404**
    - Формат ошибки:
      ```json
      {
        "code": 3,
        "message": "errors.common.notFound",
        "details": {}
      }
      ```
- При **GET**:
    - Пытаемся получить данные из Redis
    - Если нет — берем из PostgreSQL и кладем в Redis на **60 секунд**
- При **POST/PATCH/DELETE**:
    - Пишем лог в **ClickHouse** через **очередь NATS**
    - Логи отправляются пачками

---

### 💾 ClickHouse логи

При каждом изменении:

```json
{
  "timestamp": "2024-01-01T12:00:00Z",
  "operation": "CREATE" | "UPDATE" | "DELETE",
  "entity": "goods",
  "entity_id": 5,
  "data": { ... }
}
