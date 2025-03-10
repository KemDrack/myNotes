HTTP - протокол передачи данных прикладного уровня
HTTPS (HTTP Secure) — это надстройка над протоколом HTTP, которая поддерживает шифрование посредством криптографического протокола TLS. Шифрует отправляемые данные на клиенте и дешифруют их на сервере

[[Версии HTTP]]
## Структура http запроса, ответа

1.Стартовая строка (Request Line)
**Метод**(`GET`, `POST`, `PUT`, `DELETE`)
**URI (Uniform Resource Identifier)**: Путь к ресурсу
**Версия HTTP**
```
GET /api/data HTTP/1.1
```
2.Заголовки (Headers):
**Содержат метаданные для сервера**(например, тип контента, авторизация)
Каждый заголовок пишется в формате `Ключ: Значение`
```
Host: example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Authorization: Bearer token123
Access-control-allow-origin: *
```
3.Тело запроса (Body):
- Используется в методах `POST`, `PUT` и др. для передачи данных.
- Формат зависит от **заголовка** `Content-Type` (например, JSON, XML, форма)
---
## Пример полного HTTP-запроса

```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 45

{"username": "user", "password": "pass"}
```

**Структура HTTP-ответа**
1.Строка статуса (Status Line):
- **Версия HTTP**: Например, `HTTP/1.1`.
- **Код статуса**: Числовой код результата (например, `200`, `404`).
- **Пояснение**: Текстовое описание кода (например, `OK`, `Not Found`).
2.Заголовки (Headers):
- **Метаданные для клиента**
3.Тело ответа (Body):
- Данные, которые сервер отправляет клиенту (например, HTML, JSON, изображение).
```
HTTP/1.1 404 Not Found
Content-Type: text/plain
Content-Length: 25

Error: Page not found.
```

### Важные заголовки
- `Content-Type` — тип данных в теле (например, `text/html`, `application/json`).
- `Content-Length` — размер тела в байтах.
- `Authorization` — токены аутентификации.
- `Cache-Control` — управление кэшированием.
- `User-Agent` — информация о клиенте (браузер, ОС).


**Query String**
Данные на сервер можно передавать через тело запроса и через так называемую строку запроса **Query String**
```
GET /files?key=value&key2=value2 HTTP/1.0
```
Обычно параметры Query String используются в GET-запросах, чтобы конкретизировать получаемый ресурс. Например, можно получить на сервере список файлов, имена которых будут начинаться с переданного значения.