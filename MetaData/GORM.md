

#### DB.Model() - Указание модели для выполнения операций

`db.Model(&User{}).Where("id = ?", 1).Update("name", "New Name")`

Select

Используется для указания конкретных колонок, которые нужно выбрать из таблицы. db.Select("name, age").Find(&users)


Joins



Group
Find
First
Update
Updates