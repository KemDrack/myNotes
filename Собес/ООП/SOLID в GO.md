

 *Эти знания нужны просто для флекса и прохождения собеседований*

## Принципы SOLID

**SOLID** — набор из 5 принципов для создания гибкого, масштабируемого и легко поддерживаемого кода. В языке Go, несмотря на то, что он не является классическим объектно-ориентированным языком, эти принципы всё ещё актуальны.

### S: Single Responsibility Principle (Принцип единственной ответственности)

Не нужно писать сложные функции, которые отвечают за слишком большую и разную функциональность

Если нам нужно сделать функцию для заказа в интернет магазине. Вы представьте без разбиения на слои сервисы, Handlerы, репозитории, просто зафигачили одну большую функцию, которая делает все (парсинг входящих запросов, валидацию, сохранение в базу всех сущностей, оповещение какое-нибудь, что заказ сделан, все зафигачили в одну функцию) и в будущем, каждый раз когда мы будем вносить какие либо изменение в эту функцию, вроде бы маленькое, вместо того чтобы перетестировать маленький участок кода, нам нужно будет перетестировать все, но это прям проблема, это будет занимать больше времени, сложнее. Так делать не нужно


Это означает, что каждая структура отвечает за свою задачу и не занимается ничем другим 
У модуля должна быть только одна причина для изменения
**Пример**: разделение логики обработки данных и их сохранения
```go
// User содержит данные о пользователе
type User struct {
	Name  string
	Email string
}

// UserRepository отвечает только за сохранение пользователей
type UserRepository struct{}

func (r UserRepository) Save(user User) {
	fmt.Printf("Пользователь %s сохранён в базу данных\n", user.Name)
}

// EmailService отвечает только за отправку писем
type EmailService struct{}

func (e EmailService) SendEmail(email string) {
	fmt.Printf("Письмо отправлено на %s\n", email)
}

func main() {
	user := User{Name: "Иван", Email: "ivan@example.com"}
	repo := UserRepository{}
	emailService := EmailService{}

	repo.Save(user)
	emailService.SendEmail(user.Email)
}
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

Объекты должны быть открыты для расширения и закрыты для модификации, так называемый принцип открытости/закрытости



Принцип открытости/закрытости подразумевает, что сущности должны быть открыты для расширения, но закрыты для модификации. Другими словами, нужно стараться добавлять новую функциональность через **расширение** существующего кода, а не через **изменение**

Для этого часто используется интерфейсное программирование:
```go
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


### L: Liskov Substitution Principle (Принцип подстановки Лисков)

LSP говорит, что объекты должны быть заменяемы их подтипами без нарушения работы программы
```go
// Интерфейс для геометрических фигур
type Shape interface {
    Area() int
}

// Прямоугольник
type Rectangle struct {
    Width, Height int
}

func (r Rectangle) Area() int {
    return r.Width * r.Height
}

// Квадрат
type Square struct {
    Side int
}

func (s Square) Area() int {
    return s.Side * s.Side
}

// Общая функция для расчёта площади
func PrintArea(shape Shape) {
    fmt.Printf("Площадь: %d\n", shape.Area())
}

func main() {
    rect := Rectangle{Width: 5, Height: 10}
    square := Square{Side: 7}

    PrintArea(rect)   // Площадь: 50
    PrintArea(square) // Площадь: 49
}
```

Теперь интерфейс `Shape` определяет только метод `Area`, который корректно реализуется как для прямоугольника, так и для квадрата

Удовлетворяет ожидания базового типа

<u>Объекты подклассов должны заменять объекты базового класса без нарушения программы.  
Пример: нельзя нарушать контракты базового класса в дочернем.</u>

### I: Interface Segregation Principle (Принцип разделения интерфейса)

Принцип разделения интерфейсов гласит, что лучше создавать много маленьких интерфейсов, чем один большой. Интерфейсы должны быть узкими и специализированными 

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

Модули должны зависеть от абстракций (интерфейсов), а не от конкретных реализаций.
Пример: необходимо использовать интерфейсы для внедрения зависимостей
#### Пример в Go
```go
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
    notifier Notifier // ВОТ ТУТ МЫ ИСПОЛЬЗУЕМ ИНТЕРФЕЙС
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

**S** - Single Responsibility Principle: каждый модуль должен иметь одну ответственность.

**O** - Open/Closed Principle: код открыт для расширения, но закрыт для модификации

**L** - Liskov Substitution Principle: подклассы могут заменять базовые классы без нарушения программы.

**I** - Interface Segregation Principle: лучше много специфических интерфейсов, чем один общий.

**D** - Dependency Inversion Principle: высокоуровневые модули не зависят от низкоуровневых; оба зависят от абстракций.