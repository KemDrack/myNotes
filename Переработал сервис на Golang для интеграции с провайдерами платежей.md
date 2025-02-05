`payment_token` – это **уникальный идентификатор платежа**, который выдается платежной системой после валидации платежных данных **на стороне клиента**(для безопасности)


Я создал **унифицированный API**, который работает с **разными платежными провайдерами**.

### **📌 Пример запроса на оплату**
```json
{
  "user_id": 12345,
  "amount": 1000,
  "currency": "USD",
  "provider": "stripe",
  "payment_token": "tok_1JHkZlE2..."
}
```
Обработчик платежей
```go
func handlePayment(w http.ResponseWriter, r *http.Request) {
    var req PaymentRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid request", http.StatusBadRequest)
        return
    }

    paymentManager := NewPaymentManager()
    response, err := paymentManager.ProcessPayment(req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(response)
}
```
Теперь **один API** поддерживает **разных провайдеров**, и нет необходимости писать отдельные эндпоинты.

**Реализация**
Общий интерфейс для всех платежных провайдеров
```go
type PaymentProvider interface {
    Charge(req PaymentRequest) (PaymentResponse, error)
}
```
**Менеджер платежей, который выбирает провайдера**
```go
type PaymentManager struct {
    providers map[string]PaymentProvider
}

func NewPaymentManager() *PaymentManager {
    return &PaymentManager{
        providers: map[string]PaymentProvider{
            "stripe":   &StripeProvider{},
            "paypal":   &PayPalProvider{},
            "applepay": &ApplePayProvider{},
        },
    }
}

func (pm *PaymentManager) ProcessPayment(req PaymentRequest) (PaymentResponse, error) {
    provider, exists := pm.providers[req.Provider]
    if !exists {
        return PaymentResponse{}, fmt.Errorf("unsupported provider: %s", req.Provider)
    }
    return provider.Charge(req)
}
```