
```go
	"github.com/gorilla/websocket"

// Структура для хранения информации о статусе барьеров
type BarrierStatus struct {
	BarrierID string `json:"barrier_id"`
	Status    string `json:"status"`
}

// Хранилище WebSocket-клиентов
var clients = make(map[*websocket.Conn]string) // conn -> barrierID
var clientsMutex = sync.Mutex{}

var upgrader = websocket.Upgrader{
	CheckOrigin: func(r *http.Request) bool { return true },
}

// Обработчик WebSocket-соединений
func wsHandler(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("Ошибка WebSocket:", err)
		return
	}
	defer conn.Close()

	// Получаем ID барьера от клиента
	_, msg, err := conn.ReadMessage()
	if err != nil {
		log.Println("Ошибка чтения WebSocket:", err)
		return
	}
	barrierID := string(msg)

	// Добавляем клиента в список подписчиков
	clientsMutex.Lock()
	clients[conn] = barrierID
	clientsMutex.Unlock()

	log.Printf("Клиент подписался на обновления барьера: %s", barrierID)

	// Обрабатываем отключение клиента
	for {
		if _, _, err := conn.ReadMessage(); err != nil {
			clientsMutex.Lock()
			delete(clients, conn)
			clientsMutex.Unlock()
			log.Printf("Клиент отключился от барьера: %s", barrierID)
			break
		}
	}
}

// Функция для отправки обновлений клиентам
func notifyClients(barrierID string, status string) {
	clientsMutex.Lock()
	defer clientsMutex.Unlock()

	for conn, subscribedBarrier := range clients {
		if subscribedBarrier == barrierID {
			message := BarrierStatus{BarrierID: barrierID, Status: status}
			jsonMsg, _ := json.Marshal(message)

			err := conn.WriteMessage(websocket.TextMessage, jsonMsg)
			if err != nil {
				log.Println("Ошибка отправки сообщения:", err)
				conn.Close()
				delete(clients, conn)
			}
		}
	}
}

// Эмуляция изменения статуса барьеров (может быть заменено Kafka/Event-driven логикой)
func simulateBarrierUpdates() {
	barrierID := "B001"
	statuses := []string{"opened", "closed", "error"}

	for {
		for _, status := range statuses {
			log.Printf("Обновление барьера %s: %s", barrierID, status)
			notifyClients(barrierID, status)
			time.Sleep(5 * time.Second) // Имитация периодических обновлений
		}
	}
}

func main() {
	http.HandleFunc("/ws", wsHandler)

	// Запускаем имитацию изменений статусов барьеров
	go simulateBarrierUpdates()

	fmt.Println("WebSocket сервер запущен на порту 8080...")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```