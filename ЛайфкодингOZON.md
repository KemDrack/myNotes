[[Функция Zip]]
[[Задача с Cache]]
[[Является ли слайс монотонным]]
[[Реализация брутфорса для восстановления пароля]]

#### Select
```go
func main() {  
    ch := make(chan int,1) // Создаём небуферизированный канал  
  
    for i := 0; i < 10; i++ {  
       select {  
       case x := <-ch:  
          print(x) // Читаем из канала  
       case ch <- i:  
          // Пишем i в канал  
       }  
    }  
}
0 2 4 6 8
```
**0 2 4 6 8**
То есть, нечётные `i` просто **не попадают в канал, потому что в этот момент канал уже пуст**.
#### defer
```go
type X struct {  
    V int  
}  
  
func (x X) S() {  
    fmt.Println(x.V)  
}  
  
func main() {  
    x := X{123}  
    defer func() {  
       x.S()  
    }() // Откладываем вызов метода  
    x.V = 456  
}
456
```
---
```go
func main() {
    fmt.Printf("result: %d\n", test())
}

func test() (x int) {
    defer func(n int) {
        fmt.Printf("x as parameter: %d\n", n)
        fmt.Printf("x after return: %d\n", x)
        x *= 10
    }(x) // Передача x в defer

    x = 7
    return 9
}
x as parameter: 0
x after return: 9
result: 90
```
- В `defer` передается **копия** `x` (а `x` в этот момент **0**).
- В `defer` создается **локальная переменная `n`**, которая получает `0`.
- `return 9` присваивает `x = 9` перед выходом.
- После `return` вызывается `defer` (но `n` уже передан ранее как `0`)

- `x as parameter: 0` – потому что `n` был передан в `defer` до изменений `x`.
- `x after return: 9` – потому что `return 9` уже обновил `x = 9`.
- `x *= 10` делает `x = 90`, **но return уже завершился, и это изменение не повлияет на возвращаемое значение**.


#### Слайсы
```go
func main() {  
    x := []int{}  
    x = append(x, 0)  
    x = append(x, 1)  
    x = append(x, 2)  
    y := append(x, 3)  
    z := append(x, 4)  
    fmt.Println(x, y, z) // 0 1 2    0 1 2 4   0 1 2 4  
}
```

```go
Дан отсортированный массив чисел. 
Нужно вернуть отсортированный массив квадратов этих чисел.

Input: nums = [0, 1, 1, 6, 12, 20]
Output: squares = [0, 1, 1, 36, 144, 400]
func sortedSquares(nums []int) []int {
	n := len(nums)
	result := make([]int, n)
	// Два указателя на начало и конец массива
	left, right := 0, n-1
	index := n - 1 // Заполняем с конца
	for left <= right {
		leftSq, rightSq := nums[left]*nums[left], nums[right]*nums[right]
		if leftSq > rightSq {
			result[index] = leftSq
			left++
		} else {
			result[index] = rightSq
			right--
		}
		index--
	}
	return result
}
func main() {
	nums := []int{0, 1, 1, 6, 12, 20}
	fmt.Println(sortedSquares(nums)) // [0, 1, 1, 36, 144, 400]
}

```

#### Строки
```go
func main() {
 s := "test"
 println(s[0])
 s[0] = "A"
 println(s)
}
РЕШЕНИЕ
s := "test"  
println(s[0])  
r := []rune(s)  
r[0] = 'A'  
s = string(r)  
fmt.Println(s)

Доп вопрос - как сделать присвоение
```