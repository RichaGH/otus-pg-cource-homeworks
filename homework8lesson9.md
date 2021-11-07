# Домашнее задание
## Нагрузочное тестирование и тюнинг PostgreSQL

### Цель:
- сделать нагрузочное тестирование PostgreSQL
- настроить параметры PostgreSQL для достижения максимальной производительности

### Задание и ход выполнения по этапам: 
#### 1. сделать проект ---10
##### Результат:
Если вы про проект в GCP то использую созданный для домашек проект `postgres2021-19810421`
#### 2. сделать инстанс Google Cloud Engine типа e2-medium с ОС Ubuntu 20.04
##### Результат:
```
Created 
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
ub20-postgres-9  us-central1-c  e2-medium                  10.128.0.12  35.224.242.84  RUNNING

```
#### 3. поставить на него PostgreSQL 13 из пакетов собираемых postgres.org
##### Результат:
Готово
```
sudo apt update && sudo apt upgrade -y -q
sudo shutdown -r now
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql-13
sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 online postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```

#### 4. настроить кластер PostgreSQL 13 на максимальную производительность, не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
##### Результат:
- поменяем настройки в файле ` /etc/postgresql/13/main/postgresql.conf` изменял следующие параметры:
```
max_connections = 100  # оставил как рекомендует pgtune  ну у меня все равно их меньше так что этот параметр не повлиял на результаты теста
shared_buffers = 1GB # рекомендуют делать 1/4 от доступной ОЗУ
effective_cache_size = 2GB # рекомендуют делать 2/3 от доступной ОЗУ но так как в целых числах 
maintenance_work_mem = 256MB # рекомендуют делать 75 процентов от размера самой большой таблицы или индекса у меня в базе таблицы имеют размер 40 Мб максимум, но так как я не знаю как работает тесть и будет ли прирост то оставлю рекомендованное pgtune значение

checkpoint_timeout = 15min # Увеличил для минимизации влияния чекпоинта, вместе со следующим параметром это должно растянуть контрольную точку во времени, что по идее должно снизить нагрузку на систему
checkpoint_completion_target = 0.9
wal_buffers = 16MB # согласно документации увеличение буфера позволит лучше работать с большими транзакциями.
default_statistics_target = 100
random_page_cost = 2 # не рекомендубт делать меньше 2 сделал 2
effective_io_concurrency = 200
work_mem = 10240kB #5242kB дадим по больше раз уж мы количество фоновых по рекомендации от PGtune уменьшили
min_wal_size = 1GB
max_wal_size = 4GB
# Значения ниже рекоментованы PgTune на основании числа доступных ядер
max_worker_processes = 2 
max_parallel_workers_per_gather = 1
max_parallel_workers = 2
max_parallel_maintenance_workers = 1

# Параметры ниже отключают принудитульную синхронизацию с диском, это опасно в плане сохранности данных и восстановления при сбое но мы сегодня "храбрые" так как нам явно сказали так сделать
fsync=off
synchronous_commit=off
full_page_writes=off

```

#### 5. нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)
Установим sysbench согласно инструкциям из README.md (в ветке sysbench-master)
```
Student@ub20-postgres-9:~$ curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
Detected operating system as Ubuntu/focal.
Checking for curl...
Detected curl...
Checking for gpg...
Detected gpg...
Running apt-get update... done.
Installing apt-transport-https... done.
Installing /etc/apt/sources.list.d/akopytov_sysbench.list...done.
Importing packagecloud gpg key... done.
Running apt-get update... done.

The repository is setup! You can now install packages.
Student@ub20-postgres-9:~$ sudo apt -y install sysbench
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libatasmart4 libblockdev-fs2 libblockdev-loop2 libblockdev-part-err2 libblockdev-part2 libblockdev-swap2 libblockdev-utils2 libblockdev2 libnspr4 libnss3 libnuma1 libparted-fs-resize0 libudisks2-0
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libmysqlclient21 mysql-common
The following NEW packages will be installed:
  libmysqlclient21 mysql-common sysbench
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 1638 kB of archives.
After this operation, 8788 kB of additional disk space will be used.
Get:1 http://us-central1.gce.archive.ubuntu.com/ubuntu focal/main amd64 mysql-common all 5.8+1.0.5ubuntu2 [7496 B]
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu focal-updates/main amd64 libmysqlclient21 amd64 8.0.27-0ubuntu0.20.04.1 [1291 kB]
Get:3 https://packagecloud.io/akopytov/sysbench/ubuntu focal/main amd64 sysbench amd64 1.0.20-1 [340 kB]
Fetched 1638 kB in 0s (3869 kB/s)
Selecting previously unselected package mysql-common.
(Reading database ... 98002 files and directories currently installed.)
Preparing to unpack .../mysql-common_5.8+1.0.5ubuntu2_all.deb ...
Unpacking mysql-common (5.8+1.0.5ubuntu2) ...
Selecting previously unselected package libmysqlclient21:amd64.
Preparing to unpack .../libmysqlclient21_8.0.27-0ubuntu0.20.04.1_amd64.deb ...
Unpacking libmysqlclient21:amd64 (8.0.27-0ubuntu0.20.04.1) ...
Selecting previously unselected package sysbench.
Preparing to unpack .../sysbench_1.0.20-1_amd64.deb ...
Unpacking sysbench (1.0.20-1) ...
Setting up mysql-common (5.8+1.0.5ubuntu2) ...
update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Setting up libmysqlclient21:amd64 (8.0.27-0ubuntu0.20.04.1) ...
Setting up sysbench (1.0.20-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
Student@ub20-postgres-9:~$ sysbench --version
sysbench 1.0.20

```

- скопировал скрипты с https://github.com/Percona-Lab/sysbench-tpcc в папку на сервере выдал права пользователю постгрес и сделал файлы исполняемыми
```
postgres@ub20-postgres-9:~$ mkdir sbtm
postgres@ub20-postgres-9:~$ cd sbtm
postgres@ub20-postgres-9:~/sbtm$ cp /tmp/sysbench-tpcc-master/* .
postgres@ub20-postgres-9:~/sbtm$ chmod 764 ./*
```

- создал БД и пользователя для теста
```
postgres=# create database sbtest;
CREATE DATABASE
postgres=# create role sbuser superuser login password 'sbuser';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE sbtest TO sbuser;
GRANT
postgres=# ALTER DATABASE sbtest OWNER TO sbuser;
ALTER DATABASE
postgres=# \l+
                                                                List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+---------+---------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 7877 kB | pg_default | default administrative connection database
 sbtest    | sbuser   | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/sbuser           +| 7877 kB | pg_default |
           |          |          |         |         | sbuser=CTc/sbuser     |         |            |
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 7729 kB | pg_default | unmodifiable empty database
           |          |          |         |         | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 7877 kB | pg_default | default template for new databases
           |          |          |         |         | postgres=CTc/postgres |         |            |
(4 rows)

```

##### Результат:

 5.1. результаты теста до изменения настроек (прогон 10 минут):
```
[ 600s ] thds: 2 tps: 114.10 qps: 3276.10 (r/w/o: 1494.20/1549.10/232.80) lat (ms,95%): 65.65 err/s 2.75 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            550546
        write:                           571039
        other:                           86082
        total:                           1207667
    transactions:                        42346  (70.57 per sec.)
    queries:                             1207667 (2012.59 per sec.)
    ignored errors:                      854    (1.42 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.0551s
    total number of events:              42346

Latency (ms):
         min:                                    0.61
         avg:                                   28.34
         max:                                 2727.71
         95th percentile:                      153.02
         sum:                              1199929.87

Threads fairness:
    events (avg/stddev):           21173.0000/54.00
    execution time (avg/stddev):   599.9649/0.02

```

 5.2. результаты теста после  изменения настроек:
```
postgres@ub20-postgres-9:~/sbtm$ ./tpcc.lua --pgsql-user=sbuser --pgsql-password='sbuser' --pgsql-db=sbtest --time=600 --threads=2 --report-interval=20 --tables=10 --scale=1 --use_fk=0 --db-driver=pgsql run
sysbench 1.0.20 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 2
Report intermediate results every 20 second(s)
Initializing random number generator from current time


Initializing worker threads...

DB SCHEMA public
DB SCHEMA public
Threads started!

[ 20s ] thds: 2 tps: 18.40 qps: 523.64 (r/w/o: 238.67/247.47/37.50) lat (ms,95%): 331.91 err/s 0.35 reconn/s: 0.00
[ 40s ] thds: 2 tps: 13.20 qps: 386.96 (r/w/o: 175.61/184.76/26.60) lat (ms,95%): 404.61 err/s 0.10 reconn/s: 0.00
[ 60s ] thds: 2 tps: 16.40 qps: 477.30 (r/w/o: 218.45/225.75/33.10) lat (ms,95%): 337.94 err/s 0.30 reconn/s: 0.00
[ 80s ] thds: 2 tps: 22.50 qps: 647.95 (r/w/o: 295.20/306.95/45.80) lat (ms,95%): 262.64 err/s 0.50 reconn/s: 0.00
[ 100s ] thds: 2 tps: 30.20 qps: 845.95 (r/w/o: 386.00/399.05/60.90) lat (ms,95%): 193.38 err/s 0.40 reconn/s: 0.00
[ 120s ] thds: 2 tps: 48.35 qps: 1358.31 (r/w/o: 617.85/642.35/98.10) lat (ms,95%): 118.92 err/s 0.95 reconn/s: 0.00
[ 140s ] thds: 2 tps: 63.95 qps: 1858.25 (r/w/o: 847.25/881.70/129.30) lat (ms,95%): 102.97 err/s 0.80 reconn/s: 0.00
[ 160s ] thds: 2 tps: 87.25 qps: 2424.69 (r/w/o: 1105.49/1142.09/177.10) lat (ms,95%): 77.19 err/s 1.70 reconn/s: 0.00
[ 180s ] thds: 2 tps: 128.45 qps: 3680.77 (r/w/o: 1678.81/1742.46/259.50) lat (ms,95%): 51.02 err/s 1.85 reconn/s: 0.00
[ 200s ] thds: 2 tps: 184.25 qps: 5331.80 (r/w/o: 2430.70/2527.00/374.10) lat (ms,95%): 36.24 err/s 3.65 reconn/s: 0.00
[ 220s ] thds: 2 tps: 212.05 qps: 5988.49 (r/w/o: 2724.35/2833.85/430.30) lat (ms,95%): 29.19 err/s 3.80 reconn/s: 0.00
[ 240s ] thds: 2 tps: 227.65 qps: 6488.39 (r/w/o: 2961.29/3065.99/461.10) lat (ms,95%): 25.28 err/s 3.80 reconn/s: 0.00
[ 260s ] thds: 2 tps: 250.80 qps: 7270.09 (r/w/o: 3320.62/3441.07/508.40) lat (ms,95%): 21.11 err/s 4.50 reconn/s: 0.00
[ 280s ] thds: 2 tps: 262.55 qps: 7540.79 (r/w/o: 3441.97/3566.02/532.80) lat (ms,95%): 19.65 err/s 4.95 reconn/s: 0.00
[ 300s ] thds: 2 tps: 244.65 qps: 6969.68 (r/w/o: 3177.99/3296.59/495.10) lat (ms,95%): 22.28 err/s 4.20 reconn/s: 0.00
[ 320s ] thds: 2 tps: 184.42 qps: 5461.21 (r/w/o: 2492.84/2594.44/373.94) lat (ms,95%): 29.72 err/s 3.34 reconn/s: 0.00
[ 340s ] thds: 2 tps: 114.40 qps: 3290.11 (r/w/o: 1498.85/1559.35/231.90) lat (ms,95%): 134.90 err/s 2.30 reconn/s: 0.00
[ 360s ] thds: 2 tps: 105.35 qps: 3048.94 (r/w/o: 1390.10/1445.55/213.30) lat (ms,95%): 137.35 err/s 1.55 reconn/s: 0.00
[ 380s ] thds: 2 tps: 78.05 qps: 2270.16 (r/w/o: 1032.00/1080.45/157.70) lat (ms,95%): 150.29 err/s 1.05 reconn/s: 0.00
[ 400s ] thds: 2 tps: 108.45 qps: 3147.34 (r/w/o: 1434.89/1493.14/219.30) lat (ms,95%): 134.90 err/s 1.90 reconn/s: 0.00
[ 420s ] thds: 2 tps: 110.25 qps: 3183.56 (r/w/o: 1450.11/1510.46/223.00) lat (ms,95%): 134.90 err/s 1.90 reconn/s: 0.00
[ 440s ] thds: 2 tps: 96.35 qps: 2769.75 (r/w/o: 1261.10/1314.05/194.60) lat (ms,95%): 137.35 err/s 1.30 reconn/s: 0.00
[ 460s ] thds: 2 tps: 137.70 qps: 4025.50 (r/w/o: 1833.40/1914.10/278.00) lat (ms,95%): 127.81 err/s 1.95 reconn/s: 0.00
[ 480s ] thds: 2 tps: 153.75 qps: 4441.10 (r/w/o: 2027.05/2102.45/311.60) lat (ms,95%): 127.81 err/s 2.80 reconn/s: 0.00
[ 500s ] thds: 2 tps: 153.30 qps: 4476.60 (r/w/o: 2042.05/2123.95/310.60) lat (ms,95%): 127.81 err/s 2.45 reconn/s: 0.00
[ 520s ] thds: 2 tps: 151.35 qps: 4338.64 (r/w/o: 1975.00/2057.55/306.10) lat (ms,95%): 127.81 err/s 2.45 reconn/s: 0.00
[ 540s ] thds: 2 tps: 136.50 qps: 3992.62 (r/w/o: 1816.21/1900.11/276.30) lat (ms,95%): 127.81 err/s 2.25 reconn/s: 0.00
[ 560s ] thds: 2 tps: 162.95 qps: 4591.90 (r/w/o: 2091.50/2169.40/331.00) lat (ms,95%): 25.74 err/s 3.20 reconn/s: 0.00
[ 580s ] thds: 2 tps: 157.05 qps: 4484.75 (r/w/o: 2047.00/2119.95/317.80) lat (ms,95%): 127.81 err/s 2.05 reconn/s: 0.00
[ 600s ] thds: 2 tps: 138.10 qps: 3959.58 (r/w/o: 1804.69/1874.89/280.00) lat (ms,95%): 127.81 err/s 2.45 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            996434
        write:                           1035362
        other:                           153910
        total:                           2185706
    transactions:                        75980  (126.62 per sec.)
    queries:                             2185706 (3642.58 per sec.)
    ignored errors:                      1296   (2.16 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.0421s
    total number of events:              75980

Latency (ms):
         min:                                    0.55
         avg:                                   15.79
         max:                                 1926.43
         95th percentile:                       63.32
         sum:                              1199784.93

Threads fairness:
    events (avg/stddev):           37990.0000/105.00
    execution time (avg/stddev):   599.8925/0.01

```

Как видно в результате изменения параметров конфигурации произошло увеличение средних и абсолютных показателей как по количеству транзакций/запросов так и по операциям чтения/записи приблизительно в 2 раза


#### 6. написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
##### Результат:

- поменяем настройки в файле ` /etc/postgresql/13/main/postgresql.conf` изменял следующие параметры:
```
max_connections = 100  # оставил как рекомендует pgtune  ну у меня все равно их меньше так что этот параметр не повлиял на результаты теста
shared_buffers = 1GB # рекомендуют делать 1/4 от доступной ОЗУ
effective_cache_size = 2GB # рекомендуют делать 2/3 от доступной ОЗУ но так как в целых числах lkz wtktq yfituj ntcnf
maintenance_work_mem = 256MB # рекомендуют делать 75 процентов от размера самой большой таблицы или индекса у меня в базе таблицы имеют размер 40 Мб максимум, но так как я не знаю как работает тесть и будет ли прирост то оставлю рекомендованное pgtune значение

checkpoint_timeout = 15min # Увеличил для минимизации влияния чекпоинта, вместе со следующим параметром это должно растянуть контрольную точку во времени, что по идее должно снизить нагрузку на систему
checkpoint_completion_target = 0.9
wal_buffers = 16MB # согласно документации увеличение буфера позволит лучше работать с большими транзакциями.
default_statistics_target = 100
random_page_cost = 2 # не рекомендубт делать меньше 2 сделал 2
effective_io_concurrency = 200
work_mem = 10240kB #5242kB дадим по больше раз уж мы количество фоновых по рекомендации от PGtune уменьшили
min_wal_size = 1GB
max_wal_size = 4GB
# Значения ниже увеличил в 8 раз от рекомендованных
max_worker_processes = 16 
max_parallel_workers_per_gather = 8
max_parallel_workers = 16
max_parallel_maintenance_workers = 8

# Параметры ниже отключают принудитульную синхронизацию с диском, это опасно в плане сохранности данных и восстановления при сбое но мы сегодня "храбрые" так как нам явно сказали так сделать
fsync=off
synchronous_commit=off
full_page_writes=off

```

- Удалось получить выигрыш в tps приблизительно в 20-30 процентов:
```
[ 600s ] thds: 8 tps: 119.30 qps: 3490.85 (r/w/o: 1586.10/1645.15/259.60) lat (ms,95%): 277.21 err/s 11.10 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            1218000
        write:                           1260308
        other:                           200438
        total:                           2678746
    transactions:                        92322  (153.76 per sec.)
    queries:                             2678746 (4461.39 per sec.)
    ignored errors:                      8269   (13.77 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.4264s
    total number of events:              92322

Latency (ms):
         min:                                    0.75
         avg:                                   52.00
         max:                                 1973.86
         95th percentile:                      189.93
         sum:                              4800748.84

Threads fairness:
    events (avg/stddev):           11540.2500/166.73
    execution time (avg/stddev):   600.0936/0.13

```

Среднее tps 153.76 per sec.
