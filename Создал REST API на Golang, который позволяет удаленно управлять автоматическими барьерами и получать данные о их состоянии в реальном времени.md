
|Метод|URL|Описание|
|---|---|---|
|`POST`|`/barriers/:id/open`|Открыть барьер|
|`POST`|`/barriers/:id/close`|Закрыть барьер|
|`GET`|`/barriers/:id/status`|Получить статус барьера|
|`GET`|`/barriers/events`|Получить историю событий|

Я реализовал **REST API**, который позволяет:
✅ **Удаленно управлять барьерами** – открытие, закрытие, получение статуса
✅ **Обрабатывать команды через Kafka** – асинхронная передача команд
✅ **Хранить логи событий в PostgreSQL** – хранение истории работы барьеров
✅ **Кэшировать статусы в Redis** – кэширование текущего состояния барьеров
✅ **Использовать WebSockets** – обновление статуса в реальном времени


Что происходило, когда клиент отправлял команду на открытие барьера?
**1️⃣ Клиент отправляет запрос `POST /barriers/:id/open`**  
**2️⃣ API записывает команду в Kafka → барьер получает задачу**  
**3️⃣ Система обновляет статус в Redis → клиенту сразу приходит ответ**  
**4️⃣ Когда барьер открывается, он отправляет подтверждение → статус обновляется в PostgreSQL и Redis**  
**5️⃣ Клиент получает обновленный статус через WebSocket или `GET /barriers/:id/status`**


```go
func openBarrier(c echo.Context) error {
    barrierID := c.Param("id")

    // Отправляем команду в Kafka
    err := sendBarrierCommandToKafka(barrierID, "open")
    if err != nil {
        return c.JSON(http.StatusInternalServerError, echo.Map{"error": "Ошибка отправки команды"})
    }

    return c.JSON(echo.Map{
        "barrier_id": barrierID,
        "status":     "opening",
        "message":    "Команда отправлена",
    })
}
```
✅ **Теперь API принимает команду и отправляет её в Kafka.**


Как ты интегрировал Kafka для асинхронного управления барьерами?
**Функция** отправки команды в Kafka:
```go
func sendBarrierCommandToKafka(barrierID, command string) error {
    event := map[string]interface{}{
        "barrier_id": barrierID,
        "command":    command,
        "timestamp":  time.Now().Unix(),
    }
    msg, _ := json.Marshal(event)

    return kafkaProducer.Produce(&kafka.Message{
        TopicPartition: kafka.TopicPartition{Topic: &"barrier_commands", Partition: kafka.PartitionAny},
        Value:          msg,
    }, nil)
}
```


📌 **Консюмер Kafka, который слушает команды и управляет барьерами:**
```go
func listenBarrierCommands() {
    consumer.SubscribeTopics([]string{"barrier_commands"}, nil)
    for {
        msg, err := consumer.ReadMessage(-1)
        if err == nil {
            var command map[string]interface{}
            json.Unmarshal(msg.Value, &command)

            barrierID := command["barrier_id"].(string)
            action := command["command"].(string)

            processBarrierCommand(barrierID, action)
        }
    }
}
```
✅ **Теперь команды передаются асинхронно, а барьеры управляются без задержек.**



В системе было две базы данных:  
**1️⃣ Основная база** (PostgreSQL) – хранит актуальный статус барьеров  
**2️⃣ База истории** (PostgreSQL, отдельная таблица) – хранит все события, связанные с барьерами

Когда барьер **открывался или закрывался**, его **актуальный статус обновлялся в основной таблице**.
```sql
CREATE TABLE barriers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    status TEXT CHECK (status IN ('opened', 'closed', 'error')),
    last_updated TIMESTAMP DEFAULT NOW()
);
```
