# pr_10

## Аналитика с использованием сложных типов данных. Поиск и анализ продаж.

### Цели работы:
Разработать систему поиска клиентов по контактным данным и истории покупок, реализовать полнотекстовый поиск с помощью индексов GIN, проанализировать частоту совместных покупок разных товаров

### 1. 
Используя таблицу customer_sales, создайте доступное для поиска представление с одной записью для каждого клиента. Это представление
должно быть отключено от столбца customer_id и доступно для поиска по всей базе данных, что связано с этим клиентом:
• имя,
• адрес электронной почты,
• телефон,
• приобретенные продукты.
Можно также включить и другие поля.
```
CREATE MATERIALIZED VIEW customer_search AS (
SELECT
customer_json -> 'customer_id' AS customer_id, customer_json,to_tsvector('english', customer_json) AS search_vector FROM customer_sales
);
```
<img width="927" height="280" alt="image" src="https://github.com/user-attachments/assets/15e343f8-6ba7-4448-8c79-a4324f74cefb" />

![image](https://github.com/user-attachments/assets/0339ec9a-3c2d-4f00-ba94-026d144bee2c)

### 2 
Создайте доступный для поиска индекс, созданного вами ранее
представления.
```
CREATE INDEX customer_search_gin_idx ON customer_search USING GIN(search_vector);
```

![image](https://github.com/user-attachments/assets/788b54b0-9b39-47d2-af1b-9d1101306aa5)

### 3 
У кулера с водой продавец спрашивает, можете ли вы использовать свой новый поисковый прототип, чтобы найти покупателя по имени Дэнни, купившего скутер Bat. Запросите новое представление с возможностью поиска, используя ключевые слова «Danny Bat». Какое количество строк вы получили?
```
SELECT
customer_id,
customer_json
FROM customer_search
WHERE search_vector @@ plainto_tsquery('english', 'Danny Bat');
```

![image](https://github.com/user-attachments/assets/dcbfc1d2-7c78-449d-b8d9-c0867262a74f)

### 4 
Отдел продаж хочет знать, насколько часто люди покупают скутер и
автомобиль. Выполните перекрестное соединение таблицы продуктов с самой собой, чтобы получить все отдельные пары продуктов и удалить одинаковые пары (например, если название продукта совпадает). Для каждой пары выполните поиск в
представлении, чтобы узнать, сколько клиентов соответствует обоим продуктам в паре. Можно предположить, что выпуски ограниченной серии можно сгруппировать вместе с их аналогом стандартной модели (например, Bat и Bat Limited Edition можно считать одним и тем же скутером).

Вывести уникальный список скутеров и автомобилей (и удаление ограниченных выпусков) с помощью DISTINCT
```
SELECT DISTINCT
p1.model,
p2.model
FROM products p1
LEFT JOIN products p2 ON TRUE
WHERE p1.product_type = 'scooter' AND p2.product_type = 'automobile' AND p1.model NOT ILIKE '%Limited Edition%';
```

![image](https://github.com/user-attachments/assets/5d5d76f0-0999-4f33-a77d-b554d57038e9)

### 5 Преобразование вывода в запрос
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

### 6 Запрос базы данных, используя каждый из объектов tsquery, и подсчитать вхождения для каждого объекта
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

### Выводы:
Создано материализованное представление с данными клиентов и поисковым вектором, построен GIN-индекс для ускорения поисковых запросов, разработан метод анализа парных покупок с фильтрацией дубликатов
