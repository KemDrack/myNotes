

SELECT ... FOR UPDATE
Блокирует строку, чтобы другие транзакции не изменили её до `COMMIT`
```sql
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```
Это предотвращает **condition race**, когда два запроса читают старое значение и изменяют его некорректно.


`LOCK TABLE` для блокировки изменений
Позволяет **защитить данные от конкурентного доступа**.
