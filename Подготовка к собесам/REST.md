#собеседование

Это архитектурный стиль для проектирования распределённых систем(сложных интернет сервисов), таких как веб-приложения.

Отличительной особенностью сервисов REST является то, что они позволяют наилучшим образом использовать протокол HTTP

 Основные принципы REST:

1. **Клиент-серверная архитектура:**
    - Клиент и сервер работают независимо. Клиент запрашивает данные, сервер обрабатывает запрос и возвращает ответ.
2. **Stateless** (Безсостояние):
    - Сервер не хранит состояние клиента между запросами. Каждый запрос содержит всю необходимую информацию для обработки.
3. **Кэширование:**
    - Данные можно кэшировать, чтобы уменьшить количество запросов к серверу и повысить производительность.
4. **Единый интерфейс (Uniform Interface):**
    - Определённый набор правил взаимодействия:
        - **Идентификаторы ресурсов:** Каждый ресурс имеет уникальный URL (например, `/users/1`).
        - **Стандартизованные методы HTTP:** Используются методы GET, POST, PUT, DELETE и др.
        - **Самоописываемые сообщения:** Ответы и запросы содержат всю информацию (заголовки, статус, тело).
        - **Гипермедиа как движущая сила приложения (HATEOAS):** Ответы могут содержать ссылки на связанные ресурсы.
5. **Ресурсы:**
    - Все данные (пользователи, заказы, продукты) считаются ресурсами. Каждый ресурс имеет представление, например, в формате JSON или XML.


[[API]] — ==Интерфейс для взаимодействия двух или более программ между собой==

Пример: После срабатывания будильника клиент хочет получить прогноз погоды. Мы будем обращаться к другому сервису, чтобы получить информацию и отдать клиенту







]]![[REST.pdf]]