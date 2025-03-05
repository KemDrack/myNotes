#Easy

Полиндром - слева направо и справа налево читается одинаково
>- aboba
```go
func isPalindrome(s string) bool {
    i, j := 0, len(s)-1
    for i < j {
        if !isalnum(s[i]) {
            i++
        } else if !isalnum(s[j]) {
            j--
        } else if tolower(s[i]) != tolower(s[j]) {
            return false
        } else {
            i, j = i+1, j-1 // Если символы совпадают, двигаем указатели к центру
        }
    }
    return true
}
 
func isalnum(ch byte) bool { // проверка на символы
    return (ch >= 'A' && ch <= 'Z') || (ch >= 'a' && ch <= 'z') || (ch >= '0' && ch <= '9')

}
func tolower(ch byte) byte { // Сравниваем символ без учета регистра
    if ch >= 'A' && ch <= 'Z' {
        return ch + 32
    }
    return ch
}
```

Алгоритм работает за `O(n)`, так как мы проходим строку один раз.  
Не использует лишнюю память (`O(1)`), работает только с указателями