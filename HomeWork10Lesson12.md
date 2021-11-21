# Домашнее задание
## Разворачиваем и настраиваем БД с большими данными

### Цель:
- знать различные механизмы загрузки данных
- уметь пользоваться различными механизмами загрузки данных
- Необходимо провести сравнение скорости работы запросов на различных СУБД

### Задание и ход выполнения по этапам: 
#### 1. Выбрать одну из СУБД      
##### Результат:
- выбрал гугловую BigQuery

#### 2. Загрузить в неё данные (10 Гб)
##### Результат:
- Делал как показали на лекции: зашел в BigQuery  подключил публичный датасет, выгрузил bigquery-public-data.chicago_taxi_trips.taxi_trips в s3 бакет гугловый
- далее закачал 10 Гб на файловую систему ВМ (`ub20-postgres-12`)
```
Student@ub20-postgres-12:~$ sudo -u postgres -i
postgres@ub20-postgres-12:~$ cd /opt/postgres-data/
postgres@ub20-postgres-12:/opt/postgres-data$ mkdir external
postgres@ub20-postgres-12:/opt/postgres-data/external$ gsutil -m cp gs://lesson12-20211121/lesson12-tt/tt-00000000000*.csv .
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000001.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000000.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000005.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000002.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000003.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000006.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000004.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000008.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000007.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000009.csv...
/ [10/10 files][  2.5 GiB/  2.5 GiB] 100% Done 116.3 MiB/s ETA 00:00:00
Operation completed over 10 objects/2.5 GiB.
postgres@ub20-postgres-12:/opt/postgres-data/external$ ls -la
total 2606064
drwxrwxr-x 2 postgres postgres      4096 Nov 21 07:05 .
drwxr-xr-x 5 postgres postgres      4096 Nov 21 07:02 ..
-rw-rw-r-- 1 postgres postgres 267554544 Nov 21 07:05 tt-000000000000.csv
-rw-rw-r-- 1 postgres postgres 266280306 Nov 21 07:05 tt-000000000001.csv
-rw-rw-r-- 1 postgres postgres 267544192 Nov 21 07:05 tt-000000000002.csv
-rw-rw-r-- 1 postgres postgres 267502453 Nov 21 07:05 tt-000000000003.csv
-rw-rw-r-- 1 postgres postgres 266307734 Nov 21 07:05 tt-000000000004.csv
-rw-rw-r-- 1 postgres postgres 266304069 Nov 21 07:05 tt-000000000005.csv
-rw-rw-r-- 1 postgres postgres 267301294 Nov 21 07:05 tt-000000000006.csv
-rw-rw-r-- 1 postgres postgres 266355147 Nov 21 07:05 tt-000000000007.csv
-rw-rw-r-- 1 postgres postgres 267054390 Nov 21 07:05 tt-000000000008.csv
-rw-rw-r-- 1 postgres postgres 266346347 Nov 21 07:05 tt-000000000009.csv
postgres@ub20-postgres-12:/opt/postgres-data/external$ gsutil -m cp gs://lesson12-20211121/lesson12-tt/tt-00000000001*.csv .
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000010.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000015.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000011.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000012.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000016.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000013.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000018.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000017.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000019.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000014.csv...
| [10/10 files][  2.5 GiB/  2.5 GiB] 100% Done 108.7 MiB/s ETA 00:00:00
Operation completed over 10 objects/2.5 GiB.
postgres@ub20-postgres-12:/opt/postgres-data/external$ gsutil -m cp gs://lesson12-20211121/lesson12-tt/tt-00000000002*.csv .
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000021.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000020.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000022.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000023.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000025.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000024.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000029.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000027.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000028.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000026.csv...
/ [10/10 files][  2.5 GiB/  2.5 GiB] 100% Done 103.6 MiB/s ETA 00:00:00
Operation completed over 10 objects/2.5 GiB.
postgres@ub20-postgres-12:/opt/postgres-data/external$ gsutil -m cp gs://lesson12-20211121/lesson12-tt/tt-00000000003*.csv .
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000030.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000035.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000031.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000032.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000037.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000034.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000036.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000033.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000038.csv...
Copying gs://lesson12-20211121/lesson12-tt/tt-000000000039.csv...
\ [10/10 files][  2.5 GiB/  2.5 GiB] 100% Done 122.8 MiB/s ETA 00:00:00
Operation completed over 10 objects/2.5 GiB.
postgres@ub20-postgres-12:/opt/postgres-data/external$ du -h
10G     .

```
- и залил все в созданную базу `lesson12` в таблицу  `taxi_trips`, предварительно включив тайминг
```
postgres=# create database lesson12;
CREATE DATABASE
postgres=# \c lesson12

lesson12=# create table taxi_trips (
lesson12(# unique_key text,
lesson12(# taxi_id text,
lesson12(# trip_start_timestamp TIMESTAMP,
lesson12(# trip_end_timestamp TIMESTAMP,
lesson12(# trip_seconds bigint,
lesson12(# trip_miles numeric,
lesson12(# pickup_census_tract bigint,
lesson12(# dropoff_census_tract bigint,
lesson12(# pickup_community_area bigint,
lesson12(# dropoff_community_area bigint,
lesson12(# fare numeric,
lesson12(# tips numeric,
lesson12(# tolls numeric,
lesson12(# extras numeric,
lesson12(# trip_total numeric,
lesson12(# payment_type text,
lesson12(# company text,
lesson12(# pickup_latitude numeric,
lesson12(# pickup_longitude numeric,
lesson12(# pickup_location text,
lesson12(# dropoff_latitude numeric,
lesson12(# dropoff_longitude numeric,
lesson12(# dropoff_location text
lesson12(# );
CREATE TABLE

lesson12=# \timing


lesson12=# COPY taxi_trips(unique_key,
lesson12(# taxi_id,
tips,
tolls,
extras,
lesson12(# trip_start_timestamp,
lesson12(# trip_end_timestamp,
lesson12(# trip_seconds,
pickup_latitude,
lesson12(# trip_miles,
lesson12(# pickup_census_tract,
lesson12(# dropoff_census_tract,
lesson12(# pickup_community_area,
lesson12(# dropoff_community_area,
lesson12(# fare,
lesson12(# tips,
lesson12(# tolls,
lesson12(# extras,
lesson12(# trip_total,
lesson12(# payment_type,
lesson12(# company,
lesson12(# pickup_latitude,
lesson12(# pickup_longitude,
lesson12(# pickup_location,
lesson12(# dropoff_latitude,
lesson12(# dropoff_longitude,
lesson12(# dropoff_location)
lesson12-# FROM PROGRAM 'awk FNR-1 /opt/postgres-data/external/tt-00000000000*.csv | cat' DELIMITER ',' CSV HEADER;
COPY 6399679
Time: 100723.371 ms (01:40.723)
lesson12=# COPY taxi_trips(unique_key,
taxi_id,
trip_start_timestamp,
trip_end_timestamp,
trip_seconds,
trip_miles,
pickup_census_tract,
dropoff_census_tract,
pickup_community_area,
dropoff_community_area,
fare,
tips,
tolls,
extras,
trip_total,
payment_type,
company,
pickup_latitude,
pickup_longitude,
pickup_location,
dropoff_latitude,
dropoff_longitude,
dropoff_location)
FROM PROGRAM 'awk FNR-1 /opt/postgres-data/external/tt-00000000001*.csv | cat' DELIMITER ',' CSV HEADER;
COPY 6527358
Time: 117962.470 ms (01:57.962)
lesson12=# COPY taxi_trips(unique_key,
taxi_id,
trip_start_timestamp,
trip_end_timestamp,
trip_seconds,
trip_miles,
pickup_census_tract,
dropoff_census_tract,
pickup_community_area,
dropoff_community_area,
fare,
tips,
tolls,
extras,
trip_total,
payment_type,
company,
pickup_latitude,
pickup_longitude,
pickup_location,
dropoff_latitude,
dropoff_longitude,
dropoff_location)
FROM PROGRAM 'awk FNR-1 /opt/postgres-data/external/tt-00000000002*.csv | cat' DELIMITER ',' CSV HEADER;
COPY 6548709
Time: 177145.521 ms (02:57.146)
lesson12=# COPY taxi_trips(unique_key,
taxi_id,
trip_start_timestamp,
trip_end_timestamp,
trip_seconds,
trip_miles,
pickup_census_tract,
dropoff_census_tract,
pickup_community_area,
dropoff_community_area,
fare,
tips,
tolls,
extras,
trip_total,
payment_type,
company,
pickup_latitude,
pickup_longitude,
pickup_location,
dropoff_latitude,
dropoff_longitude,
dropoff_location)
FROM PROGRAM 'awk FNR-1 /opt/postgres-data/external/tt-00000000003*.csv | cat' DELIMITER ',' CSV HEADER;

COPY 6516955
Time: 961516.468 ms (16:01.516)
lesson12=#

```
- как видно выше загрузка первых 7,5Гб идет "на ура" по полторы-две минуты, но третья порция загружалась аж 16 минут, вывод: при загрузке реально больших данных следует менять параметры сервера, влияющие на загрузку данных

#### 2. Сравнить скорость выполнения запросов на PosgreSQL и выбранной СУБД
#### 2.1 Postgres14
##### 2.1 Результат:

Данные уже в postgres. Теперь поселектим данные из таблицы:
```
lesson12=# select count(tips) from taxi_trips;
  count
----------
 25992331
(1 row)

Time: 1638861.406 ms (27:18.861)

```
- ого, 27 минут, ну мы знали что postgres долго каунтит, но очень долго, теперь сделаем так: 
```
lesson12=# vacuum analyze taxi_trips;
VACUUM
Time: 54091.874 ms (00:54.092)

```
- и еще раз, каунт:
```
lesson12=# select count(tips) from taxi_trips;
  count
----------
 25992331
(1 row)

Time: 125372.413 ms (02:05.372)

```
- уже лучше
- теперь сделаем такой запрос:
```
lesson12=# select payment_type, round(sum(tips)/Sum(trip_total)*100)+0 as percent, count(*) from taxi_trips
lesson12-# group by payment_type;
 payment_type | percent |  count
--------------+---------+----------
 Cash         |       0 | 14871119
 Credit Card  |      17 | 10831826
 Dispute      |       0 |    14752
 Mobile       |      15 |    32744
 No Charge    |       2 |   125727
 Pcard        |       3 |     5380
 Prcard       |       1 |    41331
 Prepaid      |       0 |       76
 Unknown      |       2 |    69668
 Way2ride     |      15 |       78
(10 rows)

Time: 136349.591 ms (02:16.350)

```

#### 2.2 BigQuery
##### 2.2 Результат:
подсчет записей:
```
select count(*)
from `bigquery-public-data.chicago_taxi_trips.taxi_trips`
;

Query complete (0.6 sec elapsed, 0 B processed)
```
- другой запрос:
```
select payment_type, round(sum(tips)/Sum(trip_total)*100)+0 as percent, count(*)
from `bigquery-public-data.chicago_taxi_trips.taxi_trips`
group by payment_type;

Query complete (1.1 sec elapsed, 4.6 GB processed)
```
##### 2. Результат:
Данных, судя по описанию датасета BigQuery больше, а скорость выше. Ну, тут разные технологии обработки и доступа к данным

#### 3.Описать что и как делали и с какими проблемами столкнулись
##### Результат:

- Хотел еще mysql добавать к сравнению, но не смог быстро понять почему он не дает загрузить данные из файла в таблицу, что то там, не так, За последние 10 лет они ухитрились испортить отличный мануал который у них когда-то был, теперь что-то найти черт ногу сломит...  не стал тратить время, и удалил ВМ
```

```
- кроме того, что описано выше, из любопытства создал, как было показано в примере на лекции `foreign table`, поселектил из неё данные, но так как мы процедуры/функции еще не изучали не стал загружать  данные через этот механизм  (мне не понравилось, что нельзя разом подключить несколько файлов по маске => надо или по одному грузить или писать процедуру "подмены файла"), (ну и так тоже понятно, что выгружать и потом инсертить будет дольше)
```
lesson12=# create extension file_fdw;
CREATE EXTENSION
lesson12=# create server pgcsv foreign data wrapper file_fdw;
CREATE SERVER

lesson12=# create foreign table taxi_trips_fdw_2 (
unique_key text,
taxi_id text,
trip_start_timestamp TIMESTAMP,
trip_end_timestamp TIMESTAMP,
trip_seconds bigint,
trip_miles numeric,
pickup_census_tract bigint,
dropoff_census_tract bigint,
pickup_community_area bigint,
dropoff_community_area bigint,
fare numeric,
tips numeric,
tolls numeric,
extras numeric,
trip_total numeric,
payment_type text,
company text,
pickup_latitude numeric,
lesson12(# unique_key text,
pickup_location text,
dropoff_latitude numeric,
dropoff_longitude numeric,
dropoff_location text
)
server pgcsv
lesson12(# taxi_id text,
lesson12(# trip_start_timestamp TIMESTAMP,
lesson12(# trip_end_timestamp TIMESTAMP,
lesson12(# trip_seconds bigint,
lesson12(# trip_miles numeric,
lesson12(# pickup_census_tract bigint,
lesson12(# dropoff_census_tract bigint,
lesson12(# pickup_community_area bigint,
lesson12(# dropoff_community_area bigint,
lesson12(# fare numeric,
lesson12(# tips numeric,
lesson12(# tolls numeric,
lesson12(# extras numeric,
lesson12(# trip_total numeric,
lesson12(# payment_type text,
lesson12(# company text,
lesson12(# pickup_latitude numeric,
lesson12(# pickup_longitude numeric,
lesson12(# pickup_location text,
lesson12(# dropoff_latitude numeric,
lesson12(# dropoff_longitude numeric,
lesson12(# dropoff_location text
lesson12(# )
lesson12-# server pgcsv
lesson12-# options(filename '/opt/postgres-data/external/tt-000000000000.csv', format 'csv', header 'true', delimiter ',');
CREATE FOREIGN TABLE

```
