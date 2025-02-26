
### Что такое [[ООП]]? Основыне приципы
Парадигма программирования, основанная на объектах, которые инкапсулируют данные (состояние) и поведение (методы). Основные принципы: инкапсуляция, наследование, полиморфизм.
### Какие основные паттерны используются в ООП?

[[Паттерны в ООП]]

### Чем интерфейс отличается от класса?

Интерфейс определяет поведение (**методы**), но не имеет реализации. Класс может содержать как реализацию методов, так и данные

**Интерфейс**: Не может хранить данные (переменные или **состояние объекта**).**Класс**: Может содержать поля, которые хранят состояние объекта

В Go, интерфейс **не знает, кто его реализует**. Тип автоматически удовлетворяет интерфейсу, если он реализует его методы. **Класс** всегда явно указывает, от какого другого класса он наследуется.

- **Интерфейс** описывает **"что должно быть сделано"**
- **Класс** реализует **"как это делается"**

### Как реализовать наследование в Go?

Go не поддерживает прямое наследование. Используют композицию: встраивание структур и реализация интерфейсов

### Можно ли реализовать полиморфизм без интерфейсов?
Нет, в Go полиморфизм достигается через интерфейсы благодаря утиной типизации

### Что такое [[SOLID в GO]]?

Набор из 5 принципов для создания гибкого, масштабируемого и легко поддерживаемого кода:

- **Single Responsibility Principle** (Принцип единственной ответственности)
Это означает, что каждый модуль отвечает за свою задачу и не занимается ничем другим
- **Open/Closed Principle** (Принцип открытости/закрытости).
Нужно стараться добавлять новый функционал через расширение существующего кода, а не изменяя его
- **Liskov Substitution Principle** (Принцип подстановки Лисков).
- **Interface Segregation Principle** (Принцип разделения интерфейсов).
Лучше создавать много маленьких интерфейсов, чем один большой. Это позволяет клиентам использовать только те методы, которые им действительно нужны
- **Dependency Inversion Principle** (Принцип инверсии зависимостей).

### Можно ли применять все принципы SOLID в Go?

Полное соблюдение возможно, но иногда избыточно. Пример: принцип SRP часто нарушается в мелких утилитарных структурах


### Почему важно соблюдать SOLID?

Уменьшение связанности, увеличение модульности, улучшение читаемости и тестируемости кода

### Как реализована Инкапсуляция в GO?

Инкапсуляция у нас реализована через **Регистр.** Методы / переменные / объекты начинающиеся с маленькой буквы будут доступны только в видимости пакета, а те, которые начинаются с заглавной буквы будут доступны в глобальной видимости

### Как реализовано ООП в GO?

Сам по себе GO не является Объектно-Ориентированным языком, но у нас есть некоторые реализации ООП, например полиморфизм (через интерфейсы) и Композиция