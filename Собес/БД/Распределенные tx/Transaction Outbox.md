Event Sourcing и Transaction Outbox 
**Когда мы сохраняем данные и асинхронно отправляем** 

Проблема в том, что запись в таблицу и отправка в Kafka происходит не в одной транзакции. У бд свой транзакционный менеджер у кафки свой и у нас будут работать 2 разные транзакции

Проблема может быть в том, что транзакция в бд закомитилась, а транзакцию в кафку мы не смогли закомитить (сервис упал), тут происходит несогласованность данных

**Вместо отправки события напрямую**, мы:
1. Сохраняем событие в таблицу `outbox` в той же транзакции, что и запись в обычную таблицу
2. `Job` читает `outbox` и отправляет данные в Kafka
3. Если отправка успешна — удаляем запись из `outbox`

Структура таблицы outbox: **id**, **payload**(текстовая строка, json объекта, который нам нужно переслать в кафку), **created_at**, **type**(нужно предусмотреть тип отправляемых данных)

Есть основная таблица и специальная **outbox**, куда мы складываем допустим Event, что заказ был создан

В коде приложения поднят какой-то воркер(горутина), который периодично ходит в **outbox** таблицу сканирует и отправляет в кафку. При отправке он **удаляет** эту запись


**Недостатки**:
- **Кастомное решение** (мы как разработчики несем ответственность, его нужно тестировать, отлаживать, адаптировать под требования)
- Долгая транзакция
- Несколько **копий** сервиса (У каждой копии будет своя Joba и они все одновременно будут обращаться к outbox, и если не предусмотреть механизм Лока, тогда может быть такая ситуация, несколько Job считали одни и те же данные и отправили в KAFKA, а блокировки - усложнение нашего решения, Job будут блочить друг друга, рискуем попасть в ситуацию DeadLock-ов)
- **Нагрузка на бд** даже в случае пустой таблицы outBox(Job все равно будут обращаться к таблице, нагрузка бд бесполезной работой)
- **Возможно дублирование** (Job отравила сообщение в кафку, а после не смогла закомитеть свою транзакцию в бд и запись не удалилась из outBox, после рестарта будет повторно обрабатывать ту запись).Нужно предусмотреть **идемпотентность** на стороне консьюмера


