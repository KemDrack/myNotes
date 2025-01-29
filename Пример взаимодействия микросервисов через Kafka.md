
#### **1. Сервис заказов (Order Service)**
Сервис получает заказ и публикует сообщение в Kafka.
```go
package main

import (
	"context"
	"encoding/json"
	"log"
	"github.com/segmentio/kafka-go"
)

type Order struct {
	OrderID  string `json:"order_id"`
	Customer string `json:"customer"`
	Amount   float64 `json:"amount"`
	Status   string `json:"status"`
}

func main() {
	// Инициализация Kafka producer
	writer := kafka.Writer{
		Addr:     kafka.TCP("localhost:9092"),
		Topic:    "orders",
		Balancer: &kafka.LeastBytes{},
	}

	// Пример нового заказа
	order := Order{
		OrderID:  "12345",
		Customer: "John Doe",
		Amount:   100.50,
		Status:   "pending",
	}

	// Преобразование заказа в JSON
	orderData, err := json.Marshal(order)
	if err != nil {
		log.Fatal("Failed to marshal order:", err)
	}

	// Отправка сообщения в Kafka
	err = writer.WriteMessages(context.Background(),
		kafka.Message{Value: orderData},
	)
	if err != nil {
		log.Fatal("Failed to write message:", err)
	}

	log.Println("Order published:", order)
}
```

#### **Сервис оплаты (Payment Service)**
Сервис читает заказы из Kafka, обрабатывает оплату, обновляет статус заказа и публикует результат в другой topic.
```go
package main

import (
	"context"
	"encoding/json"
	"log"
	"math/rand"
	"time"
	"github.com/segmentio/kafka-go"
)

type Order struct {
	OrderID  string  `json:"order_id"`
	Customer string  `json:"customer"`
	Amount   float64 `json:"amount"`
	Status   string  `json:"status"`
}

func main() {
	// Инициализация Kafka consumer
	reader := kafka.NewReader(kafka.ReaderConfig{
		Brokers: []string{"localhost:9092"},
		Topic:   "orders",
		GroupID: "payment_service",
	})

	writer := kafka.Writer{
		Addr:     kafka.TCP("localhost:9092"),
		Topic:    "order_status",
		Balancer: &kafka.LeastBytes{},
	}

	for {
		// Чтение сообщения из Kafka
		msg, err := reader.ReadMessage(context.Background())
		if err != nil {
			log.Fatal("Failed to read message:", err)
		}

		// Десериализация данных
		var order Order
		if err := json.Unmarshal(msg.Value, &order); err != nil {
			log.Println("Invalid order data:", err)
			continue
		}

		// Обработка оплаты (симуляция)
		order.Status = "paid"
		time.Sleep(time.Duration(rand.Intn(3)) * time.Second) // Симуляция обработки

		// Публикация обновлённого заказа в Kafka
		updatedOrder, _ := json.Marshal(order)
		err = writer.WriteMessages(context.Background(),
			kafka.Message{Value: updatedOrder},
		)
		if err != nil {
			log.Fatal("Failed to write message:", err)
		}

		log.Println("Order processed and updated:", order)
	}
}

```

#### **Сервис уведомлений (Notification Service)**
Сервис получает данные об оплаченных заказах и отправляет уведомление клиенту.
```go
package main

import (
	"context"
	"encoding/json"
	"log"
	"github.com/segmentio/kafka-go"
)

type Order struct {
	OrderID  string  `json:"order_id"`
	Customer string  `json:"customer"`
	Amount   float64 `json:"amount"`
	Status   string  `json:"status"`
}

func main() {
	// Инициализация Kafka consumer
	reader := kafka.NewReader(kafka.ReaderConfig{
		Brokers: []string{"localhost:9092"},
		Topic:   "order_status",
		GroupID: "notification_service",
	})

	for {
		// Чтение сообщения из Kafka
		msg, err := reader.ReadMessage(context.Background())
		if err != nil {
			log.Fatal("Failed to read message:", err)
		}

		// Десериализация данных
		var order Order
		if err := json.Unmarshal(msg.Value, &order); err != nil {
			log.Println("Invalid order data:", err)
			continue
		}

		// Отправка уведомления клиенту
		log.Printf("Notification sent to %s: Your order #%s is now %s\n",
			order.Customer, order.OrderID, order.Status)
	}
}

```
Этот пример демонстрирует, как микросервисы могут обмениваться данными через Kafka для выполнения взаимосвязанных задач.


`Topic` — это логическая структура в Kafka, которая используется для хранения сообщений. Вы можете рассматривать `Topic` как категорию сообщений, которые отправляются продюсерами (producers) и читаются консюмерами (consumers).
`GroupID` используется для объединения консюмеров в группу (consumer group). Консюмеры одной группы совместно обрабатывают сообщения из topic.