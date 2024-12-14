
1. Отправляю {{domain}}/profile с полями:
{
    "name" : "student",
    "lname" : "student",
    "pname" : "student",
    "email" : "student@learning.com",
    "password" : "p11ass",
    "phone" : 79001002020
}

2. {{domain}}/profile/login_as
   
В представленном ответе получили JSON-объект, который является результатом успешной аутентификации или авторизации пользователя в системе

3. {{domain}}/federation - добавить федерацию
4. {{domain}}/company - добавить компанию


#### В этом проекте мы будем реализовывать задачу на добавление юр-лиц
***юр-лица ==> к компаниям ==> к федерациям***


1. В Postman создал папку legal_entities в federation, где будут храниться Мои API РУЧКИ
2. Создаем новую миграцию в проекте, чтобы мои данные где то хранились(create_legal_entitites)
3. Создаем папку olegalentities в ../web и обновляем makefile, чтобы openapi знал куда сгенерировать код 
       `oapi-codegen -config openapi/.openapi  -include-tags legalentities -package olegalentities openapi/openapi.yaml > ./internal/web/olegalentities/api.gen.go`


#### Структуры domain и oapi в проекте


oapi BankAccountDTO

BankAccountDTO:  
  type: object  
  required:  
    - uuid  
    - legalentityUUID  
    - bic  
    - bank  
    - address  
    - correspondent  
    - settlement  
    - currency  
    - comment  
  properties:  
    uuid:  
      type: string  
    legalentityUUID:  
      type: string  
    bic:  
      type: string  
    bank:  
      type: string  
    address:  
      type: string  
    correspondent:  
      type: string  
    settlement:  
      type: string  
    currency:  
      type: string  
    comment:  
      type: string

