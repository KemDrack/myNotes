**Продьюсеры**
Продьюсеры отправляют сообщения в очередь

**Консьюмеры**
Консьюмеры извлекают сообщения из очереди и обрабатывают их

**Оффсеты**: Можем понять какие сообщения уже были обработаны

GroupID используется для объединения консюмеров в группу (consumer group). Консюмеры одной группы совместно обрабатывают сообщения из topic

**Dead Letter Queue (DLQ)**
DLQ используется для сообщений, которые не удалось обработать после нескольких попыток. Если бы DLQ не было, тогда основаная очередь была бы заблокированна 
- Если сообщение не обработано успешно, оно отправляется в **отдельный топик** (например, `...-topic-dlq`)

**Брокер** — это сервер, который принимает, хранит и отправляет сообщения

**Топик** — это структура для группировки сообщений. Каждый топик может быть разделён на несколько партиций

**Партиции** — это физические подразделения топика, которые помогают распараллелить нагрузку

*Гарантии доставки*<u></u>
auto commit(**at most once**): Консьюмер получает данные и сразу происходит коммит, Kafka считает, что для этой группы консьюмеров сообщения уже обработаны, хотя мы можем в этот момент упасть и потерять сообщения; **сообщение будет доставлено не более одного раза**
manual commit(**at least once**): Мы сначала обрабатываем данные, а уже потом делаем commit; **сообщение будет доставлено хотя бы один раз**
**Exactly once:** Сообщение доставляется ровно один раз

*Гарантии доставки*<u></u>
**auto commit**(at most once), он не всегда хорош. Потому что когда мы получаем батч данных от брокера, то сразу происходит auto commit и Kafka считает, что для этой группы Консьюмеров эти сообщения уже обработаны, хотя Консьюмер их только получил и ему надо их обработать и если он в этот момент упал, то по сути он получил данные, но ничего с ними не сделал. Получается он потеряет сообщения, потому что при повторном подключении он пойдет уже со следующего offset

**manual commit**(at least once), когда Kafka нам предоставляет данные, сразу auto commit не происходит, мы обрабатываем данные и после того как мы обработали мы уже отправляем **commit** этого **offset**
Возможны дубли, **пример**: мы взяли 3 сообщения, первые 2 обработали и упали, commit не было, но при этом первые 2 мы обработали, тогда после перезапуска мы снова прочитаем эти 3 сообщения и нам нужно разбираться, мы первые 2 уже обработали или нет