- Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
- Установить на него PostgreSQL 15 с дефолтными настройками
```shell
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
```

- Создать БД для тестов: выполнить pgbench -i postgres
```shell
focus@hw5-otus:~$ pgbench -i postgres
pgbench: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "focus" does not exist
pgbench: error: could not create connection for initialization
focus@hw5-otus:~$ sudo su postgres
postgres@hw5-otus:/home/focus$ pgbench -i postgres
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.20 s (drop tables 0.01 s, create tables 0.01 s, client-side generate 0.10 s, vacuum 0.03 s, primary keys 0.06 s).
postgres@hw5-otus:/home/focus$ 

```
- Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```shell
postgres@hw5-otus:/home/focus$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 545.2 tps, lat 14.622 ms stddev 15.432, 0 failed
progress: 12.0 s, 729.5 tps, lat 10.963 ms stddev 36.765, 0 failed
progress: 18.0 s, 762.7 tps, lat 10.475 ms stddev 7.538, 0 failed
progress: 24.0 s, 779.2 tps, lat 10.267 ms stddev 6.332, 0 failed
progress: 30.0 s, 775.5 tps, lat 10.327 ms stddev 6.425, 0 failed
progress: 36.0 s, 666.2 tps, lat 12.010 ms stddev 13.281, 0 failed
progress: 42.0 s, 653.7 tps, lat 12.237 ms stddev 49.494, 0 failed
progress: 48.0 s, 773.2 tps, lat 10.352 ms stddev 7.493, 0 failed
progress: 54.0 s, 765.0 tps, lat 10.453 ms stddev 6.444, 0 failed
progress: 60.0 s, 773.2 tps, lat 10.348 ms stddev 6.199, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 43347
number of failed transactions: 0 (0.000%)
latency average = 11.072 ms
latency stddev = 20.570 ms
initial connection time = 15.755 ms
tps = 722.398108 (without initial connection time) 
```
- Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
```sql
postgres=# ALTER SYSTEM SET max_connections = 40;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET shared_buffers = '1GB';
ALTER SYSTEM
postgres=# 
postgres=# ALTER SYSTEM SET effective_cache_size='3GB';
ALTER SYSTEM
postgres=# 
postgres=# 
postgres=# ALTER SYSTEM SET maintenance_work_mem = 512MB;
ERROR:  trailing junk after numeric literal at or near "512M"
LINE 1: ALTER SYSTEM SET maintenance_work_mem = 512MB;
                                                ^
postgres=# ALTER SYSTEM SET maintenance_work_mem = '512MB';
ALTER SYSTEM
postgres=# ALTER SYSTEM SET checkpoint_completion_target = 0.9;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM
postgres=# ALTER SYSTEM SET default_statistics_target = 500;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET random_page_cost = 4;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET effective_io_concurrency = 2;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET work_mem = '6553kB';
ALTER SYSTEM
postgres=# ALTER SYSTEM SET min_wal_size = '4GB';
ALTER SYSTEM
postgres=# ALTER SYSTEM SET max_wal_size = '16GB';

```
- Протестировать заново
```shell
postgres@hw5-otus:/home/focus$ pgbench -c8 -P 6 -T 60 -U postgres postgres 
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 592.8 tps, lat 13.388 ms stddev 13.554, 0 failed
progress: 12.0 s, 860.0 tps, lat 9.341 ms stddev 7.061, 0 failed
progress: 18.0 s, 762.7 tps, lat 10.491 ms stddev 6.735, 0 failed
progress: 24.0 s, 770.8 tps, lat 10.376 ms stddev 6.473, 0 failed
progress: 30.0 s, 772.3 tps, lat 10.356 ms stddev 6.562, 0 failed
progress: 36.0 s, 593.2 tps, lat 13.472 ms stddev 11.982, 0 failed
progress: 42.0 s, 864.3 tps, lat 9.263 ms stddev 6.383, 0 failed
progress: 48.0 s, 772.7 tps, lat 10.355 ms stddev 6.159, 0 failed
progress: 54.0 s, 773.5 tps, lat 10.341 ms stddev 7.488, 0 failed
progress: 60.0 s, 764.2 tps, lat 10.468 ms stddev 6.763, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 45167
number of failed transactions: 0 (0.000%)
latency average = 10.626 ms
latency stddev = 8.090 ms
initial connection time = 16.310 ms
tps = 752.694578 (without initial connection time)
```
- Что изменилось и почему?
Кажется, почти ничего не изменилось. Разве что tps.

- Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
```shell
postgres=# CREATE TABLE random_strings(s text);
CREATE TABLE
postgres=# CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION
postgres=# INSERT INTO random_strings (s)
SELECT md5(gen_random_uuid()::text)
FROM generate_series(0, 1000000);
INSERT 0 1000000
INSERT 0 1000001
```
- Посмотреть размер файла с таблицей
```shell
postgres=# SELECT oid FROM pg_class WHERE relname = 'random_strings';
  oid  
-------
 16480
(1 row)

postgres=# SELECT pg_relation_filepath('16480');
 pg_relation_filepath 
----------------------
 base/5/16480
focus@hw5-otus:~$ sudo ls -lh /var/lib/postgresql/15/main/base/5/16480
-rw------- 1 postgres postgres 66M Mar 13 19:31 /var/lib/postgresql/15/main/base/5/16480

```
66M

- 5 раз обновить все строчки и добавить к каждой строчке любой символ
```sql
postgres=# UPDATE random_strings
SET s = md5(random()::text);
UPDATE 1000001
postgres=# UPDATE random_strings
SET s = md5(random()::text);
UPDATE 1000001
postgres=# UPDATE random_strings
SET s = md5(random()::text);
UPDATE 1000001
postgres=# UPDATE random_strings
SET s = md5(random()::text);
UPDATE 1000001
postgres=# UPDATE random_strings
SET s = md5(random()::text);
UPDATE 1000001
postgres=# UPDATE random_strings
SET s = s || substr(md5(random()::text), 1, 1);
UPDATE 1000001
```
- Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```sql
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum 
FROM pg_stat_user_TABLEs 
WHERE relname = 'random_strings';
    relname     | n_live_tup | n_dead_tup | ratio% |       last_autovacuum        
----------------+------------+------------+--------+------------------------------
 random_strings |    1000001 |     999881 |     99 | 2024-03-13 19:40:35.43591+00
(1 row)
```
last_autovacuum - пару минут назад, n_dead_tup=999881

- Подождать некоторое время, проверяя, пришел ли автовакуум
```sql
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum 
FROM pg_stat_user_TABLEs 
WHERE relname = 'random_strings';
    relname     | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
----------------+------------+------------+--------+-------------------------------
 random_strings |    1000001 |          0 |      0 | 2024-03-13 19:41:32.560696+00
(1 row)
```
- Посмотреть размер файла с таблицей
```shell
focus@hw5-otus:~$ sudo ls -lh /var/lib/postgresql/15/main/base/5/16480
-rw------- 1 postgres postgres 196M Mar 13 19:43 /var/lib/postgresql/15/main/base/5/16480 
```
- Отключить Автовакуум на конкретной таблице
```psql
postgres=# ALTER TABLE random_strings SET (autovacuum_enabled = false);
ALTER TABLE 
```
- 10 раз обновить все строчки и добавить к каждой строчке любой символ
```sql
postgres=# UPDATE random_strings
           SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = md5(random()::text);
UPDATE 1000001
    postgres=# UPDATE random_strings
               SET s = s || substr(md5(random()::text), 1, 1);
UPDATE 1000001 
```
- Посмотреть размер файла с таблицей
```shell
focus@hw5-otus:~$ sudo ls -lh /var/lib/postgresql/15/main/base/5/16480
-rw------- 1 postgres postgres 782M Mar 13 19:53 /var/lib/postgresql/15/main/base/5/16480 
```
- Объясните полученный результат
Мертвые строки. Чтобы уменьшить размер к исходному нужен
```sql
postgres=# vacuum full analyze random_strings ;
VACUUM
postgres=# SELECT oid FROM pg_class WHERE relname = 'random_strings';
oid
-------
16480
(1 row)

postgres=# SELECT pg_relation_filepath('16480');
pg_relation_filepath
----------------------
base/5/16527
(1 row)

postgres=#
\q
postgres@hw5-otus:/home/focus$ 
exit
focus@hw5-otus:~$ sudo ls -lh /var/lib/postgresql/15/main/base/5/16480
-rw------- 1 postgres postgres 0 Mar 13 19:56 /var/lib/postgresql/15/main/base/5/16480
focus@hw5-otus:~$ sudo ls -lh /var/lib/postgresql/15/main/base/5/16527
-rw------- 1 postgres postgres 66M Mar 13 19:56 /var/lib/postgresql/15/main/base/5/16527
```