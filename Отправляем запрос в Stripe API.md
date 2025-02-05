В **Stripe API** `PaymentMethod` и `PaymentIntent` - это два ключевых объекта


Stripe принимает `paymentData` и создаёт **платёжный метод** (`PaymentMethod`)
Это **не сам платёж**, а только **информация о том, чем платить**

Stripe нужны данные о способе оплаты клиента — либо номер карты, либо учетные данные для какой-либо другой платежной системы

Когда мы создаём `PaymentMethod`, мы указываем **тип оплаты**

- **URL:** `https://api.stripe.com/v1/payment_methods`
- **Тело запроса:** передаём `paymentData`
- **Заголовки:**
    - `Authorization: Bearer {YOUR_SECRET_KEY}`
    - `Content-Type: application/x-www-form-urlencoded`

**Ответ Stripe содержит `payment_method_id`**, который используется для привязки к `PaymentIntent`
```json
  "id": "pm_1N2QfK2eZvKYlo2C0mP9JkT3",
  "object": "payment_method",
  "type": "card",
  "card": {
    "brand": "visa",
    "last4": "4242"
  }
```




`PaymentIntent` – это сам платёж

**Создание `PaymentIntent` для проведения оплаты**
После того как мы создали платёжный метод (`payment_method_id`), нужно создать **PaymentIntent**
- **URL:** `https://api.stripe.com/v1/payment_intents`
- **Тип:** `POST`
- **Тело запроса:**
    - `amount=1000` (сумма в **центах**, т.е. $10.00)
    - `currency=usd`
    - `payment_method={payment_method_id}`
    - `confirm=true` моментальная оплата
В ответе Stripe пришлёт **`payment_intent_id`**, по которому можно отслеживать статус платежа.
```json
{
  "id": "pi_1N2QfK2eZvKYlo2CTnJnJX3L",
  "object": "payment_intent",
  "status": "requires_action",
}
```
**Проверяем статус `PaymentIntent` (`GET /v1/payment_intents/{id}`)**.
Отдаём ответ клиенту (успех / ошибка).



### ✅ **Пример привязки платежа к пользователю**
Можно передавать `metadata`, чтобы отслеживать, кто платит.
Stripe вернёт:
```json
{
  "id": "pi_1N2QfK2eZvKYlo2CTnJnJX3L",
  "status": "succeeded",
  "metadata": {
    "user_id": "12345",
    "order_id": "67890"
  }
}
```