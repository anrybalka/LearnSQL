### 1. **Выборка данных**
   - Получить список всех пользователей, зарегистрированных после 2023-01-01.
```sql
SELECT * FROM Users
WHERE created_at > '2023-01-01';
```
   - Вывести все товары, у которых цена выше 50000 и количество на складе меньше 10.
```sql
SELECT * FROM Products p
WHERE p.stock < 10
AND p.price >50000;
```
   - Найти пользователей, у которых день рождения в январе.
```sql
SELECT * FROM Users u
WHERE MONTH(u.birthdate) = 1;
```
### 2. **Агрегатные функции**
   - Посчитать, сколько заказов сделал каждый пользователь.
```sql
SELECT u.full_name, o.user_id, COUNT(*) AS total_Orders
FROM Orders o
JOIN Users u ON
	o.user_id = u.id
GROUP BY
	o.user_id , u.full_name
ORDER BY
	o.user_id
```
   - Найти среднюю цену товаров в таблице `Products`.
```sql
SELECT avg(p.price) AS avgPrice
FROM Products p
```
   - Определить, какой товар заказывали чаще всего.
```sql
SELECT p.title, o.product_id, count(amount) AS "Количество заказов"
FROM Orderitems o
JOIN Products p ON
	p.id = o.product_id
GROUP BY
	o.product_id, p.title
ORDER BY
	"Количество заказов" DESC
```

### 3. **Объединение таблиц (JOIN)**
   - Вывести список всех заказов с именами пользователей, которые их сделали.
```sql
SELECT u.full_name, o.created_at, o.id AS "ID Заказа"
FROM Orders o
JOIN Users u ON
	u.id = o.user_id
ORDER BY
	o.created_at
```
   - Получить список товаров, которые были заказаны хотя бы раз, с указанием их цены и количества заказов.
```sql
SELECT p.title, p.price, count(o.amount) AS "Количество заказов"
FROM Products p
JOIN Orderitems o ON
	o.product_id = p.id
GROUP BY
	p.title, p.price
HAVING
	count(o.amount) >= 1
```
   - Найти пользователей, которые не сделали ни одного заказа.
```sql
SELECT u.full_name, COUNT(o.id) AS "Количество заказов"
FROM Users u
LEFT JOIN Orders o ON
	o.user_id = u.id
GROUP BY
	u.full_name
HAVING
	COUNT(o.id) = 0
```

### 4. **Подзапросы**
   - Найти пользователей, которые заказали хотя бы один товар дороже 80000.
```sql
SELECT u.full_name, p.price
FROM Orderitems o
JOIN Products p ON
	p.id = o.product_id
JOIN Orders o2 ON
	o2.id = o.order_id
JOIN Users u ON
	u.id = o2.user_id
WHERE p.price >80000
```
   - Определить, какие товары не были заказаны ни разу.
```sql
SELECT * FROM Products p
LEFT JOIN Orderitems o ON o.product_id = p.id
WHERE o.amount IS NULL
```
   - Вывести пользователей, у которых сумма всех заказов превышает 100000.
```sql
SELECT u.full_name, SUM(p.price * o.amount) AS total_spent  FROM Orderitems o 
JOIN Orders o2 ON o2.id = o.order_id 
JOIN Users u ON u.id = o2.user_id 
JOIN Products p ON o.product_id =p.id
GROUP BY u.full_name
HAVING SUM(p.price * o.amount) > 100000

```

### 5. **Операции с датами**
   - Вывести пользователей, которые зарегистрировались в последний месяц.
```sql
SELECT * FROM Users u
WHERE MONTH(u.created_at) = 12;
```
   - Найти самый старый заказ в таблице `Orders`.
```sql
SELECT TOP 1 * FROM Orders o
ORDER BY o.created_at ASC;
```
   - Определить, сколько заказов было сделано в 2023 году.
```sql
SELECT COUNT(*) AS "Количество заказов 2023"
FROM Orders o
WHERE YEAR(o.created_at) = 2023;
```

### 6. **Работа с GROUP BY и HAVING**
   - Определить, какие пользователи сделали более одного заказа.
```sql
SELECT u.id, u.full_name, count(o.id) FROM Orders o 
JOIN Users u ON u.id = o.user_id
GROUP BY u.id
HAVING count(o.id)>1
```
   - Найти пользователей, которые потратили на заказы в сумме более 50000.
```sql
SELECT u.full_name, SUM(p.price * o.amount) AS total_spent  FROM Orderitems o 
JOIN Orders o2 ON o2.id = o.order_id 
JOIN Users u ON u.id = o2.user_id 
JOIN Products p ON o.product_id =p.id
GROUP BY u.full_name
HAVING SUM(p.price * o.amount) > 50000
```
   - Определить, какие товары заказывали больше 5 раз.
```sql
SELECT p.id, p.price, 
COALESCE(sum(o.amount),0) AS "Шт. заказано", 
COALESCE(sum(p.price*o.amount),0) AS "Заказано на сумму" 
FROM Products p 
LEFT JOIN Orderitems o ON o.product_id = p.id 
GROUP BY p.id 
HAVING COALESCE(sum(o.amount),0) > 5
```

### 7. **Работа с UPDATE и DELETE**
   - Уменьшить цену всех товаров дороже 100000 на 10%.
```sql
UPDATE Products
SET price = price-((price/100)*10)
WHERE price >100000
```
   - Удалить всех пользователей, которые не сделали ни одного заказа.
```sql
DELETE FROM Users
WHERE id IN(
SELECT u.id FROM Users u
LEFT JOIN Orders o ON o.user_id = u.id
WHERE o.id IS NULL
)
```
   - Обновить email всех пользователей, добавив к ним префикс "test_".
```sql
UPDATE Users 
SET email = CONCAT('test_',email)
```
   - Обновить email всех пользователей, убрав у них префикс "test_".
```sql
UPDATE Users
SET email = SUBSTRING(email, 6);
```