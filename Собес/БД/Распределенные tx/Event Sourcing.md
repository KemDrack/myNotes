https://habr.com/ru/companies/otus/articles/518282/

[[CQRS]] - Нужно сравнить!!!

Нужно будет хранить промежуточное состояние, а не всю историю

Шаблон проектирования архитектуры, который предполагает, что **система хранит историю событий вместо результирующего состояния**

Используют Kafka, можно согласованно изменять несколько БД - выполнять распределенную транзакцию 
