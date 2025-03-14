
### **Как я использовал PostgreSQL в работе**

Анализировал производительность запросов с помощью:
- **`EXPLAIN` и `EXPLAIN ANALYZE`** для диагностики медленных запросов.
- Оптимизировал запросы, добавляя индексы

Для создания таблиц
Для записи данных в таблицу
Агрегатные функции
Разделил запросы: `SELECT` выполнялись на реплике, а `INSERT` и `UPDATE` — на мастере.


### Опыт работы. “Твоя зона ответственности” на последнем месте работы

Разрабатывал сервисы для автоматического страхования(использовал Redis для хранения коэффициентов), классифицировал страховые случаи(сначала с помощью ключевых слов, после решил использовал еластик)
Работал с **Kafka (продьюсеры/консьюмеры)**, Настраивал **Dead Letter Queue (DLQ)** для сообщений, которые не удалось обработать после нескольких попыток.
(**3️⃣** DLQ-консьюмер (читает `orders-dlq` и логирует проблемные сообщения) если обработка **не удалась после 3 попыток**, он отправляет сообщение в `orders-dlq`
```go
default.replication.factor=3
min.insync.replicas=2

Переменные окружения в сервисах
KAFKA_DLQ_TOPIC=orders-dlq
KAFKA_MAX_RETRIES=3
```
Работал с Базой данных, добавлял индексы, оптимизировал запросы
Использовал **pprof** для поиска узких мест и профилирования кода.

### У тебя в резюме указан опыт с Kafka, как использовал?

Реализовывал **продьюсеров** на Go, которые отправляли данные в топики Kafka
Разрабатывал **консьюмеров** для обработки сообщений
Настраивал **консьюмерные группы**, чтобы **распределить нагрузку**
Настраивал **DLQ-топики**, куда отправлялись **проблемные сообщения**.
- Настраивал **репликацию Kafka**
- Использовал **acks=all**, чтобы **гарантировать запись во все ISR**. - Сообщение записывается в **лидера** и **во все In-Sync Replicas (ISR)**.

### Как мы можем отличить, если значение по умолчанию (напр пустая строка) и пустая строка которую мы туда записали, когда мы получаем из канала.

В Go **каналы** не хранят информацию о том, было ли значение в канале записано явно или это значение по умолчанию.

Вместо передачи простых типов (например, `string`) можно передавать структуру, которая содержит как значение, так и флаг, указывающий, было ли значение установлено.
```go
type Message struct {  
    Value string  
    Valid bool // Флаг, указывающий, было ли значение установлено  
}  
  
func main() {  
    ch := make(chan Message, 1)  
  
    // Отправляем пустую строку с флагом Valid = true  
    ch <- Message{Value: "", Valid: true}  
  
    // Получаем значение из канала  
  
    select {  
    case msg := <-ch:  
       fmt.Printf("Message sent %v\n  Message ib basic %v", msg.Value, msg.Valid)  
    case <-time.After(1 * time.Second):  
       fmt.Println("Timed out")  
    }
```


Можно использовать закрытие канала как сигнал о завершении
```go
func main() {
	ch := make(chan string, 1)
	// Отправляем пустую строку
	ch <- ""
	// Закрываем канал
	close(ch)
	// Получаем значение из канала
	value, ok := <-ch
	if ok {
		fmt.Println("Получено значение:", value)
	} else {
		fmt.Println("Канал закрыт, значение не было установлено")
	}
}
```






Вместо передачи значения можно передавать указатель на значение. Если указатель равен `nil`, это означает, что значение не было установлено
```go
func main() {
	ch := make(chan *string, 1)

	// Отправляем пустую строку
	emptyStr := ""
	ch <- &emptyStr

	// Получаем значение из канала
	msg := <-ch

	if msg != nil {
		fmt.Println("Получено значение:", *msg)
	} else {
		fmt.Println("Значение не было установлено")
	}
}
```



### Почему статику нужно размещать на отдельном домене.

Если статику раздаёт **тот же сервер, что и backend**, он обрабатывает **как динамические запросы, так и статику**, что создаёт **излишнюю нагрузку**

Улучшение кеширования
- Браузеры **лучше кэшируют** статику, если она раздаётся **с отдельного домена**.
- Можно использовать **долгосрочное кеширование** (`Cache-Control: max-age=31536000`), так как файлы не изменяются динамически.


**Браузер сохраняет cookie** и отправляет его со всеми последующими запросами:
Для статики не нужны куки, **cookie-заголовка нет**, что снижает **размер запроса** и ускоряет загрузку

**[[Cookies]]** - текстовые файлы, которые браузер сохраняет и автоматически отправляет серверу при каждом запросе. Они используются для идентификации пользователя, хранения сессий и персонализации

**Идентификация пользователя** (аутентификация)
- Сервер запоминает пользователя после логина.
- Пример: `session_id=abc123`.
**Хранение настроек и персонализации**
- Язык интерфейса, цветовая тема.
- Пример: `theme=dark; lang=ru`.
**Корзина товаров в интернет-магазине**
- Если пользователь **не авторизован**, корзина хранится в cookie.
- Пример: cart=[{"id":1, "count":2}, {"id":5, "count":1}.

