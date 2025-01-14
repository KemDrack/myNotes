

 *Эти знания нужны просто для флекса и прохождения собеседований*


## Принципы SOLID

**SOLID** — это набор принципов ООП, которые помогают сделать код более читабельным, поддерживаемым и расширяемым. В языке Go, несмотря на то, что он не является классическим объектно-ориентированным языком, эти принципы всё ещё актуальны.

### S: Single Responsibility Principle (Принцип единственной ответственности)

Это означает, что каждый модуль отвечает за свою задачу и не занимается ничем другим. 

В Go принцип единственной ответственности легко достигается разделением функций на мелкие и четко определённые.

```go
package main

import "fmt"

// Нарушение принципа: функция занимается и логикой приветствия, и выводом результата
func GreetAndPrint(name string) {
    greeting := fmt.Sprintf("Привет, %s!", name)
    fmt.Println(greeting)
}

// Исправление: разделим ответственность на две функции
func Greet(name string) string {
    return fmt.Sprintf("Привет, %s!", name)
}

func PrintGreeting(greeting string) {
    fmt.Println(greeting)
}

func main() {
    greeting := Greet("Алиса")
    PrintGreeting(greeting)
}
```


### O: Open/Closed Principle (Принцип открытости/закрытости)

Принцип открытости/закрытости подразумевает, что сущности должны быть открыты для расширения, но закрыты для модификации. Другими словами, нужно стараться добавлять новую функциональность через расширение существующего кода, а не изменяя его.

Для этого часто используется интерфейсное программирование:
```go
package main

import "fmt"

// Определим интерфейс, который можно расширять
type Notifier interface {
    Notify() string
}

// EmailNotifier реализует интерфейс Notifier
type EmailNotifier struct {
    email string
}

func (e EmailNotifier) Notify() string {
    return fmt.Sprintf("Отправка уведомления на email: %s", e.email)
}

// Можно добавить новый способ уведомления, не изменяя существующий код
type SMSNotifier struct {
    phoneNumber string
}

func (s SMSNotifier) Notify() string {
    return fmt.Sprintf("Отправка SMS на номер: %s", s.phoneNumber)
}

func main() {
    notifiers := []Notifier{
        EmailNotifier{email: "example@example.com"},
        SMSNotifier{phoneNumber: "+123456789"},
    }

    for _, notifier := range notifiers {
        fmt.Println(notifier.Notify())
    }
}
```
Здесь мы можем добавлять новые реализации Notifier (например, PushNotifier), не изменяя код существующих структур и функций, что соответствует принципу открытости/закрытости.


### L: Liskov Substitution Principle (Принцип подстановки Барбары Лисков)

Принцип Лисков говорит, что объекты должны быть заменяемы их подтипами без нарушения работы программы. В Go это часто относится к тому, что структуры, реализующие интерфейс, должны корректно вести себя как подтип интерфейса.
```go
package main

import "fmt"

// Определим интерфейс
type Vehicle interface {
    Drive() string
}

// Реализуем интерфейс в структуре Car
type Car struct {}

func (c Car) Drive() string {
    return "Машина едет"
}

// Реализуем интерфейс в структуре Bike
type Bike struct {}

func (b Bike) Drive() string {
    return "Велосипед едет"
}

func main() {
    vehicles := []Vehicle{Car{}, Bike{}}
    for _, vehicle := range vehicles {
        fmt.Println(vehicle.Drive())
    }
}
```

В этом примере структуры Car и Bike реализуют интерфейс Vehicle. Мы можем использовать объекты этих структур как подтипы Vehicle, не нарушая работу программы.


### I: Interface Segregation Principle (Принцип разделения интерфейса)

Принцип разделения интерфейсов гласит, что лучше создавать много маленьких интерфейсов, чем один большой. Это позволяет клиентам использовать только те методы, которые им действительно нужны.

```GO
package main

import "fmt"

// Плохая практика: один большой интерфейс
type Worker interface {
    Work()
    Report()
    AttendMeeting()
}

// Хорошая практика: разделённые интерфейсы
type WorkerDoer interface {
    Work()
}

type Reporter interface {
    Report()
}

type Attendee interface {
    AttendMeeting()
}

// Структура Engineer реализует только нужные интерфейсы
type Engineer struct {}

func (e Engineer) Work() {
    fmt.Println("Инженер работает")
}

func (e Engineer) Report() {
    fmt.Println("Инженер отчитывается о работе")
}

func main() {
    var worker WorkerDoer = Engineer{}
    worker.Work()
}
```

Здесь интерфейсы разделены, и структура Engineer может реализовывать только нужные методы, не обязываясь поддерживать ненужные.


### D: Dependency Inversion Principle (Принцип инверсии зависимостей)

Принцип инверсии зависимостей говорит, что модули верхнего уровня не должны зависеть от модулей нижнего уровня. Вместо этого оба типа модулей должны зависеть от абстракций.

### Пример в Go

```go
package main

import "fmt"

// Определим интерфейс, от которого будет зависеть верхний уровень
type Notifier interface {
    Notify() string
}

// SMSNotifier — низкоуровневый модуль
type SMSNotifier struct {
    phoneNumber string
}

func (s SMSNotifier) Notify() string {
    return fmt.Sprintf("Отправка SMS на номер: %s", s.phoneNumber)
}

// Модуль верхнего уровня зависит от интерфейса, а не от конкретной реализации
type NotificationService struct {
    notifier Notifier
}

func (ns NotificationService) SendNotification() {
    fmt.Println(ns.notifier.Notify())
}

func main() {
    smsNotifier := SMSNotifier{phoneNumber: "+123456789"}
    service := NotificationService{notifier: smsNotifier}
    service.SendNotification()
}

```

Здесь `NotificationService` зависит от интерфейса `Notifier`, а не от конкретной реализации `SMSNotifier`. Это позволяет легко изменять способ уведомления, не изменяя `NotificationService`.


## Заключение

**S** - Single Responsibility Principle: каждый класс/модуль должен иметь одну ответственность.

**O** - Open/Closed Principle: код открыт для расширения, но закрыт для модификации.

**L** - Liskov Substitution Principle: подклассы могут заменять базовые классы без нарушения программы.

**I** - Interface Segregation Principle: лучше много специфических интерфейсов, чем один общий.

**D** - Dependency Inversion Principle: высокоуровневые модули не зависят от низкоуровневых; оба зависят от абстракций.