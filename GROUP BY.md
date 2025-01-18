

**`GROUP BY`** используется для группировки строк таблицы по значениям одного или нескольких столбцов. После группировки к этим группам можно применять агрегатные функции.

`GROUP BY` группирует строки с одинаковыми значениями в указанном столбце.

```sql
SELECT department, COUNT(*)
FROM employees;
```
- Этот запрос выдаст ошибку, так как SQL не знает, как обработать столбец `department`

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```
- Здесь строки группируются по значениям столбца `department`.
- Для каждой группы `COUNT(*)` подсчитывает количество строк.



```sql
SELECT department, COUNT(*) AS employees_count, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

**Результат:**

| department | employees_count | avg_salary |
| ---------- | --------------- | ---------- |
| IT         | 2               | 5500       |
| HR         | 2               | 4250       |
| Finance    | 1               | 7000       |




Example(table `sales`):

1) Запрос: "Сколько продаж у каждого продукта в каждом регионе?"
```
| id  | product | region | amount |
| --- | ------- | ------ | ------ |
| 1   | Phone   | North  | 1000   |
| 2   | Laptop  | North  | 2000   |
| 3   | Phone   | South  | 1500   |
| 4   | Laptop  | South  | 3000   |
| 5   | Tablet  | North  | 1200   |
| 6   | Phone   | North  | 800    |
```

```sql
SELECT product, region, COUNT(*) AS sales_count
FROM sales
GROUP BY product, region;
```

2) Запрос: "Сколько продаж у каждого продукта?"
```sql
SELECT product, COUNT(*) AS sales_count
FROM sales
GROUP BY product;
```

3) Запрос: "Какой общий объём продаж у каждого продукта?"
```sql
SELECT product, SUM(amount) AS total_sales
FROM sales
GROUP BY product;
```

4) Запрос: "Какой средний объём продаж в каждом регионе?"
```sql
SELECT region, AVG(amount) AS avg_sales
FROM sales
GROUP BY region;
```

5) Запрос: "Какие продукты имеют общий объём продаж больше 3000?"
Использование `HAVING` с `GROUP BY`
```sql
SELECT product, SUM(amount) AS total_sales
FROM sales
GROUP BY product
HAVING SUM(amount) > 3000;
```

6) Запрос: "Сколько уникальных продуктов продаётся в каждом регионе?"
```sql
SELECT region, COUNT(DISTINCT product) AS unique_products
FROM sales
GROUP BY region;
```

7) Запрос: "Отсортируй регионы по объёму продаж (по убыванию)"
```sql
SELECT region, SUM(amount) AS total_sales
FROM sales
GROUP BY region
ORDER BY total_sales DESC;
```
