- посмотреть текущий уровень изоляции: show transaction isolation level
- начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
- в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
- сделать select from persons во второй сессии

Q: Видите ли вы новую запись и если да то почему?

A: Нет, новая запись не будет видна, т.к. мы делаем вставку и не фиксируем данные т.е. транзакция из второй сессии ничего не знает про незакомиченные изменения из первой.
Изоляция:
```sql
postgres=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 read committed
(1 row)
```
Вывод из второй сессии:
```sql
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```

- завершить первую транзакцию - commit;
- сделать select from persons во второй сессии

Q: Видите ли вы новую запись и если да то почему?
A: Да, т.к. данные закомичены.

- завершите транзакцию во второй сессии
- начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
- в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
- сделать select* from persons во второй сессии*

Q: Видите ли вы новую запись и если да то почему?
A: Нет, т.к. транзакция не закомичена.

```sql
postgres=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```

- завершить первую транзакцию - commit;
- сделать select from persons во второй сессии
- видите ли вы новую запись и если да то почему?
Нет, т.к. на этом уровне изоляции работаем со снимком, сделанным до начала транзакции.
- завершить вторую транзакцию
- сделать select * from persons во второй сессии
```sql
postgres=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(6 rows)
```

Q: видите ли вы новую запись и если да то почему?
A: Да, траназкции закомичены, изменения видны.