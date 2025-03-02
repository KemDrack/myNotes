```
 При запросе данных о полисе я сначала проверяю, есть ли они в Redis. Если данные есть в кэше, я возвращаю их клиенту. Если данных нет, я извлекаю их из базы данных, сохраняю в Redis и затем возвращаю клиенту. Также я настроил инвалидацию кэша: при изменении данных о полисе я удаляю соответствующие данные из Redis, чтобы избежать устраревших данных
```
Перед внедрением Redis ты, скорее всего, участвовал в проектировании кэширования. Это включало:
- **Определение данных для кэширования**: Какие данные чаще всего запрашиваются и могут быть кэшированы (например, данные о полисах, пользователях, настройках).
- **Определение времени жизни кэша (TTL)**: Как долго данные будут храниться в кэше.
- **Определение стратегии инвалидации кэша**: Когда и как обновлять кэш при изменении данных.

Какие данные я кэшировал?
**Статусы полисов**
- Чтобы не делать частые запросы в базу при проверке статуса (`active`, `expired`, `canceled`). **10 минут**
**Коэффициенты страхования**
- (Например, по возрасту и стажу), их можно было кэшировать в **Redis Hash**. **без TTL** (изменяются редко)
```go
redisClient.HSet(ctx, "insurance_factors", "age:18-25", "1.5")
```
- `policy_history:*` (история изменений) → **30 минут**


- **Кэширование сложных запросов**: Можно кэшировать результаты сложных запросов, например, списки полисов с фильтрами.
- **Кластеризация Redis**: Для повышения отказоустойчивости и производительности можно настроить кластер Redis.
- **Мониторинг Redis**: Можно настроить мониторинг использования памяти и производительности Redis с помощью инструментов, таких как Prometheus или Grafana.