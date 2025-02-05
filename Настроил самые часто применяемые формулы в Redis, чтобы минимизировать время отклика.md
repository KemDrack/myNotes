Когда клиент подавал страховой случай, система могла быстро **извлекать нужные коэффициенты из Redis** и рассчитывать риск **без обращения к базе данных**
- Риск ДТП зависит от возраста и стажа водителя
- Риск болезни зависит от возраста и хронических заболеваний
**Коэффициенты** хранились в Redis:
```go
redisClient.HSet(ctx, "risk_factors", "age:18-25", "1.5")
redisClient.HSet(ctx, "risk_factors", "age:26-35", "1.2")
redisClient.HSet(ctx, "risk_factors", "experience:0-5", "1.8")
redisClient.HSet(ctx, "risk_factors", "experience:6-10", "1.3")
redisClient.HSet(ctx, "risk_factors", "chronic:yes", "2.0")
redisClient.HSet(ctx, "risk_factors", "chronic:no", "1.0")
```



[[Redis]] - in-memory key-value хранилище, идеально подходящее для хранения часто используемых данных

Чтобы ускорить отклик системы, результаты формул можно сохранить в Redis. Это называется **кешированием**.
- Сначала проверяется, есть ли результат формулы в Redis (кеш-хит).
- Если результата нет (кеш-мисс), он вычисляется, сохраняется в Redis, а затем возвращается клиенту.

```go
func calculateFinalPrice(basePrice, discount, tax float64) float64 {
	return basePrice * (1 - discount) + tax
}

func getCachedPrice(rdb *redis.Client, basePrice, discount, tax float64) float64 {
	// Формируем уникальный ключ для формулы
	key := fmt.Sprintf("price:%f:%f:%f", basePrice, discount, tax)

	// Проверяем, есть ли результат в Redis
	val, err := rdb.Get(ctx, key).Result()
	if err == redis.Nil {
		// Если результата нет, вычисляем его
		finalPrice := calculateFinalPrice(basePrice, discount, tax)

		// Сохраняем результат в Redis с TTL (например, 1 час)
		err := rdb.Set(ctx, key, finalPrice, time.Hour).Err()
		if err != nil {
			log.Fatalf("Ошибка сохранения в Redis: %v", err)
		}

		log.Println("Результат сохранён в Redis")
		return finalPrice
	} else if err != nil {
		log.Fatalf("Ошибка получения из Redis: %v", err)
	}

	// Конвертируем строку из Redis в float64
	var cachedPrice float64
	fmt.Sscanf(val, "%f", &cachedPrice)
	log.Println("Результат получен из Redis")
	return cachedPrice
}

func main() {
	rdb := connectRedis()
	defer rdb.Close()

	basePrice := 100.0
	discount := 0.15
	tax := 10.0

	finalPrice := getCachedPrice(rdb, basePrice, discount, tax)
	fmt.Printf("Итоговая цена: %.2f\n", finalPrice)
}
```
