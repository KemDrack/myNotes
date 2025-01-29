

- **Классифицированный случай:**
```json
{
    "case_id": "12345",
    "description": "Кейс о повреждении оборудования",
    "classification": "Critical",
    "created_at": "2025-01-28T12:34:56Z"
}
```
- **История решений:**
```json
{
    "case_id": "12345",
    "solution_id": "54321",
    "actions_taken": ["Restart equipment", "Check logs"],
    "result": "Resolved",
    "timestamp": "2025-01-28T13:00:00Z"
}
```
- Коллекция `cases` для хранения классифицированных случаев.
- Коллекция `solutions` для хранения истории решений.

