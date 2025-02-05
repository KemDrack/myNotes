- **`POST /notifications`**: Создание уведомления (вызывается другими сервисами).
- **`GET /notifications`**: Получение списка уведомлений для пользователя (используется фронтендом).
- **`POST /notifications/subscribe`**: Подписка на push-уведомления (мобильное приложение).

Kafka отправляет события при обновлении статуса заявления
Консюмер Kafka обрабатывает уведомления и отправляет их пользователям




**Процесс работы:**  
**1️⃣** **Пользователь создает заявление (`POST /claims`)**  
**2️⃣ Когда статус заявления меняется, система отправляет уведомление**  
**3️⃣ REST API уведомляет фронтенд и мобильное приложение**  
**4️⃣ Уведомления передаются через WebSockets, Firebase (FCM) и Email/SMS**

📌 **Схема работы:**
`MongoDB → Kafka → REST API → WebSockets / Firebase / Email / SMS → Пользователь`


📌 **Настроил WebSockets для пуш-уведомлений в браузере**
```GO
var clients = make(map[*websocket.Conn]bool)
var broadcast = make(chan NotificationMessage)

func handleConnections(w http.ResponseWriter, r *http.Request) {
    ws, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println(err)
        return
    }
    defer ws.Close()

    clients[ws] = true
    for msg := range broadcast {
        for client := range clients {
            err := client.WriteJSON(msg)
            if err != nil {
                log.Printf("WebSocket error: %v", err)
                client.Close()
                delete(clients, client)
            }
        }
    }
}
```


📌 **Функция отправки пуш-уведомления через Firebase**
```GO
func sendPushNotification(userID int, message string) error {
    fcmToken, err := getUserFCMToken(userID)
    if err != nil {
        return err
    }

    payload := map[string]interface{}{
        "to": fcmToken,
        "notification": map[string]string{
            "title": "Обновление заявления",
            "body":  message,
        },
    }

    body, _ := json.Marshal(payload)
    req, _ := http.NewRequest("POST", "https://fcm.googleapis.com/fcm/send", bytes.NewBuffer(body))
    req.Header.Set("Authorization", "Bearer YOUR_FCM_SERVER_KEY")
    req.Header.Set("Content-Type", "application/json")

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    return nil
}
```
✅ Теперь **мобильное приложение получает пуш-уведомления через Firebase**.


📌 **Функция отправки Email через SendGrid**
```GO
func sendEmailNotification(email string, message string) error {
    payload := map[string]interface{}{
        "to":      email,
        "subject": "Обновление заявления",
        "body":    message,
    }

    body, _ := json.Marshal(payload)
    req, _ := http.NewRequest("POST", "https://api.sendgrid.com/v3/mail/send", bytes.NewBuffer(body))
    req.Header.Set("Authorization", "Bearer YOUR_SENDGRID_API_KEY")
    req.Header.Set("Content-Type", "application/json")

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    return nil
}
```

📌 **Функция отправки SMS через Twilio**
```GO
func sendSMSNotification(phone string, message string) error {
    payload := url.Values{}
    payload.Set("To", phone)
    payload.Set("From", "YOUR_TWILIO_PHONE")
    payload.Set("Body", message)

    req, _ := http.NewRequest("POST", "https://api.twilio.com/2010-04-01/Accounts/YOUR_TWILIO_SID/Messages.json",
        strings.NewReader(payload.Encode()))
    req.SetBasicAuth("YOUR_TWILIO_SID", "YOUR_TWILIO_AUTH_TOKEN")
    req.Header.Set("Content-Type", "application/x-www-form-urlencoded")

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    return nil
}
```

"Я разработал систему автоматических уведомлений, интегрированную с фронтендом и мобильным приложением. При изменении статуса заявления микросервисы отправляют событие в REST API, которое сохраняет уведомление в MongoDB и отправляет задачу в Kafka. Асинхронно обрабатывая задачи, система рассылает уведомления через email, SMS и push-сообщения (через Firebase). Для фронтенда я реализовал WebSocket, чтобы пользователи видели уведомления в реальном времени. Мобильное приложение подключается к FCM, а токены устройств сохраняются через отдельный эндпоинт. При ошибках отправки система делает до 3 повторных попыток с экспоненциальной задержкой.