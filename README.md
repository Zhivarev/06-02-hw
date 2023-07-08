# Домашнее задание к занятию "`SQL`" - `Живарев Игорь`


### Задание 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.


Ответ:

docker-compose.yaml
```
version: '3'
services:
  postgres:
    container_name: postgres
    image: postgres:12
    environment:
      POSTGRES_USER: psql
      POSTGRES_PASSWORD: psql
      POSTGRES_DB: netology_db
    ports:
      - "5432:5432"
    volumes:
      - database_volume:/opt/database/
      - backup_volume:/opt/backup/
    restart: always

volumes:
  database_volume:
  backup_volume:
```

---

### Задание 2

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


Ответ:

Создаём базу test_db и пользователя test-admin-user:

```
root@d76958094306:/# createdb test_db -U psql
root@d76958094306:/# psql -d test_db -U psql
psql (12.15 (Debian 12.15-1.pgdg120+1))
Type "help" for help.

test_db=# CREATE USER test_admin_user;
CREATE ROLE
test_db=#
```

Создание таблиц orders и clients:

```
test_db=# CREATE TABLE orders
(
   id SERIAL PRIMARY KEY,
   наименование TEXT,
   цена INTEGER
);
CREATE TABLE
test_db=# CREATE TABLE clients
(
    id SERIAL PRIMARY KEY,
    фамилия TEXT,
    "страна проживания" TEXT,
    заказ INTEGER,
    FOREIGN KEY (заказ) REFERENCES orders(id)
);
CREATE TABLE
```

Предоставление привилегий пользователю test-admin-user:

```
test_db=# GRANT ALL ON TABLE orders TO test_admin_user;
GRANT
test_db=# GRANT ALL ON TABLE clients TO test_admin_user;
GRANT
```

Создание пользователя test-simple-user и предоставление ему требуемых прав:

```
test_db=# CREATE USER test_simple_user;
CREATE ROLE
test_db=# GRANT SELECT,INSERT,UPDATE,DELETE ON TABLE orders, clients TO test_simple_user;
GRANT
test_db=#
```

Итоговый список БД:

```
test_db-# \l+
                                                                List of databases
    Name     | Owner | Encoding |  Collate   |   Ctype    | Access privileges |  Size   | Tablespace |                Description
-------------+-------+----------+------------+------------+-------------------+---------+------------+--------------------------------------------
 netology_db | psql  | UTF8     | en_US.utf8 | en_US.utf8 |                   | 7833 kB | pg_default |
 postgres    | psql  | UTF8     | en_US.utf8 | en_US.utf8 |                   | 7977 kB | pg_default | default administrative connection database
 template0   | psql  | UTF8     | en_US.utf8 | en_US.utf8 | =c/psql          +| 7833 kB | pg_default | unmodifiable empty database
             |       |          |            |            | psql=CTc/psql     |         |            |
 template1   | psql  | UTF8     | en_US.utf8 | en_US.utf8 | =c/psql          +| 7833 kB | pg_default | default template for new databases
             |       |          |            |            | psql=CTc/psql     |         |            |
 test_db     | psql  | UTF8     | en_US.utf8 | en_US.utf8 |                   | 8121 kB | pg_default |
(5 rows)

```

Описание таблиц (describe):

```
test_db-# \d+ clients                                                                                                                            
                                                      Table "public.clients"                                                                     
      Column       |  Type   | Collation | Nullable |               Default               | Storage  | Stats target | Description                
-------------------+---------+-----------+----------+-------------------------------------+----------+--------------+-------------               
 id                | integer |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |                            
 фамилия           | text    |           |          |                                     | extended |              |                            
 страна проживания | text    |           |          |                                     | extended |              |                            
 заказ             | integer |           |          |                                     | plain    |              |                            
Indexes:                                                                                                                                         
    "clients_pkey" PRIMARY KEY, btree (id)                                                                                                       
Foreign-key constraints:                                                                                                                         
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)                                                                             
Access method: heap                                                                                                                              
                                                                                                                                                 
test_db-# \d+ orders                                                                                                                             
                                                   Table "public.orders"                                                                         
    Column    |  Type   | Collation | Nullable |              Default               | Storage  | Stats target | Description                      
--------------+---------+-----------+----------+------------------------------------+----------+--------------+-------------                     
 id           | integer |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |                                  
 наименование | text    |           |          |                                    | extended |              |                                  
 цена         | integer |           |          |                                    | plain    |              |                                  
Indexes:                                                                                                                                         
    "orders_pkey" PRIMARY KEY, btree (id)                                                                                                        
Referenced by:                                                                                                                                   
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)                                                  
Access method: heap                                                                                                                              
                                                                                                                                                 
```

SQL-запрос для выдачи списка пользователей с правами над таблицами test_db:

```
SELECT grantee, table_catalog, table_name, privilege_type 
FROM information_schema.table_privileges 
WHERE table_name IN ('orders','clients');

```

Список пользователей с правами над таблицами test_db:

```
test_db=# SELECT grantee, table_catalog, table_name, privilege_type FROM information_schema.table_privileges WHERE table_name IN ('orders','clients');
     grantee      | table_catalog | table_name | privilege_type
------------------+---------------+------------+----------------
 psql             | test_db       | orders     | INSERT
 psql             | test_db       | orders     | SELECT
 psql             | test_db       | orders     | UPDATE
 psql             | test_db       | orders     | DELETE
 psql             | test_db       | orders     | TRUNCATE
 psql             | test_db       | orders     | REFERENCES
 psql             | test_db       | orders     | TRIGGER
 test_admin_user  | test_db       | orders     | INSERT
 test_admin_user  | test_db       | orders     | SELECT
 test_admin_user  | test_db       | orders     | UPDATE
 test_admin_user  | test_db       | orders     | DELETE
 test_admin_user  | test_db       | orders     | TRUNCATE
 test_admin_user  | test_db       | orders     | REFERENCES
 test_admin_user  | test_db       | orders     | TRIGGER
 test_simple_user | test_db       | orders     | INSERT
 test_simple_user | test_db       | orders     | SELECT
 test_simple_user | test_db       | orders     | UPDATE
 test_simple_user | test_db       | orders     | DELETE
 psql             | test_db       | clients    | INSERT
 psql             | test_db       | clients    | SELECT
 psql             | test_db       | clients    | UPDATE
 psql             | test_db       | clients    | DELETE
 psql             | test_db       | clients    | TRUNCATE
 psql             | test_db       | clients    | REFERENCES
 psql             | test_db       | clients    | TRIGGER
 test_admin_user  | test_db       | clients    | INSERT
 test_admin_user  | test_db       | clients    | SELECT
 test_admin_user  | test_db       | clients    | UPDATE
 test_admin_user  | test_db       | clients    | DELETE
 test_admin_user  | test_db       | clients    | TRUNCATE
 test_admin_user  | test_db       | clients    | REFERENCES
 test_admin_user  | test_db       | clients    | TRIGGER
 test_simple_user | test_db       | clients    | INSERT
 test_simple_user | test_db       | clients    | SELECT
 test_simple_user | test_db       | clients    | UPDATE
 test_simple_user | test_db       | clients    | DELETE
(36 rows)
                                                                                                                                
```

---

### Задание 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование	|цена|
|:--------------|----|
|Шоколад	|10|
|Принтер	|3000|
|Книга	|500|
|Монитор	|7000|
|Гитара	|4000|

Таблица clients

|ФИО |Страна проживания|
|:---------------------|----|
|Иванов Иван Иванович	|USA|
|Петров Петр Петрович	|Canada|
|Иоганн Себастьян Бах	|Japan|
|Ронни Джеймс Дио	|Russia|
|Ritchie Blackmore	|Russia|
Используя SQL-синтаксис:

вычислите количество записей для каждой таблицы.
Приведите в ответе:
```
- запросы,
- результаты их выполнения.
```


Ответ:

Наполнение таблиц:

```
test_db=# INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
test_db=# INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
INSERT 0 5
```

Количество записей для каждой таблицы:

```
test_db=# SELECT COUNT (*) FROM orders;
 count
-------
     5
(1 row)

test_db=# SELECT COUNT (*) FROM clients;
 count
-------
     5
(1 row)
```

### Задание 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО	|Заказ|
|:------|-----|
|Иванов Иван Иванович	|Книга|
|Петров Петр Петрович	|Монитор|
|Иоганн Себастьян Бах	|Гитара|
Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.

Подсказка: используйте директиву `UPDATE`.


Ответ: 

```
test_db=# UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE "наименование"='Книга') WHERE "фамилия"='Иванов Иван Иванович';
UPDATE 1
test_db=# UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE "наименование"='Монитор') WHERE "фамилия"='Петров Петр Петрович';
UPDATE 1
test_db=# UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE "наименование"='Гитара') WHERE "фамилия"='Иоганн Себастьян Бах';
UPDATE 1
test_db=# SELECT * FROM clients WHERE заказ IS NOT NULL;
 id |       фамилия        | страна проживания | заказ
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)
```

---

### Задание 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.


Ответ:


```
test_db=# SELECT * FROM clients WHERE заказ IS NOT NULL;
 id |       фамилия        | страна проживания | заказ
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)

test_db=# EXPLAIN SELECT * FROM clients WHERE "заказ" IS NOT NULL;
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72)
   Filter: ("заказ" IS NOT NULL)
(2 rows)
```

Чтение данных из таблицы clients происходит с использованием метода Seq Scan — последовательного чтения данных. Значение 0.00 — ожидаемые затраты на получение первой строки. Второе — 18.10 — ожидаемые затраты на получение всех строк. rows - ожидаемое число строк, которое должен вывести этот узел плана. При этом так же предполагается, что узел выполняется до конца. width - ожидаемый средний размер строк, выводимых этим узлом плана (в байтах). Каждая запись сравнивается с условием "заказ" IS NOT NULL. Если условие выполняется, запись вводится в результат. Иначе — отбрасывается.


```
test_db=# EXPLAIN (ANALYZE) SELECT * FROM clients WHERE "заказ" IS NOT NULL;
                                             QUERY PLAN
-----------------------------------------------------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72) (actual time=0.023..0.025 rows=3 loops=1)
   Filter: ("заказ" IS NOT NULL)
   Rows Removed by Filter: 2
 Planning Time: 0.076 ms
 Execution Time: 0.046 ms
(5 rows)
```

Здесь уже видны реальные затраты на обработку первой и всех строк, количество выведенных строк (3), удовлетворяющих фильру "заказ" IS NOT NULL, количество проходов (1), количество строк, которые были удалены из запроса по фильтру (2), планируемое и затраченное время, а также общее количество строк, по которым производилась выборка.

---

### Задание 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.


Ответ:
 

```
root@vm1:~/sql# docker run --rm -d -e POSTGRES_USER=test-admin-user -e POSTGRES_PASSWORD=psql -e POSTGRES_DB=test_db -v backup_volume:/opt/backup/ --nam e postgres_new postgres:12
82b73d8a55a22b7b70dedb40f4fa66f1165f1280389da9f95a53d9fc2eaa755c
root@vm1:~/sql# docker exec -it postgres_new bash
root@82b73d8a55a2:/# ls /opt/backup/
test_db.backup
root@82b73d8a55a2:/# export PGPASSWORD=psql && psql -h localhost -U test-admin-user -f /opt/backup/test_db.backup test_db         7 months ago    Exited (0) 7 months ago                    goofy_mclaren
root@82b73d8a55a2:/# psql -h localhost -U test-admin-user test_db
psql (12.15 (Debian 12.15-1.pgdg120+1))
Type "help" for help.

test_db=#
```

---
