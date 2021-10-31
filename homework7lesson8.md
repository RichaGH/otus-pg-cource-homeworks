# Домашнее задание
## Механизм блокировок

### Цель:
- понимать как работает механизм блокировок объектов и строк

### Задание и ход выполнения по этапам: 

#### 1. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
##### Результат:
Меняем две настройки log_lock_waits и deadlock_timeout:
- Вариант 1 из psql:
```
postgres=# ALTER SYSTEM SET log_lock_waits=on;
ALTER SYSTEM
postgres=# ALTER SYSTEM SET deadlock_timeout=200;
ALTER SYSTEM
```
- Вариант 2 напрямую в файле postgresql.conf:
```
postgres=# show config_file;
               config_file
-----------------------------------------
 /etc/postgresql/14/main/postgresql.conf
(1 row)
postgres=# \! vim /etc/postgresql/14/main/postgresql.conf
```
находим и изменяем строки (раскомментировать и задать значения on  и 200ms соответственно)
```
log_lock_waits = on                     # log lock waits >= deadlock_timeout
deadlock_timeout = 200ms

```
В обоих случаях не забываем обновить(применить) конфигурацию достаточно перезапуск сервиса не нужен:
```
postgres=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show deadlock_timeout ;
 deadlock_timeout
------------------
 200ms
(1 row)

postgres=# SHOW log_lock_waits;
 log_lock_waits
----------------
 on
(1 row)

```
Организуем ожидание транзакции так:
 - В первой сессии создадим базу, таблицу, внесем в таблицу несколько строк
```
postgres=# create database le8;
CREATE DATABASE
postgres=# \c le8
You are now connected to database "le8" as user "postgres".
le8=# create table t1 (id integer, num numeric);
CREATE TABLE
le8=# \d t1
                 Table "public.t1"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           |          |
 num    | numeric |           |          |

le8=# insert into t1 values (1,12.0);
INSERT 0 1
le8=# insert into t1 values (2,16.0);
INSERT 0 1
le8=# insert into t1 values (2,160);
INSERT 0 1

```
 и начнем транзакцию изменяющую данные в табице:
```
le8=# Begin;
BEGIN
le8=*# insert into t1 values (4,480);
INSERT 0 1
le8=*# update t1 set id=3 where id=2 and num=160;
UPDATE 1

```
- (Затем) Во второй сессии начнем транзакцию, которая должна обновить данные в таблице:
```
postgres=# \c le8
You are now connected to database "le8" as user "postgres".
le8=# begin;
BEGIN
le8=*# update t1 set num=100;

```
видим что второй апдейт что-то долго висит (ну это понятно он ждет своей очереди)
Заглянем в лог (200 мсек не долго там должно что-то появиться)
```
postgres@ub20-postgres-8:~$ tail -n 10 /var/log/postgresql/postgresql-14-main.log
...
2021-10-30 19:30:44.157 UTC [4854] postgres@le8 LOG:  process 4854 still waiting for ShareLock on transaction 738 after 200.154 ms
2021-10-30 19:30:44.157 UTC [4854] postgres@le8 DETAIL:  Process holding the lock: 4707. Wait queue: 4854.
2021-10-30 19:30:44.157 UTC [4854] postgres@le8 CONTEXT:  while updating tuple (0,3) in relation "t1"
2021-10-30 19:30:44.157 UTC [4854] postgres@le8 STATEMENT:  update t1 set num=100;

```

#### 2. Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
##### Результат:
Для удобства работы подключим расширение и создадим пару вьюшек для нашей таблицы и посмотрим её текущее содержимое:
```
le8=# CREATE EXTENSION pageinspect;
CREATE EXTENSION

le8=# CREATE VIEW t1_v AS
SELECT '(0,'||lp||')' AS ctid,
       t_xmax as xmax,
       CASE WHEN (t_infomask & 128) > 0   THEN 't' END AS lock_only,
       CASE WHEN (t_infomask & 4096) > 0  THEN 't' END AS is_multi,
       CASE WHEN (t_infomask2 & 8192) > 0 THEN 't' END AS keys_upd,
       CASE WHEN (t_infomask & 16) > 0 THEN 't' END AS keyshr_lock,
       CASE WHEN (t_infomask & 16+64) = 16+64 THEN 't' END AS shr_lock
FROM heap_page_items(get_raw_page('t1',0))
ORDER BY lp;
CREATE VIEW


le8=# CREATE VIEW locks_v AS
SELECT pid,
       locktype,
       CASE locktype
         WHEN 'relation' THEN relation::regclass::text
         WHEN 'transactionid' THEN transactionid::text
         WHEN 'tuple' THEN relation::regclass::text||':'||tuple::text
       END AS lockid,
       mode,
       granted
FROM pg_locks
WHERE locktype in ('relation','transactionid','tuple')
AND (locktype != 'relation' OR relation = 't1'::regclass);
CREATE VIEW

le8=# select * from t1;
 id | num
----+-----
  2 | 100
  2 | 100
  1 |   3
(3 rows)

```

- теперь в сеансе 1:
```
le8=# begin;
BEGIN
le8=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          749 |           5550
(1 row)

le8=*# update t1 set num=33 where id=1;
UPDATE 1
le8=*#
```

- теперь в сеансе 2:
```
le8=# begin;
BEGIN
le8=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          750 |           5559
(1 row)

le8=*# update t1 set num=333 where id=1;


```

- теперь в сеансе 3:
```
le8=# begin;
BEGIN
le8=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          751 |           5568
(1 row)

le8=*# update t1 set num=21 where id=1;


```

- Вернемся в первый сеанс и посмотрим что там у нас происходит:
```
le8=*# SELECT pid,locktype, relation::REGCLASS, virtualxid AS virtxid, transacti    onid AS xid, mode, granted
le8-*# FROM pg_locks where pid in (5550,5559,5568)order by pid;
 pid  |   locktype    | relation | virtxid | xid |       mode       | granted   номер строки   
------+---------------+----------+---------+-----+------------------+---------|                
 5550 | virtualxid    |          | 4/105   |     | ExclusiveLock    | t       | 1 
 5550 | relation      | pg_locks |         |     | AccessShareLock  | t       | 2
 5550 | transactionid |          |         | 749 | ExclusiveLock    | t       | 3              
 5550 | relation      | t1       |         |     | RowExclusiveLock | t       | 4
 5559 | transactionid |          |         | 750 | ExclusiveLock    | t       | 5 
 5559 | relation      | t1       |         |     | RowExclusiveLock | t       | 6
 5559 | virtualxid    |          | 5/9     |     | ExclusiveLock    | t       | 7 
 5559 | transactionid |          |         | 749 | ShareLock        | f       | 8              
 5559 | tuple         | t1       |         |     | ExclusiveLock    | t       | 9
 5568 | relation      | t1       |         |     | RowExclusiveLock | t       | 10
 5568 | tuple         | t1       |         |     | ExclusiveLock    | f       | 11
 5568 | transactionid |          |         | 751 | ExclusiveLock    | t       | 12
 5568 | virtualxid    |          | 6/12    |     | ExclusiveLock    | t       | 13
(13 rows)
```
- Здесь transactionid в режиме ExclusiveLock (строки 3,5,12) это блокировки собственного номера транзакции, необходимы для понимания того, что транзакция еще не завершена и более того совершает манипуляции с данными (тип transactionid)
- Здесь строки 1,7,13 говорят о том что Транзакции начатые процессами из столбца Pid Еще не завершены и имеют виртуальные идентификаторы указанные в столбце virtxid

На протяжении транзакции серверный процесс удерживает исключительную блокировку виртуального идентификатора транзакции. Если транзакции назначается постоянный идентификатор (что обычно происходит, только если транзакция изменяет состояние базы данных), он также удерживает до её завершения блокировку этого постоянного идентификатора. Когда процесс находит необходимым ожидать именно какую-то другую транзакцию, он делает это, запрашивая разделяемую блокировку для идентификатора этой транзакции (виртуального или постоянного, в зависимости от ситуации). Этот запрос будет выполнен, только когда другая транзакция завершится и освободит свои блокировки.

https://postgrespro.ru/docs/postgrespro/13/view-pg-locks

- Здесь строка 8 говорит о том что (transactionid в режиме ShareLock granted=false) говорит о том, что процесс 5559 ожидает пока завершится транзакция 749 чтобы получить блокировку, объекта в настоящее время удерживаемую этой транзакцией
- Здесь строка 2 говорит о том что мы обратились к отношению pg_locks и получили (granted=true) блокировку в режиме AccessShareLock из процесса 5550

- Здесь строки 4,6,10 говорят о том что Транзакции начатые процессами из столбца Pid собираются манипулировать данными отношения (relation) в режиме RowExclusiveLock (изменять какие-то строки) делать это они будут именно отношении t1 
- Здесь строка 9 говорит о том что процесс 5559 при попытке изменения строки в t1 обнаружил что на уровне таблицы нужная ему строка заблокирована, он был первым процессом который встал в очередь на изменение данного кортежа и поэтому ему удалось получить блокировку (granted=true) типа tuple на таблице t1
- Здесь строка 11 говорит о том что процесс 5568 при попытке изменения строки в t1 обнаружил что на уровне таблицы нужная ему строка заблокирована, он не был первым процессом который встал в очередь на изменение данного кортежа и поэтому ему не удалось получить блокировку (granted=false) типа tuple на таблице t1 И он будет ждать пока не освободится блокировка кортежа

Посмотрим что там в t1 :
```
le8=*# select * from t1_v;
  ctid  | xmax | lock_only | is_multi | keys_upd | keyshr_lock | shr_lock
--------+------+-----------+----------+----------+-------------+----------
 (0,1)  |  739 |           |          |          |             |
 (0,2)  |  739 |           |          |          |             |
 (0,3)  |  739 |           |          |          |             |
 (0,4)  |    0 |           |          |          |             |
 (0,5)  |    0 |           |          |          |             |
 (0,6)  |  744 |           |          |          |             |
 (0,7)  |    0 |           |          |          |             |
 (0,8)  |    0 |           |          |          |             |
 (0,9)  |  746 |           |          |          |             |
 (0,10) |    0 |           |          |          |             |
 (0,11) |  747 |           |          |          |             |
 (0,12) |  745 |           |          |          |             |
 (0,13) |  749 |           |          |          |             |
 (0,14) |    0 |           |          |          |             |
(14 rows)
```
- здесь мы видно что 13й кортеж "задет" транзакцией 749, больше наших тут нет, мультитранзакций тоже нет

Посмотрим нашу вторую вьюшку:
```
le8=*# select * from locks_v ;
 pid  |   locktype    | lockid |       mode       | granted   номер строки
------+---------------+--------+------------------+---------|             
 5550 | relation      | t1     | RowExclusiveLock | t       | 1           
 5559 | relation      | t1     | RowExclusiveLock | t       | 2           
 5568 | relation      | t1     | RowExclusiveLock | t       | 3           
 5568 | transactionid | 751    | ExclusiveLock    | t       | 4           
 5559 | transactionid | 749    | ShareLock        | f       | 5           
 5559 | tuple         | t1:13  | ExclusiveLock    | t       | 6           
 5568 | tuple         | t1:13  | ExclusiveLock    | f       | 7           
 5550 | transactionid | 749    | ExclusiveLock    | t       | 8           
 5559 | transactionid | 750    | ExclusiveLock    | t       | 9           
(9 rows)                                                    


```
- Первые 3 строки говорят о том что наши процессы в своих транзакциях получили(granted=true) блокировку в режиме RowExclusiveLock на relation t1
- строка 4 говорит о том что процесс 5568 получил и удерживает (granted=true) в режиме ExclusiveLock свой номер транзакции для манипуляции с данными (751)
- строка 5 говорит о том что процесс 5559 НЕ получил (granted=false) в режиме ShareLock номер транзакции для манипуляции с данными (749) пытался он это сделать, чтобы понять не завершила ли еще транзакция 749 свою работу, теперь процесс ожидает её завершения
- строка 6 говорит о том что процесс 5559 получил и удерживает (granted=true) в режиме ExclusiveLock блокировку строки (номер строки 13), он первый в очереди за транзакцией которая блокирует строку и приступит к работе как только транзакция удерживающая блокировку строки снимет её и "сообщит" об этом другим через снятие своих блокировок
- строка 7 говорит о том что процесс 5568 НЕ получил (granted=false) в режиме ExclusiveLock блокировку строки (номер строки 13) это говорит о том что кто-то уже стоит в очереди заэтой строкой и процессу остается ждать освобождения блокировки кортежа чтобы получить её в следующий раз если повезет
- строка 8 говорит о том что процесс 5550 получил и удерживает (granted=true) в режиме ExclusiveLock свой номер транзакции для манипуляции с данными (749)
- строка 9 говорит о том что процесс 5559 получил и удерживает (granted=true) в режиме ExclusiveLock свой номер транзакции для манипуляции с данными (750)
 
Также можем посмотреть какие процессы какими заблокированы:
```
le8=*# select pg_blocking_pids(5568);
 pg_blocking_pids
------------------
 {5559}
(1 row)

le8=*# select pg_blocking_pids(5559);
 pg_blocking_pids
------------------
 {5550}
(1 row)

le8=*# select pg_blocking_pids(5550);
 pg_blocking_pids
------------------
 {}
(1 row)

```

#### 3. Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
##### Результат:

- сессия 1 (шаг 1):
```
le8=# begin;
BEGIN
le8=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          755 |           5550
(1 row)

le8=*# update t1 set num=33 where id=1;
UPDATE 1

```

- сессия 2 (шаг 1):
```
le8=# begin;
BEGIN
le8=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          756 |           5559
(1 row)

le8=*# update t1 set num=22 where id=3;
UPDATE 1


```

- сессия 3 (шаг 1):
```
le8=# begin;
BEGIN
le8=*# SELECT txid_current(), pg_backend_pid();
 txid_current | pg_backend_pid
--------------+----------------
          757 |           5568
(1 row)

le8=*# update t1 set num=66 where id=8;
UPDATE 1
```

- сессия 1 (шаг 2):
```

le8=*# update t1 set num=33 where id=8;
UPDATE 1
le8=*#
```

- сессия 2 (шаг 2):
```
le8=*# update t1 set num=22 where id=1;

```
- сессия 3 (шаг 2):
```
le8=*# update t1 set num=66 where id=3;
ERROR:  deadlock detected
DETAIL:  Process 5568 waits for ShareLock on transaction 756; blocked by process 5559.
Process 5559 waits for ShareLock on transaction 755; blocked by process 5550.
Process 5550 waits for ShareLock on transaction 757; blocked by process 5568.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,17) in relation "t1"
le8=!#
```

- Можно ли понять что-то по логам?
Посмотрим:
```
le8=!# \! tail -n 30 /var/log/postgresql/postgresql-14-main.log
2021-10-30 22:16:25.649 UTC [5568] postgres@le8 CONTEXT:  while rechecking updated tuple (0,14) in relation "t1"
2021-10-30 22:16:25.649 UTC [5568] postgres@le8 STATEMENT:  update t1 set num=21 where id=1;
2021-10-30 22:18:36.184 UTC [5568] postgres@le8 ERROR:  syntax error at or near "t1" at character 8
2021-10-30 22:18:36.184 UTC [5568] postgres@le8 STATEMENT:  delete t1 where id=2;
2021-10-30 22:20:12.831 UTC [5568] postgres@le8 WARNING:  there is no transaction in progress
2021-10-30 22:25:24.731 UTC [5550] postgres@le8 LOG:  process 5550 still waiting for ShareLock on transaction 757 after 200.136 ms
2021-10-30 22:25:24.731 UTC [5550] postgres@le8 DETAIL:  Process holding the lock: 5568. Wait queue: 5550.
2021-10-30 22:25:24.731 UTC [5550] postgres@le8 CONTEXT:  while updating tuple (0,18) in relation "t1"
2021-10-30 22:25:24.731 UTC [5550] postgres@le8 STATEMENT:  update t1 set num=33 where id=8;
2021-10-30 22:25:59.867 UTC [5559] postgres@le8 LOG:  process 5559 still waiting for ShareLock on transaction 755 after 200.166 ms
2021-10-30 22:25:59.867 UTC [5559] postgres@le8 DETAIL:  Process holding the lock: 5550. Wait queue: 5559.
2021-10-30 22:25:59.867 UTC [5559] postgres@le8 CONTEXT:  while updating tuple (0,16) in relation "t1"
2021-10-30 22:25:59.867 UTC [5559] postgres@le8 STATEMENT:  update t1 set num=22 where id=1;
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 LOG:  process 5568 detected deadlock while waiting for ShareLock on transaction 756 after 200.164 ms
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 DETAIL:  Process holding the lock: 5559. Wait queue: .
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 CONTEXT:  while updating tuple (0,17) in relation "t1"
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 STATEMENT:  update t1 set num=66 where id=3;
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 ERROR:  deadlock detected
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 DETAIL:  Process 5568 waits for ShareLock on transaction 756; blocked by process 5559.
        Process 5559 waits for ShareLock on transaction 755; blocked by process 5550.
        Process 5550 waits for ShareLock on transaction 757; blocked by process 5568.
        Process 5568: update t1 set num=66 where id=3;
        Process 5559: update t1 set num=22 where id=1;
        Process 5550: update t1 set num=33 where id=8;
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 HINT:  See server log for query details.
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 CONTEXT:  while updating tuple (0,17) in relation "t1"
2021-10-30 22:26:17.076 UTC [5568] postgres@le8 STATEMENT:  update t1 set num=66 where id=3;
2021-10-30 22:26:17.077 UTC [5550] postgres@le8 LOG:  process 5550 acquired ShareLock on transaction 757 after 52545.381 ms
2021-10-30 22:26:17.077 UTC [5550] postgres@le8 CONTEXT:  while updating tuple (0,18) in relation "t1"
2021-10-30 22:26:17.077 UTC [5550] postgres@le8 STATEMENT:  update t1 set num=33 where id=8;
le8=!#

```

В глаза бросается: `ERROR:  deadlock detected`
в окружающих его строчках видно что сначала транзакции ждали друг друга, а потом идут четкие указания какие команды мызвали deadlock 

Делаем вывод, если логирование включено то да можно.


#### 4. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
##### Результат:

Получить чистый положительный результат не получилось:
В статьях  вроде этой 
https://habr.com/ru/company/postgrespro/blog/465263/
приводятся примеры похожие на задение НО там ЕСТЬ условие WHERE

У меня получилась воспроизвести только случай с созданием индекса, но это не совсем корректно сюда записывать так как лок возник именно в момент создания индекса, а не на операции UPDATE

Поэтому ответ "возможно могут."


#### 5. Попробуйте воспроизвести такую ситуацию.
##### Результат:
```
```
