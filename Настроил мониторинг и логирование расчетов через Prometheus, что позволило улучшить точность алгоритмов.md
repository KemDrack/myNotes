


**Какие показатели отслеживались в Prometheus?**  
✔ Время расчета стоимости полиса (`calculation_duration_seconds`)  
✔ Количество расчетов (`calculation_total`)  
✔ Среднее отклонение от прогнозируемой цены (`price_deviation`)  
✔ Ошибки в расчетах (`calculation_errors_total`)

**📌 Как выглядит мониторинг?**  
✅ Grafana показывает график расчета стоимости полисов  
✅ Если расчет занимает больше 500 мс – выводится алерт  
✅ Если цена слишком сильно изменилась – отправляется лог


Создал метрики
```go
var (
    calculationDuration = prometheus.NewHistogram(prometheus.HistogramOpts{
        Name:    "calculation_duration_seconds",
        Help:    "Время расчета стоимости полиса",
        Buckets: prometheus.LinearBuckets(0.01, 0.05, 10),
    })

    calculationTotal = prometheus.NewCounter(prometheus.CounterOpts{
        Name: "calculation_total",
        Help: "Общее количество расчетов стоимости полиса",
    })

    calculationErrors = prometheus.NewCounter(prometheus.CounterOpts{
        Name: "calculation_errors_total",
        Help: "Количество ошибок при расчете стоимости полиса",
    })
)

func init() {
    prometheus.MustRegister(calculationDuration, calculationTotal, calculationErrors)
}
```



📌 **Измерение времени выполнения алгоритма расчета:**
```go
func calculatePolicyPrice(riskFactors map[string]float64) float64 {
    start := time.Now()
    defer func() {
        duration := time.Since(start).Seconds()
        calculationDuration.Observe(duration)
    }()

    basePrice := 10000.0
    finalPrice := basePrice
    for _, factor := range riskFactors {
        finalPrice *= factor
    }

    calculationTotal.Inc() // Увеличиваем счетчик расчетов
    return finalPrice
}
```

 **Как ты логировал ошибки в расчетах?**
 ```go
 func validateCalculation(userID int, expectedPrice, calculatedPrice float64) {
    deviation := math.Abs(calculatedPrice-expectedPrice) / expectedPrice
    if deviation > 0.2 { // Если отклонение больше 20%
        log.Warnf("Аномальный расчет! UserID: %d, Expected: %.2f, Calculated: %.2f", userID, expectedPrice, calculatedPrice)
        calculationErrors.Inc() // Увеличиваем метрику ошибок
    }
}
```


Анализ логов помог выявить и исправить баги, что снизило количество ошибок. Все метрики визуализировались в Grafana, что позволило оперативно реагировать на проблемы."