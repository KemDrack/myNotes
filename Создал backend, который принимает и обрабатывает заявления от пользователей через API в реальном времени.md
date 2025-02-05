
Для асинхронной обработки (проверка полиса, верификация документов, расчет выплат) используется Kafka. Например, при создании заявления отправляется событие в Kafka, которое обрабатывается отдельным сервисом.


📌 **Схема работы**  
1️⃣ **Клиент** отправляет заявление через API (`POST /claims`)  
2️⃣ **Сервис валидирует данные** (есть ли все нужные поля?)  
3️⃣ **Классифицирует страховой случай** (например, `автострахование`, `медицинская страховка`)  
4️⃣ **Сохраняет в базу данных** (MongoDB)
5️⃣ **Отправляет в Kafka для асинхронной обработки**  
6️⃣ **Отправляет ответ клиенту**


📌 Структура заявления
```json
{
  "user_id": 12345,
  "policy_id": 67890,
  "description": "ДТП на парковке, поврежден бампер",
  "attachments": ["file1.jpg", "file2.jpg"],
  "timestamp": "2024-01-30T12:45:00Z"
}
```

- Убедиться, что полис активен на момент страхового случая.
- Проверить, покрывает ли полис указанный инцидент.

Я использовал **MongoDB** для хранения заявлений, потому что:
- можно легко хранить вложенные данные (например, `attachments`)

 **Как ты классифицировал страховые случаи?**
Я использовал ключевые слова, чтобы автоматически определять категорию заявления
```go
func classifyClaim(description string) string {
    if strings.Contains(description, "ДТП") || strings.Contains(description, "авто") {
        return "auto_insurance"
    }
    if strings.Contains(description, "медицинская помощь") {
        return "health_insurance"
    }
    return "general"
}
```


**Я использовал Redis, чтобы хранить статусы заявлений и ускорить работу**


**Как ты уведомлял пользователей о статусе заявления?**
Я использовал **Kafka + [[Webhooks]] + Сервис уведомлений Email/SMS**, чтобы клиенты получали уведомления


Отправка события об обновлении статуса в Kafka
```go
func notifyClaimStatusChange(claimID string, newStatus string) {
    event := map[string]interface{}{
        "claim_id":  claimID,
        "status":    newStatus,
        "timestamp": time.Now().Unix(),
    }

    message, _ := json.Marshal(event)
    kafkaProducer.Produce(&kafka.Message{
        TopicPartition: kafka.TopicPartition{Topic: &"claim_notifications", Partition: kafka.PartitionAny},
        Value:          message,
    }, nil)
}
```
📌 **Подписка на обновления статусов в Kafka**
