# Домашнее задание к занятию " `Индексы` " - `Сулименков Алексей`

---

## Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ

```SQL
SELECT table_schema,
    SUM(data_length) AS SUM_data_length,
    SUM(index_length) AS SUM_index_length,
    SUM(index_length) * 100 / SUM(data_length) AS 'index_length %'
FROM
    INFORMATION_SCHEMA.TABLES
GROUP BY
    table_schema
HAVING
    table_schema = "sakila";
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

---

## Задание 3\*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

---
