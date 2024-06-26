# Домашнее задание к занятию 2. «SQL» - `Шульгатый Станислав`

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

### Ответ
```
version: '3'
services:
  postgreSQL:
    image:  postgres:12
    container_name: postgreSQL
    ports:
      - "5432:5432"
    volumes:
    - pg_data:/var/lib/postgresql/data/pgdata
    - /home/megion/Projects/sql/backups:/backups
    environment:
      POSTGRES_USER:  postgres
      POSTGRES_PASSWORD:  test123!
      PGDATA: /var/lib/postgresql/data/pgdata
    restart:  always
volumes:
  pg_data:

```

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

---
### Ответ
- итоговый список БД после выполнения пунктов выше:

![Screenshot2_1](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot2_1.png)

- описание таблиц (describe):

![Screenshot2_2](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot2_2.png)

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db:

```sql
select t.grantee as User_name, t.table_name, t.privilege_type
from information_schema.table_privileges t
where t.grantee!='postgres' and t.grantee!='PUBLIC';
```

- список пользователей с правами над таблицами test_db:

![Screenshot2_3](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot2_3.png)

---

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.

---
### Ответ
1. 
```sql
insert into orders (name, price) values ('Шоколад', 10),
('Принтеp', 3000),
('Книга', 500),
('Монитор', 7000),
('Гитара', 4000)
;
select * from orders;
```

![Screenshot3_1](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot3_1.png)

2. 
```sql
insert into clients (last_name, country) values
('Иванов Иван Иванович', 'USA'),
('Петров Петр Петрович', 'Canada'),
('Иоганн Себастьян Бах', 'Japan'),
('Ронни Джеймс Дио', 'Russia'),
('Ritchie Blackmore', 'Russia')
;
select * from clients;
```

![Screenshot3_2](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot3_2.png)

3.
```sql
select count(orders.id) кол_во_записей_таблицы_заказов
from orders
;
select count(clients.id) кол_во_записей_таблицы_клиентов
from clients
;
```

![Screenshot3_3](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot3_3.png)

---


## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

---
### Ответ

1. SQL-запросы:
```sql
UPDATE clients SET orderid = (
	SELECT id 
	FROM orders o
	WHERE o.name='Книга'
)
WHERE last_name ='Иванов Иван Иванович'
;
UPDATE clients SET orderid = (SELECT id	FROM orders o WHERE o.name='Монитор') WHERE last_name ='Петров Петр Петрович';
UPDATE clients SET orderid = (SELECT id	FROM orders o WHERE o.name='Гитара') WHERE last_name ='Иоганн Себастьян Бах';
```

2. SQL-запрос для выдачи всех пользователей, которые совершили заказ:

```sql
select c.last_name as пользователи_совершили_заказ, o.name as наименование_заказа
from clients c
join orders o on o.id=c.orderid;
```

вывод этого запроса:

![Screenshot4_1](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot4_1.png)

---

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

---
### Ответ

![Screenshot5_1](https://github.com/megasts/06-db-02-sql/blob/main/img/Screenshot5_1.png)

Полученные данные представляют собой план запроса: как postgresql планирует выпонять запрос (не изменяетсяя от запуска к запуску), корме того можно оценить "стоимость" запроса в целом и каждой операции в отдельности.
1. Построчно прочитана таблица orders (Seq Scan on orders)
2. Построена Hash таблица данных из таблицы orders (Hash)
3. Построчно прочитана таблица clients (Seq Scan on clients)
4. Построена итоговая таблица на основе Hash соединения двух таблиц с провекой условия Hash Cond (Hash Cond: (c.orderid = o.id))

---


## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---
### Ответ

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

```
# pg_dumpall -U postgres > /backups/test_db_backup.sql
```

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

```
$ docker stop postgreSQL
```

Поднимите новый пустой контейнер с PostgreSQL.

docker-compose-манифест:
```
version: '3'
services:
  postgreSQL:
    image:  postgres:12
    container_name: pg_restor
    ports:
      - "5432:5432"
    volumes:
    - /home/megion/Projects/sql/backups:/backups
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD:  test123!
    restart:  always
```

```
$ docker compose -f "/home/megion/Projects/sql_from_backup/docker-compose.yml" up -d --buil
```

Восстановите БД test_db в новом контейнере.

```
$ docker exec -it pg_restor bash

# psql -U postgres < /backups/test_db_backup.sql
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

