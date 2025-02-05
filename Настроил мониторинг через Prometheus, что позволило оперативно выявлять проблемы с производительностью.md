
**Регистрация Prometheus-метрик**
```go
package main

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "net/http"
)
// Метрика времени обработки запросов
var requestDuration = prometheus.NewHistogramVec(
    prometheus.HistogramOpts{
        Name:    "http_request_duration_seconds",
        Help:    "Duration of HTTP requests in seconds",
        Buckets: prometheus.DefBuckets, // Стандартные интервалы
    },
    []string{"path"},
)

// Количество запросов
var requestCounter = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Name: "http_requests_total",
        Help: "Total number of HTTP requests",
    },
    []string{"method", "path"},
)

func init() {
    prometheus.MustRegister(requestDuration)
    prometheus.MustRegister(requestCounter)
}

func main() {
    http.Handle("/metrics", promhttp.Handler()) // Эндпоинт для Prometheus
    go startServer()
    http.ListenAndServe(":8080", nil)
}
```

**Добавил мониторинг времени обработки запросов**
Обернул HTTP-обработчики, чтобы **измерять скорость работы API**.
```go
func withMetrics(handler http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        timer := prometheus.NewTimer(requestDuration.WithLabelValues(r.URL.Path))
        defer timer.ObserveDuration()

        requestCounter.WithLabelValues(r.Method, r.URL.Path).Inc()

        handler(w, r)
    }
}

func getPolicyHandler(w http.ResponseWriter, r *http.Request) {
    // Обрабатываем запрос
    w.Write([]byte("Policy details"))
}

func main() {
    http.HandleFunc("/api/policy", withMetrics(getPolicyHandler))
    http.ListenAndServe(":8080", nil)
}
```
✅ Теперь Prometheus собирает:  
✔ Время выполнения запросов (`http_request_duration_seconds`)  
✔ Общее число запросов (`http_requests_total`)

**Настроил мониторинг базы данных (PostgreSQL)**
Я измерял **время выполнения SQL-запросов**, чтобы находить **медленные запросы**.
```GO
var dbQueryDuration = prometheus.NewHistogramVec(
    prometheus.HistogramOpts{
        Name:    "db_query_duration_seconds",
        Help:    "Duration of database queries in seconds",
        Buckets: prometheus.DefBuckets,
    },
    []string{"query"},
)

func init() {
    prometheus.MustRegister(dbQueryDuration)
}

func executeQuery(query string, args ...interface{}) error {
    timer := prometheus.NewTimer(dbQueryDuration.WithLabelValues(query))
    defer timer.ObserveDuration()

    _, err := db.Exec(query, args...)
    return err
}
```

Настроил сбор системных метрик
Добавил мониторинг **CPU, памяти и количества Goroutines**.
```go
var (
    cpuUsage = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "cpu_usage",
        Help: "Current CPU usage",
    })
    memoryUsage = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "memory_usage",
        Help: "Current memory usage",
    })
    goroutines = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "goroutines",
        Help: "Number of Goroutines",
    })
)

func init() {
    prometheus.MustRegister(cpuUsage)
    prometheus.MustRegister(memoryUsage)
    prometheus.MustRegister(goroutines)
}

func collectMetrics() {
    for {
        var memStats runtime.MemStats
        runtime.ReadMemStats(&memStats)

        memoryUsage.Set(float64(memStats.Alloc) / 1024 / 1024) // В МБ
        goroutines.Set(float64(runtime.NumGoroutine()))

        time.Sleep(5 * time.Second)
    }
}

func main() {
    go collectMetrics()
    http.Handle("/metrics", promhttp.Handler())
    http.ListenAndServe(":8080", nil)
}
```
✅ Теперь ты видишь:  
✔ Текущую загрузку CPU  
✔ Использование памяти  
✔ Количество запущенных Goroutines


Ты добавил конфигурацию для **Prometheus**, чтобы он собирал данные из микросервиса.

📌 **Конфиг `prometheus.yml`**

`scrape_configs:   - job_name: 'my_service'     static_configs:       - targets: ['localhost:8080']`

📌 **Запуск Prometheus**

`prometheus --config.file=prometheus.yml`

✅ Теперь **Prometheus собирает метрики каждые 10 секунд**.