

**Graceful Shutdown** — это **корректное завершение работы приложения**, при котором:
- Завершаются все **активные запросы**.
- Корректно закрываются **соединения с БД, Kafka, Redis и т. д.**.
- **Не принимаем новые запросы**
- **Остановить Kafka** (продюсеры зафлэшить, консьюмеры зафиксировать offset).
- **Дождаться завершения уже запущенных запросов**.
Всё это должно происходить **за ограниченное время** (например, 10–15 секунд), чтобы сервис не завис.


- Graceful Shutdown в Go реализуется через `os/signal` + `context.WithCancel()`.
- Для HTTP-сервера → используем `srv.Shutdown(ctx)`, чтобы завершить запросы перед остановкой.
- Для БД и кэша → вызываем `db.Close()` / `redisClient.Close()`, чтобы не оставлять соединения.
- Для Kafka / RabbitMQ → останавливаем потребителя и фиксируем offset'ы перед завершением.
- Если нужно задать таймаут на завершение → используем `context.WithTimeout()`.


**Закрываем HTTP-сервер корректно**
```go
    // Ловим SIGTERM/SIGINT для Graceful Shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    fmt.Println("Выключаем сервер...")
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    if err := srv.Shutdown(ctx); err != nil {
        fmt.Printf("Ошибка завершения сервера: %v\n", err)
    }
    fmt.Println("Сервер корректно завершил работу.")
}
```
Ловим сигнал `SIGTERM` или `SIGINT`
Вызываем `srv.Shutdown(ctx)`, передавая **контекст с таймаутом** (`5s`), чтобы запросы завершились.

**Закрытие соединений с БД**
```go
    ctx, cancel := context.WithCancel(context.Background())
    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan
        fmt.Println("Завершаем работу...")
        cancel()
        // Закрываем соединение с БД
        fmt.Println("Закрываем соединение с БД...")
        db.Close()
    }()
    <-ctx.Done()
    fmt.Println("Приложение завершено корректно.")
}
```

**Kafka**
**Остановить консьюмера** перед завершением работы
Закрыть соединение с брокером
Если просто убить процесс → сообщения могут потеряться, потому что не будут зафиксированы offset'ы. добавляют **`consumer.Commit()`**

- **Перестаём** забирать новые сообщения (`pause` или останавливаем цикл чтения).
- Делаем **flush** (дожидаемся отправки всех сообщений), чтобы не потерять их в очереди продюсера.
- Закрываем соединение (`producer.Close()`).
- Закрываем соединение (`consumer.Close()`).
```go
func (k *KafkaService) Close() {
    // Останавливаем Consumer
    fmt.Println("Закрываем Kafka Consumer...")
    if err := k.Consumer.Close(); err != nil {
        fmt.Println("Ошибка при закрытии Consumer:", err)
    }
    // Флашим Producer (ждём, пока все сообщения отправятся)
    fmt.Println("Флашим Kafka Producer...")
    k.Producer.Flush(5000) // 5 секунд
    k.Producer.Close()
}
```

