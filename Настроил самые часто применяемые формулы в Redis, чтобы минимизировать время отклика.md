

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
