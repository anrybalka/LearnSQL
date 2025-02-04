### 1. **Выборка данных**
   - Получить список всех пользователей, зарегистрированных после 2023-01-01.
```sql
SELECT * FROM public.users
WHERE created_at > '2023-01-01';
```
   - Вывести все товары, у которых цена выше 50000 и количество на складе меньше 10.
```sql
SELECT * FROM public.products p
WHERE p.stock < 10
AND p.price >50000;
```
   - Найти пользователей, у которых день рождения в январе.
```sql
SELECT * FROM public.users u
WHERE EXTRACT(MONTH FROM u.birthdate) = 1
```
### 2. **Агрегатные функции**
   - Посчитать, сколько заказов сделал каждый пользователь.
```sql
SELECT u."name", o.user_id, COUNT(*) AS total_orders
FROM orders o
JOIN users u ON
	o.user_id = u.id
GROUP BY
	o.user_id , u."name"
ORDER BY
	o.user_id
```
   - Найти среднюю цену товаров в таблице `Products`.
```sql
SELECT avg(p.price) AS avgPrice
FROM products p
```
   - Определить, какой товар заказывали чаще всего.
```sql
SELECT p."name", o.product_id, count(amount) AS "Количество заказов"
FROM orderitems o
JOIN products p ON
	p.id = o.product_id
GROUP BY
	o.product_id, p."name"
ORDER BY
	"Количество заказов" DESC
```

### 3. **Объединение таблиц (JOIN)**
   - Вывести список всех заказов с именами пользователей, которые их сделали.
```sql
SELECT u."name", o.created_at, o.id AS "ID Заказа"
FROM orders o
JOIN users u ON
	u.id = o.user_id
ORDER BY
	o.created_at
```
   - Получить список товаров, которые были заказаны хотя бы раз, с указанием их цены и количества заказов.
```sql
SELECT p."name", p.price, count(o.amount) AS "Количество заказов"
FROM products p
JOIN orderitems o ON
	o.product_id = p.id
GROUP BY
	p."name", p.price
HAVING
	count(o.amount) >= 1
```
   - Найти пользователей, которые не сделали ни одного заказа.
```sql
SELECT u."name", COUNT(o.id) AS "Количество заказов"
FROM users u
LEFT JOIN orders o ON
	o.user_id = u.id
GROUP BY
	u."name"
HAVING
	COUNT(o.id) = 0
```

### 4. **Подзапросы**
   - Найти пользователей, которые заказали хотя бы один товар дороже 80000.
```sql
SELECT u."name", p.price
FROM orderitems o
JOIN products p ON
	p.id = o.product_id
JOIN orders o2 ON
	o2.id = o.order_id
JOIN users u ON
	u.id = o2.user_id
WHERE p.price >80000
```
   - Определить, какие товары не были заказаны ни разу.
```sql
SELECT * FROM products p
LEFT JOIN orderitems o ON o.product_id = p.id
WHERE o.amount IS NULL
```
   - Вывести пользователей, у которых сумма всех заказов превышает 100000.
```sql
SELECT u."name", SUM(p.price * o.amount) AS total_spent  FROM orderitems o 
JOIN orders o2 ON o2.id = o.order_id 
JOIN users u ON u.id = o2.user_id 
JOIN products p ON o.product_id =p.id
GROUP BY u."name"
HAVING SUM(p.price * o.amount) > 100000

```

### 5. **Операции с датами**
   - Вывести пользователей, которые зарегистрировались в последний месяц.
```sql
SELECT * FROM users u 
WHERE EXTRACT(MONTH FROM u.created_at)=12
```
   - Найти самый старый заказ в таблице `Orders`.
```sql
SELECT * FROM orders o 
ORDER BY o.created_at ASC
LIMIT 1
```
   - Определить, сколько заказов было сделано в 2023 году.
```sql
SELECT COUNT(*) AS "Количество заказов 2023" FROM orders o 
WHERE EXTRACT(YEAR FROM o.created_at) = 2023
```

### 6. **Работа с GROUP BY и HAVING**
   - Определить, какие пользователи сделали более одного заказа.
```sql
SELECT u.id, u."name", count(o.id) FROM orders o 
JOIN users u ON u.id = o.user_id
GROUP BY u.id
HAVING count(o.id)>1
```
   - Найти пользователей, которые потратили на заказы в сумме более 50000.
```sql
SELECT u."name", SUM(p.price * o.amount) AS total_spent  FROM orderitems o 
JOIN orders o2 ON o2.id = o.order_id 
JOIN users u ON u.id = o2.user_id 
JOIN products p ON o.product_id =p.id
GROUP BY u."name"
HAVING SUM(p.price * o.amount) > 50000
```
   - Определить, какие товары заказывали больше 5 раз.
```sql
SELECT p.id, p.price, 
COALESCE(sum(o.amount),0) AS "Шт. заказано", 
COALESCE(sum(p.price*o.amount),0) AS "Заказано на сумму" 
FROM products p 
LEFT JOIN orderitems o ON o.product_id = p.id 
GROUP BY p.id 
HAVING COALESCE(sum(o.amount),0) > 5
```

### 7. **Работа с UPDATE и DELETE**
   - Уменьшить цену всех товаров дороже 100000 на 10%.
```sql
UPDATE products
SET price = price-((price/100)*10)
WHERE price >100000
```
   - Удалить всех пользователей, которые не сделали ни одного заказа.
```sql
DELETE FROM users
WHERE id IN(
SELECT u.id FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL
)
```
   - Обновить email всех пользователей, добавив к ним префикс "test_".
```sql
UPDATE users 
SET email = CONCAT('test_',email)
```
   - Обновить email всех пользователей, убрав у них префикс "test_".
```sql
UPDATE users
SET email = SUBSTRING(email, 6);
```