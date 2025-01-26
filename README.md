# Домашнее задание к занятию " `Индексы` " - `Сулименков Алексей`

---

## Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ

```SQL
SELECT TABLE_SCHEMA,
    SUM(data_length) AS SUM_data_length,
    SUM(index_length) AS SUM_index_length,
    SUM(index_length) * 100 / SUM(data_length) AS 'index_length %'
FROM
    INFORMATION_SCHEMA.TABLES
WHERE
    TABLE_SCHEMA = 'sakila'
GROUP BY
    TABLE_SCHEMA;
```

<details> <summary>Скриншот к заданию 1</summary>

![task1](https://github.com/biparasite/DB-12-05HW/blob/main/task_1.png "task1")

</details>

---

## Задание 2

Выполните explain analyze следующего запроса:

```SQL
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Ответ

```SQL
EXPLAIN ANALYZE
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id;
```

![task2](https://github.com/biparasite/DB-12-05HW/blob/main/task_2.1.png "task2")

Перечислите узкие места:

- Перебор большого числа строк (642000)
- отсутствие индексов

Оптимизируйте запрос:

- Создание индектов для payment_date и rental_date

```SQL
ALTER TABLE `payment` ADD INDEX `payment_date_index` (`payment_date`);
ALTER TABLE `rental` ADD INDEX `rental_date_index` (`rental_date`);
SHOW INDEX FROM `payment` AND `rental`;
```

<details> <summary>Скриншот к заданию 2</summary>

![task2](https://github.com/biparasite/DB-12-05HW/blob/main/task_2.2.png "task2")

</details>

- Убрал из запроса f.title

```SQL
EXPLAIN ANALYZE
SELECT DISTINCT  CONCAT(c.last_name, ' ', c.first_name ), sum(p.amount) OVER (PARTITION BY c.customer_id )
FROM
    payment p, rental r, customer c
WHERE
    payment_date >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
    AND
    p.payment_date = r.rental_date
    AND
    r.customer_id = c.customer_id;
```

![task2](https://github.com/biparasite/DB-12-05HW/blob/main/task_2.3.png "task2")

- Выриант 2

#### С использованием мультиколоночных индексов

```SQL
CREATE INDEX idx_r_customer_rental_date ON rental(customer_id, rental_date);
CREATE INDEX idx_p_payment_date_amount ON payment(payment_date, amount);
```

```SQL
EXPLAIN ANALYZE
SELECT DISTINCT  CONCAT(c.last_name, ' ', c.first_name ), SUM(p.amount) OVER (PARTITION BY c.customer_id )
FROM sakila.customer c
JOIN sakila.rental r ON c.customer_id = r.customer_id
JOIN sakila.payment p ON r.rental_date = p.payment_date
WHERE r.rental_date BETWEEN '2005-07-30' AND '2005-07-31';
```

![task2](https://github.com/biparasite/DB-12-05HW/blob/main/task_2.4.png "task2")

---

## Задание 3\*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

### Ответ

GiST, SP-GiST, BRIN индексы.

---
