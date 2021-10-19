# Домашнее задание
## Работа с базами данных, пользователями и правами

## Цель:
1. создание новой базы данных, схемы и таблицы
2. создание роли для чтения данных из созданной схемы созданной базы данных
3. создание роли для чтения и записи из созданной схемы созданной базы данных

### Задание и ход выполнения по этапам: 
#### 1. Cоздайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL)
##### Результат:
создаем новую ВМ с centos8 "на борту": 

```
*Created*

NAME           ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
c8-postgres-1  us-central1-c  e2-medium                  10.128.0.8   35.188.18.32  RUNNING
```

Обновим пакеты
```
sudo dnf update

....
....

Complete!

```

Теперь установим postgres 
согласно инструкции с (https://www.postgresql.org/download/linux/redhat/)
```
# Install the repository RPM:
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Disable the built-in PostgreSQL module:
sudo dnf -qy module disable postgresql

# Install PostgreSQL:
sudo dnf install -y postgresql13-server

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable postgresql-13
sudo systemctl start postgresql-13
```
готово

#### 2. зайдите в созданный кластер под пользователем postgres
##### Результат:
```
[Student@c8-postgres-1 ~]$ sudo -u postgres -i psql
psql (13.4)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" via socket in "/var/run/postgresql" at port "5432".

```

#### 3. создайте новую базу данных testdb
##### Результат:
```
postgres=# create database testdb;
CREATE DATABASE
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(4 rows)

```

#### 4. зайдите в созданную базу данных под пользователем postgres
##### Результат:
```
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=#
```

#### 5. создайте новую схему testnm
##### Результат:
```
testdb=# create schema testnm;
CREATE SCHEMA
testdb=# \dn
  List of schemas
  Name  |  Owner
--------+----------
 public | postgres
 testnm | postgres
(2 rows)

testdb=#

```
#### 6. создайте новую таблицу t1 с одной колонкой c1 типа integer
##### Результат:
```
testdb=# create table t1 (c1 integer);
CREATE TABLE
testdb=# \dt t1
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

testdb=# \dt+ t1
                           List of relations
 Schema | Name | Type  |  Owner   | Persistence |  Size   | Description
--------+------+-------+----------+-------------+---------+-------------
 public | t1   | table | postgres | permanent   | 0 bytes |
(1 row)

```
#### 7. вставьте строку со значением c1=1
##### Результат:
```
testdb=# insert into t1 (c1)values (1);
INSERT 0 1
testdb=# select c1 from t1;
 c1
----
  1
(1 row)

testdb=#

```
#### 8. создайте новую роль readonly
##### Результат:
```
testdb=# create role readonly;
CREATE ROLE
testdb=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 readonly  | Cannot login                                               | {}

```
#### 9. дайте новой роли право на подключение к базе данных testdb
##### Результат:
Здесь и далее нам помогла документация доступная по ссылке (https://postgrespro.ru/docs/postgrespro/13/sql-grant)
```
testdb=# GRANT CONNECT ON DATABASE testdb TO  readonly;
GRANT
testdb=#
```
#### 10. дайте новой роли право на использование схемы testnm
##### Результат:
```
testdb=# GRANT USAGE ON SCHEMA testnm TO readonly;
GRANT
testdb=#
```
#### 11. дайте новой роли право на select для всех таблиц схемы testnm
##### Результат:
```
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
testdb=# 
```
#### 12. создайте пользователя testread с паролем test123
##### Результат:
```
testdb=# CREATE ROLE testread WITH LOGIN PASSWORD 'test123';
CREATE ROLE
testdb=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 readonly  | Cannot login                                               | {}
 testread  |                                                            | {}

testdb=#

```
#### 13. дайте роль readonly пользователю testread
##### Результат:
```
testdb=# GRANT readonly TO testread;
GRANT ROLE
testdb=# \du
                                    List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+------------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 readonly  | Cannot login                                               | {}
 testread  |                                                            | {readonly}

testdb=#

```
#### 14. зайдите под пользователем testread в базу данных testdb
##### Результат:

- Вариант 1: сменим "контекст" пользователя (это дает возможность выпольнять команды с правами указанной роли оставаясь при этом реально подключенным под суперпользователем postgres) и при необходимости вернуться в контекст суперпользователя через `SET ROLE`
(см https://postgrespro.ru/docs/postgrespro/13/sql-set-role)
```
testdb=# SET ROLE testread;
SET
testdb=> \conninfo
You are connected to database "testdb" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
testdb=> select current_user;
 current_user
--------------
 testread
(1 row)

testdb=>

```

- Вариант 2: подключиться к нашей БД под пользователем testread из окна psql:

```
postgres=# \c testdb testread 127.0.0.1
Password for user testread:
You are now connected to database "testdb" as user "testread" on host "127.0.0.1" at port "5432".
testdb=> \conninfo
You are connected to database "testdb" as user "testread" on host "127.0.0.1" at port "5432".

```

- Вариант 3: выйти и зайти, подключившись напрямую к нашей БД под пользователем testread:
```
[Student@c8-postgres-1 ~]$ sudo -u postgres -i psql -h 127.0.0.1 -d testdb -U testread -W
Password:
psql (13.4)
Type "help" for help.

testdb=>
```

#### 15. сделайте select * from t1;
##### Результат:
```
testdb=> select * from t1;
ERROR:  permission denied for table t1
```
#### 16. получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
##### Результат:
Нет считать таблицу не удалось, так как к ней не достаточно прав у пользователя testread
#### 17. напишите что именно произошло в тексте домашнего задания
##### Результат:
Так как в задании пункт 6 не было указано в какой схеме создавать таблицу, то она была создана без модификатора определяющего принадлежность к схеме, кроме того, она создавалась под суперпользователем и как следствие попала в схему public (в П6 есть вывод команды `\dt+ t1`).
Это произошло потому что (как видим search path содержит 2 элемента "$user", public):
```
testdb=> SHOW search_path;
   search_path
-----------------
 "$user", public
(1 row)

testdb=>

```
Первый элемент ссылается на схему с именем текущего пользователя. Если такой схемы не существует,(а мы не создавали в testdb схемы postgres) ссылка на неё игнорируется. Второй элемент ссылается на схему public поэтому таблица и попала в неё.

Прав на указанную схему мы не давали ни пользователю testuser персонально, ни роли readonly, в которую включили пользователя.


#### 18. у вас есть идеи почему? ведь права то дали?
##### Результат:
При создании  таблицы она, если не указан модификатор, задающий имя схемы (имя предваряется именем схемы и точкой: например `myschemaName.myTableName`), то таблица создается в первой доступной схеме из searchPath.
В нашем случае это public.
Прав на схему public мы не давали ни пользователю testuser персонально, ни роли readonly, в которую включили пользователя.
Вот доступа и нет.
#### 19. посмотрите на список таблиц
##### Результат:

- вариант 1:
```
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres

```

- Вариант 2:
```
testdb=> SELECT n.nspname as "Schema",
testdb->   c.relname as "Name",
testdb->   CASE c.relkind WHEN 'r' THEN 'table' WHEN 'v' THEN 'view' WHEN 'm' THEN 'materialized view' WHEN 'i' THEN 'index' WHEN 'S' THEN 'sequence' WHEN 's' THEN 'special' WHEN 'f' THEN 'foreign table' WHEN 'p' THEN 'partitioned table' WHEN 'I' THEN 'partitioned index' END as "Type",
testdb->   pg_catalog.pg_get_userbyid(c.relowner) as "Owner"
testdb-> FROM pg_catalog.pg_class c
testdb->      LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
testdb-> WHERE c.relkind IN ('r','p','')
testdb->       AND n.nspname <> 'pg_catalog'
testdb->       AND n.nspname <> 'information_schema'
testdb->       AND n.nspname !~ '^pg_toast'
testdb->   AND pg_catalog.pg_table_is_visible(c.oid)
testdb-> ORDER BY 1,2;
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

```

это тот же запрос что и выше, но по нему видно что в выборку по умолчанию для команды `\dt` попадают не все таблицы.

Если мы хотим видеть "больше", то либо используем `\dtS` (тогда к стандартному выводу получим системные таблицы и pg_catalog) или в селекте из варианта 2 снимаем часть ограничений

#### 20. подсказка в шпаргалке под пунктом 20
##### Результат:
Ну да так и есть
#### 21. а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
##### Результат:
Могло получиться иначе в двух случаях:
- таблицу создавали с явным указанием схемы, 
- если search path был изменен до того как создавали таблицу и в него внесли перед схемой public схему testnm. Например командой вида `SET search_path TO testns,public;`
Но задание этого не предусматривало.
#### 22. вернитесь в базу данных testdb под пользователем postgres
##### Результат:
- Вариант1: Если мы входили через `SET ROLE` то можно вернуться через (если заходили напрямую под testread то так не получится не хватит прав)
```
testdb=> SET ROLE postgres;
SET
testdb=#
```

- Вариант 2: (не получится если пользователю postgres не задан пароль)
```
testdb=> \c testdb postgres
Password for user postgres:
You are now connected to database "testdb" as user "postgres".
testdb=#
```

- Вариант 3: выйти и зайти из КС запущенной в контексте postgres и переключиться в указанную БД
```
[Student@c8-postgres-1 ~]$ sudo -u postgres -i psql
psql (13.4)
Type "help" for help.

postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=#
```

#### 23. удалите таблицу t1
##### Результат:
```
testdb=# drop table t1;
DROP TABLE
testdb=# \dt
Did not find any relations.
testdb=#

```
#### 24. создайте ее заново но уже с явным указанием имени схемы testnm
##### Результат:
```
testdb=# create table testnm.t1 (c1 integer);
CREATE TABLE
```
#### 25. вставьте строку со значением c1=1
##### Результат:
```
testdb=# insert into testnm.t1 (c1)values(1);
INSERT 0 1
```
#### 26. зайдите под пользователем testread в базу данных testdb
##### Результат:
Воспользуемся "вариантом 1"
```
testdb=# SET ROLE testread ;
SET
testdb=>
```
#### 27. сделайте select * from testnm.t1;
##### Результат:
```
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
testdb=>

```
#### 28. получилось?
##### Результат:
Нет.
#### 29. есть идеи почему? если нет - смотрите шпаргалку
##### Результат:
Судя по результатам эксперимента в котором я проверяю права и вижу что их нет:
```
testdb=# \dp testnm.t1;
                            Access privileges
 Schema | Name | Type  | Access privileges | Column privileges | Policies
--------+------+-------+-------------------+-------------------+----------
 testnm | t1   | table |                   |                   |
(1 row)
```
затем создаю таблицу testnm.t2, снова выдаю права на все таблицы в схеме 
```
testdb=# create table testnm.t2 (c2 integer);
CREATE TABLE
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
```
и проверяю что они появились
```
testdb=# \dp testnm.t*
                                Access privileges
 Schema | Name | Type  |     Access privileges     | Column privileges | Policies
--------+------+-------+---------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t2   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
(2 rows)
```
затем добавляю таблицу 3 (права на неё специально не выдавались)
```
testdb=# create table testnm.t3 (c3 integer);
CREATE TABLE
```
и проверяю права на 3ех таблицах (первые 2 права установлены третья нет):
```
testdb=# \dp testnm.t*
                                Access privileges
 Schema | Name | Type  |     Access privileges     | Column privileges | Policies
--------+------+-------+---------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t2   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t3   | table |                           |                   |
(3 rows)

```
можно сделать вывод, что права ALL TABLES выдаются только на существоющие в момент операции выдачи прав таблицы (К сожалению явных указаний на это не нашел в докуметации
https://postgrespro.ru/docs/postgresql/13/ddl-priv

https://postgrespro.ru/docs/postgrespro/13/sql-grant
и еще нескольких статьях "этого цикла"
)
поэтому пришлось эксперементировать

#### 30. как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
##### Результат:
```
testdb=# ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;
ALTER DEFAULT PRIVILEGES
testdb=# ;
testdb=# \dp testnm.t*
                                Access privileges
 Schema | Name | Type  |     Access privileges     | Column privileges | Policies
--------+------+-------+---------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t2   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t3   | table |                           |                   |
(3 rows)

```
теперь создадим еще одну таблицу и проверим права:
```
testdb=# create table testnm.t4 (c4 integer);
CREATE TABLE

testdb=# \dp testnm.t*
                                Access privileges
 Schema | Name | Type  |     Access privileges     | Column privileges | Policies
--------+------+-------+---------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t2   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t3   | table |                           |                   |
 testnm | t4   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
(4 rows)

```
Почему нет прав на t3? ответ в документации по  ALTER DEFAULT PRIVILEGES (https://postgrespro.ru/docs/postgresql/13/sql-alterdefaultprivileges)
"ALTER DEFAULT PRIVILEGES позволяет задавать права, применяемые к объектам, которые будут создаваться в будущем. (Эта команда не затрагивает права, назначенные уже существующим объектам.) В настоящее время можно задавать права только для схем, таблиц (включая представления и сторонние таблицы), последовательностей, функций и типов (включая домены)."

#### 31. сделайте select * from testnm.t1;
##### Результат:
сделаем 2 селекта
```
testdb=# SET ROLE readonly;
SET
testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)

testdb=> select * from testnm.t3;
ERROR:  permission denied for table t3

```

#### 32. получилось?
##### Результат:
Получилось для таблицы t1 так как в ходе эксперимента я выдал права на селект в эту таблицу,

А вот для таблицы t3 которой специально права не выдавались мы по прежнему не имеем прав.

#### 33. есть идеи почему? если нет - смотрите шпаргалку
##### Результат:
в документации по  ALTER DEFAULT PRIVILEGES (https://postgrespro.ru/docs/postgresql/13/sql-alterdefaultprivileges) находим что
"ALTER DEFAULT PRIVILEGES позволяет задавать права, применяемые к объектам, которые будут создаваться в будущем. (Эта команда не затрагивает права, назначенные уже существующим объектам.) В настоящее время можно задавать права только для схем, таблиц (включая представления и сторонние таблицы), последовательностей, функций и типов (включая домены)."
поэтому в таблицах остались те права которые были на них выданы до исполнения команды
`ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;`
Если теперь выполнить :
```
testdb=> SET ROLE postgres;
SET
testdb=#  GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
GRANT
```

то права на селект по всем таблицам схемы выровняются:
```
testdb=# \dp testnm.t*
                                Access privileges
 Schema | Name | Type  |     Access privileges     | Column privileges | Policies
--------+------+-------+---------------------------+-------------------+----------
 testnm | t1   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t2   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t3   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
 testnm | t4   | table | postgres=arwdDxt/postgres+|                   |
        |      |       | readonly=r/postgres       |                   |
(4 rows)
```
#### 31. сделайте select * from testnm.t1;
##### Результат:
```
testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)

testdb=> select * from testnm.t2;
 c2
----
(0 rows)

testdb=> select * from testnm.t3;
 c3
----
(0 rows)

testdb=> select * from testnm.t4;
 c4
----
(0 rows)

```
#### 32. получилось?
##### Результат:
33 ура!
##### Результат:
#### 34. теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
##### Результат:
```
testdb=> create table t2(c1 integer); insert into t2 values (2);
CREATE TABLE
INSERT 0 1
```
#### 35. а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
##### Результат:
мы опять попали в схему public
```
testdb=> \dt+
                             List of relations
 Schema | Name | Type  |  Owner   | Persistence |    Size    | Description
--------+------+-------+----------+-------------+------------+-------------
 public | t2   | table | testread | permanent   | 8192 bytes |
(1 row)


testdb=> SHOW search_path;
  search_path
----------------
 testns, public
(1 row)
```
хотя прав на создание объектов в схеме testns у нас нет, тем не менее, они есть в схеме public  так как роль public  неявно присутствует у любого пользователя и её права "присоединяются" к правам пользователя

#### 36. есть идеи как убрать эти права? если нет - смотрите шпаргалку
##### Результат:
- Вариант 1 (облегченный): ограничить роль public в праве создания таблиц в схеме public:
```
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```
при этом postgres сможет успешно создать таблицу, а вот наш testread  уже нет
```
testdb=# create table public.t5(c5 integer);
CREATE TABLE
testdb=# SET ROLE readonly ;
SET
testdb=> create table public.t6(c6 integer);
ERROR:  permission denied for schema public
LINE 1: create table public.t6(c6 integer);
                     ^
```
- Вариант 2 и мы можем усилить ограничения так: 
```
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON DATABASE testdb FROM PUBLIC; 
```
здесь слеудует помнить что никто не мешает владельцу схемы дать права на создание таблиц в своей схеме (в примере ниже testpub2 владелец схемы testpub):
```
testdb=> SET ROLE testpub2;
SET
testdb=> GRANT CREATE ON SCHEMA testpub TO PUBLIC;
GRANT
```
что приведет к открытию данного права для всех, но уже в этой конкретной схеме
```
testdb=> SET ROLE testread ;
SET
testdb=> create table testpub.tp1 (ctp1 integer);
CREATE TABLE                    
```
Дать более широкие права, владелец схемы не сможет, так как на текущий момент согласно (https://postgrespro.ru/docs/postgrespro/13/sql-alterdefaultprivileges)
"В настоящее время можно задавать права только для схем, таблиц (включая представления и сторонние таблицы), последовательностей, функций и типов (включая домены)." т.е. права по умолчанию на все схемы задать не получится


#### 37. если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
##### Результат:
Делал так: читал справку и статьи в интернете по ролям, как выдавать и отзывать и тд. и тп. (Иногда подглядывал в ответы чтобы понять в том ли направлении ищу).
В *36.* была шальная идея грохнуть PUBLIC, но подумав решил этого не делать так как удаление схемы не решает проблемы с выдачей прав владельцами других схем, а удаление Роли PUBLIC нигде не описано и предсказать как себя поведет система я на данном этапе не могу. (Подзаглянув в ответ понял, что вовремя остановился)
#### 38. теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
##### Результат:
```
testdb=# SET ROLE testread ;
SET
testdb=> create table tt3(c1 integer); insert into tt2 values (2);
ERROR:  permission denied for schema public
LINE 1: create table tt3(c1 integer);
                     ^
ERROR:  relation "tt2" does not exist
LINE 1: insert into tt2 values (2);

```
#### 39. расскажите что получилось и почему 
##### Результат:
Права у роли public на создание таблиц в схеме public отозваны, а в схеме testnm их не было, так как в путях поиска схем других схем нет а у роли readonly права только на select, то таблица не создалась.

#### PS
останавливаем postgres
```
sudo systemctl stop postgresql-13
sudo systemctl status postgresql-13
```
останавливаем хост
`sudo shutdown now`

Проверяем что ВМ остановлена
```
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
c8-postgres-1    us-central1-c  e2-medium                  10.128.0.8                TERMINATED

```
