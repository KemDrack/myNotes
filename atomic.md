

Пакет `sync/atomic` предоставляет функции для выполнения атомарных операций над переменными. Эти операции позволяют **избежать** состояния гонки (race condition) при работе с общими данными из нескольких горутин, без использования блокировок (**`Mutex`** или **`RWMutex`**).

[[Атомарные операции]] гарантируют, что чтение, запись или изменение значения выполняется за один шаг (атомарно), без прерываний другими горутинами.

### **Ключевые функции пакета** `atomic`

atomic.**AddInt32**(&var, delta int32)
- Атомарно увеличивает значение переменной `var` на `delta`
```go
var counter int32

func increment(wg *sync.WaitGroup) {
    defer wg.Done()
    atomic.AddInt32(&counter, 1) // Увеличиваем счётчик атомарно
}

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go increment(&wg)
    }
    wg.Wait()
    fmt.Println("Итоговый счётчик:", counter) // Итоговый счётчик: 10
}

```

 atomic.**LoadInt32**(&var)
 - Атомарно читает значение переменной

atomic.**StoreInt32**(&var, value int32)
- Атомарно записывает значение в переменную

atomic.**CompareAndSwapInt32**(&var, old, new int32)
- Сравнивает значение переменной с `old`, и если они равны, заменяет его на `new`. Возвращает `true`, если замена произошла
```go
var value int32 = 10
func compareAndSwapExample() {
    old := int32(10)
    new := int32(20)
    // Если value равен old, заменяем его на new
    swapped := atomic.CompareAndSwapInt32(&value, old, new)
    if swapped {
        fmt.Println("Успешно заменено:", value) // Успешно заменено: 20
    } else {
        fmt.Println("Замена не удалась. Текущее значение:", value)
    }
}
func main() {
    compareAndSwapExample()
}
```
