
Участвовал в проектировании структуры базы данных
- **Определение таблиц**: Какие таблицы нужны для хранения данных о полисах и их истории.
- **Определение связей**: Как таблицы связаны между собой (например, полис принадлежит пользователю).
- **Определение индексов**: Какие индексы нужны для ускорения запросов
---
- **Таблица `users`**: Хранит данные о пользователях.
- **Таблица `policies`**: Хранит данные о страховых полисах.
- **Таблица `policy_history`**: Хранит историю изменений полисов.


```sql
CREATE TABLE policies (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    policy_type VARCHAR(50) NOT NULL,  -- Автострахование, здоровье и т. д.
    start_date TIMESTAMP NOT NULL,     -- Дата начала полиса
    end_date TIMESTAMP NOT NULL,       -- Дата окончания полиса
    status VARCHAR(20) NOT NULL CHECK (status IN ('active', 'expired', 'canceled')),
    created_at TIMESTAMP DEFAULT NOW()
);

```

```sql
CREATE TABLE policy_history (
    id SERIAL PRIMARY KEY,
    policy_id INT NOT NULL REFERENCES policies(id) ON DELETE CASCADE,
    action VARCHAR(50) NOT NULL CHECK (action IN ('created', 'renewed', 'canceled', 'expired', 'updated')),
    action_time TIMESTAMP DEFAULT NOW(),
    changed_by INT NULL,  -- ID оператора, если изменение ручное
    details TEXT NULL     -- Причина аннулирования, дополнительные комментарии
);
```