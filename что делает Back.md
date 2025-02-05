
Endpoint **`POST /applepay/validate-merchant`**

Когда Frontend вызывает этот endpoint, он передаёт в теле `validationURL`.  
**Задача** бэкенда: сходить к `validationURL` с помощью **Merchant Identity Certificate** и получить `merchantSession`.

Нужно сформировать запрос к validationURL

**Важно** послать данные JSON. Apple ждёт поля: `merchantIdentifier`, `displayName`, `initiative`, `initiativeContext`.

// merchantSession вернётся как JSON
// Возвращаем на Frontend





Endpoint **`POST /applepay/process-payment`**
Здесь к вам приходит JSON с `token` (сам зашифрованный Base64). Задача: передать этот `token` провайдеру.
```go
type ProcessPaymentRequest struct {
    Token json.RawMessage `json:"token"`
}

func processPaymentHandler(w http.ResponseWriter, r *http.Request) {
    var req ProcessPaymentRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // Допустим, у нас есть функция sendToPSP(token)
    success, err := sendToPSP(req.Token)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    resp := map[string]bool{"success": success}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}

func sendToPSP(token json.RawMessage) (bool, error) {
    // Предположим, что ваш PSP принимает такой токен и ответом даёт статус
    // Например, делаем HTTP POST к https://api.mypsp.com/applepay
    // Здесь уже детали зависят от документации провайдера.

    // Если всё ок -> return true, nil
    // Если ошибка -> return false, err
    return true, nil
}
```

- **При валидации мерчанта**:
    - Принимает `validationURL` от Frontend,
    - **Запрашивает** `merchantSession` у Apple (с помощью Merchant Identity Certificate),
    - Возвращает `merchantSession` на Frontend.
- **При оплате**:
    - Принимает зашифрованный `token` (paymentData),
    - Передаёт этот токен в PSP,
    - Возвращает результат (успех/неудача