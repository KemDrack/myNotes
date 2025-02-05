

✅ **Мгновенно получать статус барьеров** вместо запроса в PostgreSQL

Клиент отправляет `GET /barriers/:id/status`  
2️⃣ API делает запрос в PostgreSQL → Медленно  
3️⃣ Возвращает статус

📌 **С Redis**:  
1️⃣ Клиент отправляет `GET /barriers/:id/status`  
2️⃣ API проверяет Redis → Если данные есть → мгновенный ответ  
3️⃣ Если данных нет → Запрашивает PostgreSQL → Записывает в Redis`
```go
func getBarrierStatus(barrierID string) (string, error) {
    // Проверяем в Redis
    status, err := redisClient.Get(ctx, "barrier:"+barrierID).Result()
    if err == nil {
        return status, nil
    }
    
    if err != redis.Nil {
        return "", err // Ошибка Redis
    }

    // Если в Redis нет – запрашиваем из PostgreSQL
    row := db.QueryRow("SELECT status FROM barriers WHERE id = $1", barrierID)
    if err := row.Scan(&status); err != nil {
        return "", err
    }

    // Кэшируем в Redis
    cacheBarrierStatus(barrierID, status)
    return status, nil
}
```


**Как Redis обновлялся при изменении статуса?**
📌 **Kafka-консюмер, который слушает обновления статусов и обновляет Redis:**
```go
func listenBarrierUpdates() {
    sub := kafkaConsumer.SubscribeTopics([]string{"barrier_status_updates"}, nil)
    for {
        msg, err := sub.ReadMessage(-1)
        if err == nil {
            var event map[string]interface{}
            json.Unmarshal(msg.Value, &event)

            barrierID := event["barrier_id"].(string)
            status := event["status"].(string)

            updateBarrierStatus(barrierID, status) // Обновляем БД и Redis
        }
    }
}
```
