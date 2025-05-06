# pr_10

## Аналитика с использованием сложных типов данных. Поиск и анализ продаж.
# 1. Создать материализованное представление для таблицы customer_sales
```
CREATE MATERIALIZED VIEW customer_search AS (
SELECT
customer_json -> 'customer_id' AS customer_id, customer_json,to_tsvector('english', customer_json) AS search_vector FROM customer_sales
);
```

![image](https://github.com/user-attachments/assets/0339ec9a-3c2d-4f00-ba94-026d144bee2c)

# 2 Создать индекс GIN в представлении
```
CREATE INDEX customer_search_gin_idx ON customer_search USING GIN(search_vector);
```

![image](https://github.com/user-attachments/assets/788b54b0-9b39-47d2-af1b-9d1101306aa5)

# 3 Выполнить запрос, используя новую базу данных с возможностью поиска
```
SELECT
customer_id,
customer_json
FROM customer_search
WHERE search_vector @@ plainto_tsquery('english', 'Danny Bat');
```

![image](https://github.com/user-attachments/assets/dcbfc1d2-7c78-449d-b8d9-c0867262a74f)

# 4 Вывести уникальный список скутеров и автомобилей (и удаление ограниченных выпусков) с помощью DISTINCT
```
SELECT DISTINCT
p1.model,
p2.model
FROM products p1
LEFT JOIN products p2 ON TRUE
WHERE p1.product_type = 'scooter'
AND p2.product_type = 'automobile'
AND p1.model NOT ILIKE '%Limited
Edition%';
```

![image](https://github.com/user-attachments/assets/5d5d76f0-0999-4f33-a77d-b554d57038e9)

# 5 Преобразование вывода в запрос
```
SELECT DISTINCT
plainto_tsquery('english', p1.model) &&
plainto_tsquery('english', p2.model)
FROM products p1
LEFT JOIN products p2 ON TRUE
WHERE p1.product_type = 'scooter'
AND p2.product_type = 'automobile'
AND p1.model NOT ILIKE '%Limited Edition%';
```

![image](https://github.com/user-attachments/assets/ce656360-ee7f-4553-a04e-c1fdde03c7cd)

# 6 Запрос базы данных, используя каждый из объектов tsquery, и подсчитать вхождения для каждого объекта
```
SELECT
sub.query,
(
SELECT COUNT(1)
FROM customer_search
WHERE customer_search.search_vector @@ sub.query)FROM (
SELECT DISTINCT
plainto_tsquery('english', p1.model) &&
plainto_tsquery('english', p2.model) AS query
FROM products p1
LEFT JOIN products p2 ON TRUE
WHERE p1.product_type = 'scooter’
AND p2.product_type = 'automobile’
AND p1.model NOT ILIKE '%Limited Edition%’
) sub
ORDER BY 2 DESC;
```

![image](https://github.com/user-attachments/assets/d799559e-ca3f-4a2e-a794-3629e3294f15)
