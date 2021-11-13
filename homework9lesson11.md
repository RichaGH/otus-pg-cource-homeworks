# Домашнее задание
## Репликация

### Цель:
- реализовать свой миникластер на 3 ВМ.

### Задание и ход выполнения по этапам: 
#### 1.0 Подготовительный этап

- Создадим 4 ВМ:
```
NAME               ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
ub20-postgres-11a  us-central1-c  e2-medium                  10.128.0.14  35.239.249.24  RUNNING
ub20-postgres-11b  us-central1-c  e2-medium                  10.128.0.15  35.225.225.98  RUNNING
ub20-postgres-11c  us-central1-c  e2-medium                  10.128.0.16  34.135.60.166  RUNNING
ub20-postgres-11d  us-central1-c  e2-medium                  10.128.0.17  34.133.54.171  RUNNING
```

- Установим postgres 14 на всех ВМ по одному сценарию:
```
sudo apt update && sudo apt upgrade -y -q
sudo shutdown -r now
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
sudo systemctl status postgresql
sudo -u postgres pg_lsclusters
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

- Подготовим конфиги, создадим пользователей и тд 
```
sudo -u postgres -i psql
postgres=# create user replica with replication encrypted password '123';
CREATE ROLE
postgres=# CREATE USER rewind SUPERUSER encrypted PASSWORD '123';
CREATE ROLE
postgres=# \! echo "host all rewind 0.0.0.0/0 scram-sha-256" >> /etc/postgresql/14/main/pg_hba.conf
postgres=# \! echo "host replication replica 0.0.0.0/0 scram-sha-256" >> /etc/postgresql/14/main/pg_hba.conf
postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# \q
```
Здесь изменим строку с параметром listen_addresses (в `sudo -u postgres -i vim /etc/postgresql/14/main/postgresql.conf`) получим это:
```
listen_addresses = '10.128.0.14,10.128.0.15,10.128.0.16,10.128.0.17,localhost'          # what IP address(es) to listen on;
```
- перезапустим кластер
```
sudo pg_ctlcluster 14 main restart
```
  
#### 1.1 На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение. 
##### Результат:
```
postgres=# create database db1_s1;
CREATE DATABASE
postgres=# \c db1_s1
You are now connected to database "db1_s1" as user "postgres".

db1_s1=# create table test (id integer, name varchar(40));
CREATE TABLE
db1_s1=# create table test2 (id integer, textus varchar(250));
CREATE TABLE
db1_s1=# insert into test (id,name)Values(1,'Sidor');
INSERT 0 1
db1_s1=# insert into test (id,name)Values(2,'Fedor');
INSERT 0 1
db1_s1=# insert into test (id,name)Values(3,'Feofan');
INSERT 0 1
```

#### 1.2 Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2. 
##### Результат:
- Создаем публикацию (сразу так как не зависим от другого сервера)
```
db1_s1=# CREATE PUBLICATION test_pub FOR TABLE test;
CREATE PUBLICATION
```
- подписываемся (только после публикации на втором сервере)
```
db1_s1=# CREATE SUBSCRIPTION test2_sub
db1_s1-# CONNECTION 'host=10.128.0.15 port=5432 user=rewind password=123 dbname=db2_s2'
db1_s1-# PUBLICATION test2_pub WITH (copy_data = true);
NOTICE:  created replication slot "test2_sub" on publisher
CREATE SUBSCRIPTION
db1_s1=# select * from test2;
 id |          textus
----+---------------------------
  1 | This is a table
  2 | Odnazhdy v studenuyu....
  3 | I vot poshli ony v les...
(3 rows)

```

#### 1.3 На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение. 
##### Результат:
```
postgres=# create database db2_s2;
CREATE DATABASE
postgres=# \c db2_s2
You are now connected to database "db2_s2" as user "postgres".

db2_s2=# create table test (id integer, name varchar(40));
CREATE TABLE
db2_s2=# create table test2 (id integer, textus varchar(250));
CREATE TABLE
db2_s2=# insert into test2 (id,textus)Values(1,'This is a table');
INSERT 0 1
db2_s2=# insert into test2 (id,textus)Values(2,'Odnazhdy v studenuyu....');
INSERT 0 1
db2_s2=# insert into test2 (id,textus)Values(3,'I vot poshli ony v les...');
INSERT 0 1

```

#### 1.4 Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1. 
##### Результат:
- Создаем публикацию (сразу так как не зависим от другого сервера)
```
db2_s2=# CREATE PUBLICATION test2_pub FOR TABLE test2;
CREATE PUBLICATION
```
- подписываемся (только после публикации на другом сервере)
```
db2_s2=# CREATE SUBSCRIPTION test_sub
CONNECTION 'host=10.128.0.14 port=5432 user=rewind password=123 dbname=db1_s1'
PUBLICATION test_pub WITH (copy_data = true);
NOTICE:  created replication slot "test_sub" on publisher
CREATE SUBSCRIPTION
db2_s2=# select * from test;
 id |  name
----+--------
  1 | Sidor
  2 | Fedor
  3 | Feofan
(3 rows)
```
- Добавим в таблицы пару строчек: 
```
Student@ub20-postgres-11a:~$ sudo -u postgres -i psql
psql (14.1 (Ubuntu 14.1-1.pgdg20.04+1))
Type "help" for help.

postgres=# \c db1_s1
You are now connected to database "db1_s1" as user "postgres".
db1_s1=# insert into test (id,name)Values(4,'Vasiliy');
INSERT 0 1
```
```
Student@ub20-postgres-11b:~$ sudo -u postgres -i psql
psql (14.1 (Ubuntu 14.1-1.pgdg20.04+1))
Type "help" for help.

postgres=# \c db2_s2
You are now connected to database "db2_s2" as user "postgres".

db2_s2=# insert into test2 (id,textus)Values(2,'Skazhika DyaDya...');
INSERT 0 1

```
- и убедимся что все реплицируется:
```
db2_s2=# select * from test;
 id |  name
----+---------
  1 | Sidor
  2 | Fedor
  3 | Feofan
  4 | Vasiliy
(4 rows)

```
```
db1_s1=# select * from test2;
 id |          textus
----+---------------------------
  1 | This is a table
  2 | Odnazhdy v studenuyu....
  3 | I vot poshli ony v les...
  2 | Skazhika DyaDya...
(4 rows)

```
#### 1.5 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). 
##### Результат:

- так как слот репликации создается на публикующем сервере, мы должны дать другое имя подписке на ВМ3 (такое которое еще не задействовано) поэтому в имя подписки в конце дабавим 2
```
postgres=# create database db3_s3;
CREATE DATABASE

postgres=# \c db3_s3
You are now connected to database "db3_s3" as user "postgres".

db3_s3=# CREATE SUBSCRIPTION test_sub2
CONNECTION 'host=10.128.0.14 port=5432 user=rewind password=123 dbname=db1_s1'
PUBLICATION test_pub WITH (copy_data = true);
NOTICE:  created replication slot "test_sub2" on publisher
CREATE SUBSCRIPTION


db3_s3=# CREATE SUBSCRIPTION test2_sub2
db3_s3-# CONNECTION 'host=10.128.0.15 port=5432 user=rewind password=123 dbname=db2_s2'
db3_s3-# PUBLICATION test2_pub WITH (copy_data = true);
NOTICE:  created replication slot "test2_sub2" on publisher
CREATE SUBSCRIPTION

db3_s3=# select * from test;
 id |  name
----+---------
  1 | Sidor
  2 | Fedor
  3 | Feofan
  4 | Vasiliy
(4 rows)

db3_s3=# select * from test2;
 id |          textus
----+---------------------------
  1 | This is a table
  2 | Odnazhdy v studenuyu....
  3 | I vot poshli ony v les...
  2 | Skazhika DyaDya...
(4 rows)

```

#### 1.6 Небольшое описание, того, что получилось.
##### Результат:
В результате имеем 3 ВМ: 

- на первой  мы можем писать в таблицу test и из таблицы test2 *должны* только читать. 
По хорошему пользователям, для работы в базе на первой ВМ нужно будет ограничить права на таблицу test2 только операцией select
```
db1_s1=# select * from pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 5212
usesysid         | 16385
usename          | rewind
application_name | test_sub
client_addr      | 10.128.0.15
client_hostname  |
client_port      | 38356
backend_start    | 2021-11-13 14:37:00.061589+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/1717788
write_lsn        | 0/1717788
flush_lsn        | 0/1717788
replay_lsn       | 0/1717788
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2021-11-13 15:50:59.46268+00
-[ RECORD 2 ]----+------------------------------
pid              | 5268
usesysid         | 16385
usename          | rewind
application_name | test_sub2
client_addr      | 10.128.0.16
client_hostname  |
client_port      | 56440
backend_start    | 2021-11-13 14:45:24.622086+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/1717788
write_lsn        | 0/1717788
flush_lsn        | 0/1717788
replay_lsn       | 0/1717788
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2021-11-13 15:50:59.555978+00
```
- на второй  мы можем писать в таблицу test2 и из таблицы test *должны* только читать. 
По хорошему пользователям, для работы в базе на первой ВМ нужно будет ограничить права на таблицу test только операцией select
```
db2_s2=# insert into test2 (id,textus)Values(2,'Skazhika DyaDya...');
INSERT 0 1
db2_s2=# select * from pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 5160
usesysid         | 16385
usename          | rewind
application_name | test2_sub
client_addr      | 10.128.0.14
client_hostname  |
client_port      | 51148
backend_start    | 2021-11-13 14:39:57.296606+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/171A2C8
write_lsn        | 0/171A2C8
flush_lsn        | 0/171A2C8
replay_lsn       | 0/171A2C8
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2021-11-13 15:50:19.725651+00
-[ RECORD 2 ]----+------------------------------
pid              | 5206
usesysid         | 16385
usename          | rewind
application_name | test2_sub2
client_addr      | 10.128.0.16
client_hostname  |
client_port      | 57324
backend_start    | 2021-11-13 14:46:45.0742+00
backend_xmin     |
state            | streaming
sent_lsn         | 0/171A2C8
write_lsn        | 0/171A2C8
flush_lsn        | 0/171A2C8
replay_lsn       | 0/171A2C8
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2021-11-13 15:50:19.829182+00

```
- Третья ВМ предназначена по замыслу только для чтения следовательно пользователей следует ограничить правом на чтение обоих таблиц

- На всех 3ех ВМ мы использовали реплики таблицы в разные базы, что подтверждает возможность переносить данные отдельных таблиц между серверами независимо от того в каких БД они находяться, все что нужно для данного вида репликации идентичность самих таблиц.

#### 2. реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.
##### Результат:
не вышло:

