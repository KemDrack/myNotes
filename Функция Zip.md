

```go
func main() {
	s1, s2 := []int{1, 2, 3}, []int{4, 5, 6, 7, 8}
	fmt.Println(zip(s1, s2)) // [[1 4] [2 5] [3 6]]
}

func zip(s1 []int, s2 []int) [][]int {
	minLen := len(s1)
	if len(s2) < minLen {
		minLen = len(s2) // Берём минимальную длину
	}

	result := make([][]int, minLen)

	for i := 0; i < minLen; i++ {
		result[i] = []int{s1[i], s2[i]} // Создаём пары
	}

	return result
}
```