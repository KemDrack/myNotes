**Аутентификация**: JWT-токены с проверкой через Autostrade Identity Provider
Для обеспечения безопасного доступа к API использовалась система аутентификации на основе OAuth 2.0 и JWT-токенов.
- При каждом запросе клиент отправлял JWT-токен.
- API валидировал токен перед выполнением критической операции (например, поднятием шлагбаума).

**RBAC** (Role-Based Access Control)
Система ограничивала доступ к определённым операциям в зависимости от роли пользователя. Например, оператор мог управлять барьерами, а инженер — только просматривать их статус
- При авторизации проверялась роль пользователя.
- Доступ к API ограничивался на основе ролей.


**Авторизация через систему безопасности Autostrade** → Только проверенные пользователи могут управлять барьерами
Все запросы к API проходили аутентификацию через Autostrade(токен авторизации проверялся):

Ограничение запросов (Rate Limiting) через Redis:
```go
func rateLimitMiddleware(next echo.HandlerFunc) echo.HandlerFunc {
    return func(c echo.Context) error {
        userIP := c.RealIP()
        key := "rate_limit:" + userIP

        count, _ := redisClient.Incr(ctx, key).Result()
        if count > 10 {
            return c.JSON(http.StatusTooManyRequests, echo.Map{"error": "Too many requests"})
        }
        redisClient.Expire(ctx, key, 1*time.Minute)
        return next(c)
    }
}
```
📌 **Защита от [[SQL-инъекций]] (используем `$` вместо строковых подстановок):**
Безопасный вариант с параметризованными запросами
```go
func updateBarrierStatusInDB(barrierID, status string) error {
    _, err := db.Exec(
        "UPDATE barriers SET status = $1, last_updated = NOW() WHERE id = $2",
        status, barrierID,
    )
    return err
}
```
Теперь даже если атакующий передаст `42; DROP TABLE barriers;`, SQL-инъекция **не сработает**, так как база данных воспримет `barrierID` как строку, а не код.



**Как ты находил SQL-инъекции в запросе?**
📌 **Простой метод – искать подозрительные символы:**
```go
func isSQLInjection(input string) bool {
    blacklist := []string{";", "'", "--", "#", "/*", "*/", " OR ", " AND "}
    for _, b := range blacklist {
        if strings.Contains(strings.ToUpper(input), b) {
            return true
        }
    }
    return false
}
```

```go
if isSQLInjection(barrierID) {
    log.Warn("Попытка SQL-инъекции:", barrierID)
}
```
✅ Теперь **если в `barrierID` найдены `;`, `--`, `'`, система фиксирует попытку взлома.**


