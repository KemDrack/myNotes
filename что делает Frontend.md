
1. **Проверка доступности Apple Pay и создание ApplePaySession**
Когда пользователь нажимает эту кнопку, вы создаёте `ApplePaySession`, передавая:
- **paymentRequest** (страна, валюта, сумма, поддерживаемые сети карт).
```js
const paymentRequest = {
  countryCode: 'US',
  currencyCode: 'USD',
  total: {
    label: 'My Online Store',
    amount: '10.00'
  },
  supportedNetworks: ['visa', 'masterCard'],
  merchantCapabilities: ['supports3DS'],
};

const session = new ApplePaySession(3, paymentRequest);
```

2. Когда запускается сессия (`session.begin()`), Apple Pay вызывает колбэк:
```js
session.onvalidatemerchant = (event) => {
  const validationURL = event.validationURL;

  // Нужен запрос на бэкенд, чтобы бэкенд сходил к Apple
  fetch('/applepay/validate-merchant', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ validationURL }),
  })
    .then(response => response.json())
    .then((merchantSession) => {
      // Вызываем completeMerchantValidation
      session.completeMerchantValidation(merchantSession);
    })
    .catch((error) => {
      // Обработка ошибки
    });
};
```
**сам Frontend не делает запрос к `validationURL`**; это должен делать сервер, потому что при запросе к Apple необходим ваш **Merchant Identity Certificate**

3. Когда пользователь подтверждает платёж через Touch ID / Face ID, сессия вызывает:
```js
session.onpaymentauthorized = (event) => {
  const paymentData = event.payment.token.paymentData;
  // Это "зашифрованный токен", blob с информацией о карте

  // Отправляем paymentData на бэкенд
  fetch('/applepay/process-payment', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ token: paymentData }),
  })
    .then(res => res.json())
    .then(result => {
      if (result.success) {
        session.completePayment(ApplePaySession.STATUS_SUCCESS);
      } else {
        session.completePayment(ApplePaySession.STATUS_FAILURE);
      }
    })
    .catch(() => session.completePayment(ApplePaySession.STATUS_FAILURE));
};
```
После этого бэкенд уже взаимодействует с PSP (платёжным провайдером), передаёт им `token`, и возвращает результат (`success: true/false`).
