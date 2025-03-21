
Для поиска можно не поддерживать полную консистентность данных, мы можем подождать, чтобы избежать постоянной индексации
Можно разделить Search & Read




- Микросервис принимает описание страхового случая от клиента
- **Elasticsearch выполняет поиск по индексу** с уже классифицированными случаями, чтобы найти наиболее похожие.
- Если совпадения найдены с высокой релевантностью, **кейс классифицируется автоматически**


**Создание индекса в Elasticsearch**
Для хранения и поиска страховых случаев создаётся индекс
```json
PUT insurance_cases
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "standard",
          "stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "custom_analyzer"
      },
      "category": {
        "type": "keyword"
      }
    }
  }
}
```
✅ `description` — текстовое поле, анализируется с помощью предустановленного анализатора.
✅ `category` — поле типа `keyword`, в котором хранится категория страхового случая.

**Примеры известных случаев добавляются в Elasticsearch:**(обучение модели)
```json
POST insurance_cases/_doc/1
{
  "description": "The car was damaged in an accident on the highway.",
  "category": "Car Accident"
}

POST insurance_cases/_doc/2
{
  "description": "A fire broke out in the apartment, causing major damage.",
  "category": "Fire Damage"
}

POST insurance_cases/_doc/3
{
  "description": "The house was burglarized and valuables were stolen.",
  "category": "Theft"
}
```


Когда приходит новый страховой случай, мы ищем **похожие описания в индексе** и выбираем наиболее релевантную категорию.
```go
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"strings"

	"github.com/elastic/go-elasticsearch/v8"
)

type InsuranceCase struct {
	Description string `json:"description"`
	Category    string `json:"category,omitempty"`
}

var es *elasticsearch.Client

func init() {
	var err error
	es, err = elasticsearch.NewDefaultClient()
	if err != nil {
		log.Fatalf("Failed to connect to Elasticsearch: %v", err)
	}
}

func classifyInsuranceCase(description string) (string, error) {
	// Поиск похожих записей в Elasticsearch
	query := fmt.Sprintf(`
	{
		"query": {
			"match": {
				"description": "%s"
			}
		},
		"size": 1
	}`, description)

	res, err := es.Search(
		es.Search.WithContext(context.Background()),
		es.Search.WithIndex("insurance_cases"),
		es.Search.WithBody(strings.NewReader(query)),
		es.Search.WithPretty(),
	)
	if err != nil {
		return "", err
	}
	defer res.Body.Close()

	// Разбираем ответ
	var result map[string]interface{}
	if err := json.NewDecoder(res.Body).Decode(&result); err != nil {
		return "", err
	}

	// Достаём первую найденную категорию
	hits := result["hits"].(map[string]interface{})["hits"].([]interface{})
	if len(hits) == 0 {
		return "Unknown", nil
	}

	source := hits[0].(map[string]interface{})["_source"].(map[string]interface{})
	category := source["category"].(string)

	return category, nil
}

func classifyHandler(w http.ResponseWriter, r *http.Request) {
	var insuranceCase InsuranceCase
	if err := json.NewDecoder(r.Body).Decode(&insuranceCase); err != nil {
		http.Error(w, "Invalid request body", http.StatusBadRequest)
		return
	}

	category, err := classifyInsuranceCase(insuranceCase.Description)
	if err != nil {
		http.Error(w, "Failed to classify insurance case", http.StatusInternalServerError)
		return
	}

	insuranceCase.Category = category

	// Возвращаем ответ клиенту
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(insuranceCase)
}

func main() {
	http.HandleFunc("/classify", classifyHandler)

	log.Println("Server started on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```