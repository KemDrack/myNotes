


**Merchant ID**
- Сначала в [Apple Developer Account](https://developer.apple.com/account/) необходимо **создать Merchant ID** – это специальный идентификатор продавца, который используется для идентификации вашего бизнеса в рамках экосистемы Apple Pay.
**Merchant Certificate**
- Чтобы устанавливать безопасный канал и Нужен чтобы удостоверять бэкенд как «законного мерчанта»
- В итоге руках будет `.cer`/`.pem`, а также приватный ключ.
- Этот сертификат важен для **взаимодействия** с Apple Pay, в частности для расшифровки токена
- Его нужно хранить в безопасном месте, он Подгружается при старте сервиса из [[Vault]]

- Но если использовать **провайдера** (например, **Stripe**, Braintree, Adyen и т. д.), то токен просто пробрасывается к ним, и уже провайдер хранит у себя нужные сертификаты.
---
Чтобы **подтвердить владение доменом**, нужно разместить специальный файл:
```url
https://<your-domain>/.well-known/apple-developer-merchantid-domain-association
```
Apple проверяет доступность этого файла для подтверждения, что мы действительно контролируете указанный домен.
Apple делает запрос к этому URL. Если всё корректно – домен будет ассоциирован с вашим Merchant ID