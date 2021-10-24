# Домашнее задание

## Настройка autovacuum с учетом оптимальной производительности

## Цель:
1. запустить нагрузочный тест pgbench
2. настроить параметры autovacuum для достижения максимального уровня устойчивой производительности

### Задание и ход выполнения по этапам: 

#### 1. создать GCE инстанс типа e2-medium и standard disk 10GB
##### Результат:
Готово
```
Created ...
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
ub20-postgres-6  us-central1-c  e2-medium                  10.128.0.9   35.239.129.94  RUNNING
```

#### 2. установить на него PostgreSQL 13 с дефолтными настройками
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

#### 3. применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
##### Результат:
Изменен файл `/etc/postgresql/13/main/postgresql.conf`

```
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB
```
перезапустил службу postgres 
```
Student@ub20-postgres-6:~$ sudo systemctl stop postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2021-10-23 14:25:51 UTC; 15s ago
   Main PID: 3426 (code=exited, status=0/SUCCESS)

Oct 23 14:00:46 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 14:00:46 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Oct 23 14:25:51 ub20-postgres-6 systemd[1]: postgresql.service: Succeeded.
Oct 23 14:25:51 ub20-postgres-6 systemd[1]: Stopped PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo systemctl start postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 14:26:17 UTC; 3s ago
    Process: 4835 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 4835 (code=exited, status=0/SUCCESS)

Oct 23 14:26:17 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 14:26:17 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$

```

#### 4. зайти под пользователем postgres - sudo su postgres
##### Результат:
Готово.
```
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$

```

#### 5. выполнить pgbench -i postgres
##### Результат:
Готово
```
postgres@ub20-postgres-6:~$ pgbench -i postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.10 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.50 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.31 s, vacuum 0.11 s, primary keys 0.08 s).
postgres@ub20-postgres-6:~$
```

#### 6. запустить pgbench -c8 -P 10 -T 600 -U postgres postgres
##### Результат:
```
progress: 160.0 s, 812.0 tps, lat 9.853 ms stddev 6.714
progress: 170.0 s, 818.8 tps, lat 9.769 ms stddev 6.422
progress: 180.0 s, 825.4 tps, lat 9.693 ms stddev 6.379
progress: 190.0 s, 817.9 tps, lat 9.778 ms stddev 6.499
progress: 200.0 s, 821.9 tps, lat 9.725 ms stddev 7.428
progress: 210.0 s, 816.2 tps, lat 9.806 ms stddev 7.666
progress: 220.0 s, 809.1 tps, lat 9.890 ms stddev 7.526
progress: 230.0 s, 812.7 tps, lat 9.842 ms stddev 7.548
progress: 240.0 s, 750.9 tps, lat 10.650 ms stddev 7.833
progress: 250.0 s, 738.4 tps, lat 10.835 ms stddev 8.089
progress: 260.0 s, 802.2 tps, lat 9.970 ms stddev 7.400
progress: 270.0 s, 807.3 tps, lat 9.910 ms stddev 7.430
progress: 280.0 s, 811.6 tps, lat 9.856 ms stddev 7.716
progress: 290.0 s, 800.9 tps, lat 9.987 ms stddev 7.455
progress: 300.0 s, 798.5 tps, lat 10.016 ms stddev 7.667
progress: 310.0 s, 819.2 tps, lat 9.763 ms stddev 7.490
progress: 320.0 s, 809.3 tps, lat 9.885 ms stddev 7.246
progress: 330.0 s, 799.4 tps, lat 10.004 ms stddev 7.724
progress: 340.0 s, 801.3 tps, lat 9.986 ms stddev 7.883
progress: 350.0 s, 814.3 tps, lat 9.821 ms stddev 7.406
progress: 360.0 s, 784.4 tps, lat 10.199 ms stddev 7.624
progress: 370.0 s, 826.6 tps, lat 9.679 ms stddev 7.624
progress: 380.0 s, 816.2 tps, lat 9.798 ms stddev 7.400
progress: 390.0 s, 801.0 tps, lat 9.990 ms stddev 7.710
progress: 400.0 s, 814.5 tps, lat 9.820 ms stddev 7.402
progress: 410.0 s, 811.9 tps, lat 9.855 ms stddev 7.396
progress: 420.0 s, 786.9 tps, lat 10.166 ms stddev 7.686
progress: 430.0 s, 567.8 tps, lat 14.042 ms stddev 19.281
progress: 440.0 s, 522.8 tps, lat 15.300 ms stddev 23.137
progress: 450.0 s, 511.3 tps, lat 15.630 ms stddev 23.568
progress: 460.0 s, 503.1 tps, lat 15.912 ms stddev 24.693
progress: 470.0 s, 517.6 tps, lat 15.472 ms stddev 22.187
progress: 480.0 s, 527.1 tps, lat 15.160 ms stddev 22.889
progress: 490.0 s, 506.7 tps, lat 15.820 ms stddev 23.773
progress: 500.0 s, 520.2 tps, lat 15.379 ms stddev 22.844
progress: 510.0 s, 530.9 tps, lat 15.069 ms stddev 22.480
progress: 520.0 s, 526.4 tps, lat 15.178 ms stddev 21.851
progress: 530.0 s, 524.0 tps, lat 15.301 ms stddev 23.092
progress: 540.0 s, 525.6 tps, lat 15.183 ms stddev 23.034
progress: 550.0 s, 522.2 tps, lat 15.356 ms stddev 23.484
progress: 560.0 s, 480.9 tps, lat 16.598 ms stddev 23.640
progress: 570.0 s, 497.9 tps, lat 16.043 ms stddev 24.190
progress: 580.0 s, 513.4 tps, lat 15.600 ms stddev 25.026
progress: 590.0 s, 527.7 tps, lat 15.176 ms stddev 23.552
progress: 600.0 s, 511.8 tps, lat 15.572 ms stddev 23.805
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 433728
latency average = 11.065 ms
latency stddev = 12.723 ms
tps = 722.851815 (including connections establishing)
tps = 722.855276 (excluding connections establishing)
postgres@ub20-postgres-6:~$

```

#### 7. дать отработать до конца
##### Результат:
```
...
progress: 600.0 s, 511.8 tps, lat 15.572 ms stddev 23.805
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 433728
latency average = 11.065 ms
latency stddev = 12.723 ms
tps = 722.851815 (including connections establishing)
tps = 722.855276 (excluding connections establishing)
postgres@ub20-postgres-6:~$

```
Имеем таблицы со следующими параметрами заполненности полезными данными в процентах в строчках tuple_percent:
```
postgres=# CREATE EXTENSION pgstattuple;
CREATE EXTENSION
postgres=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
(4 rows)

postgres=# select * from pgstattuple('pgbench_history') \gx
-[ RECORD 1 ]------+---------
table_len          | 22700032
tuple_count        | 433728
tuple_len          | 20818944
tuple_percent      | 91.71
dead_tuple_count   | 0
dead_tuple_len     | 0
dead_tuple_percent | 0
free_space         | 68588
free_percent       | 0.3

postgres=# select * from pgstattuple('pgbench_branches') \gx
-[ RECORD 1 ]------+-------
table_len          | 163840
tuple_count        | 1
tuple_len          | 32
tuple_percent      | 0.02
dead_tuple_count   | 0
dead_tuple_len     | 0
dead_tuple_percent | 0
free_space         | 143696
free_percent       | 87.71

postgres=# select * from pgstattuple('pgbench_accounts') \gx
-[ RECORD 1 ]------+---------
table_len          | 14376960
tuple_count        | 100000
tuple_len          | 12100000
tuple_percent      | 84.16
dead_tuple_count   | 2240
dead_tuple_len     | 271040
dead_tuple_percent | 1.89
free_space         | 414468
free_percent       | 2.88

postgres=# select * from pgstattuple('pgbench_tellers') \gx
-[ RECORD 1 ]------+------
table_len          | 57344
tuple_count        | 10
tuple_len          | 360
tuple_percent      | 0.63
dead_tuple_count   | 0
dead_tuple_len     | 0
dead_tuple_percent | 0
free_space         | 51624
free_percent       | 90.03

```

#### 8. дальше настроить autovacuum максимально эффективно
##### Результат:
Ну вот тут что-то больше похоже на стат погрешность чем на реальные результаты:

```

progress: 500.0 s, 498.6 tps, lat 16.041 ms stddev 23.626
progress: 510.0 s, 525.5 tps, lat 15.246 ms stddev 22.837
progress: 520.0 s, 523.9 tps, lat 15.266 ms stddev 23.098
progress: 530.0 s, 523.7 tps, lat 15.248 ms stddev 24.459
progress: 540.0 s, 527.6 tps, lat 15.173 ms stddev 23.789
progress: 550.0 s, 528.4 tps, lat 15.142 ms stddev 23.188
progress: 560.0 s, 539.5 tps, lat 14.817 ms stddev 22.820
progress: 570.0 s, 535.7 tps, lat 14.940 ms stddev 22.771
progress: 580.0 s, 503.2 tps, lat 15.893 ms stddev 24.198
progress: 590.0 s, 512.4 tps, lat 15.577 ms stddev 24.034
progress: 600.0 s, 519.9 tps, lat 15.424 ms stddev 23.620
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 420210
latency average = 11.421 ms
latency stddev = 13.338 ms
tps = 700.324803 (including connections establishing)
tps = 700.328414 (excluding connections establishing)
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl stop postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2021-10-23 17:06:37 UTC; 9s ago
    Process: 4835 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 4835 (code=exited, status=0/SUCCESS)

Oct 23 14:26:17 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 14:26:17 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Oct 23 17:06:37 ub20-postgres-6 systemd[1]: postgresql.service: Succeeded.
Oct 23 17:06:37 ub20-postgres-6 systemd[1]: Stopped PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo systemctl start postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 17:06:57 UTC; 3s ago
    Process: 6145 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 6145 (code=exited, status=0/SUCCESS)

Oct 23 17:06:57 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 17:06:57 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ ls -la 13/main/
total 92
drwx------ 19 postgres postgres 4096 Oct 23 17:06 .
drwxr-xr-x  3 postgres postgres 4096 Oct 23 14:00 ..
-rw-------  1 postgres postgres    3 Oct 23 14:00 PG_VERSION
drwx------  6 postgres postgres 4096 Oct 23 14:28 base
drwx------  2 postgres postgres 4096 Oct 23 17:07 global
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_commit_ts
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_dynshmem
drwx------  4 postgres postgres 4096 Oct 23 17:06 pg_logical
drwx------  4 postgres postgres 4096 Oct 23 14:00 pg_multixact
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_notify
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_replslot
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_serial
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_snapshots
drwx------  2 postgres postgres 4096 Oct 23 17:06 pg_stat
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_stat_tmp
drwx------  2 postgres postgres 4096 Oct 23 17:05 pg_subtrans
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_tblspc
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_twophase
drwx------  3 postgres postgres 4096 Oct 23 17:06 pg_wal
drwx------  2 postgres postgres 4096 Oct 23 16:46 pg_xact
-rw-------  1 postgres postgres   88 Oct 23 14:00 postgresql.auto.conf
-rw-------  1 postgres postgres  130 Oct 23 17:06 postmaster.opts
-rw-------  1 postgres postgres  108 Oct 23 17:06 postmaster.pid
postgres@ub20-postgres-6:~$ ls -la 13/main/pg_stat
total 8
drwx------  2 postgres postgres 4096 Oct 23 17:06 .
drwx------ 19 postgres postgres 4096 Oct 23 17:06 ..
postgres@ub20-postgres-6:~$ ls -la 13/main/pg_xact/
total 680
drwx------  2 postgres postgres   4096 Oct 23 16:46 .
drwx------ 19 postgres postgres   4096 Oct 23 17:06 ..
-rw-------  1 postgres postgres 262144 Oct 23 16:16 0000
-rw-------  1 postgres postgres 262144 Oct 23 16:46 0001
-rw-------  1 postgres postgres 163840 Oct 23 17:06 0002
postgres@ub20-postgres-6:~$ ls -la 13/main/pg_xact/0000
-rw------- 1 postgres postgres 262144 Oct 23 16:16 13/main/pg_xact/0000
postgres@ub20-postgres-6:~$ ls -la 13/main/pg_logical/
total 20
drwx------  4 postgres postgres 4096 Oct 23 17:06 .
drwx------ 19 postgres postgres 4096 Oct 23 17:06 ..
drwx------  2 postgres postgres 4096 Oct 23 14:00 mappings
-rw-------  1 postgres postgres    8 Oct 23 17:06 replorigin_checkpoint
drwx------  2 postgres postgres 4096 Oct 23 14:00 snapshots
postgres@ub20-postgres-6:~$ ls -la /etc/postgresql/13/main/
total 64
drwxr-xr-x 3 postgres postgres  4096 Oct 23 16:52 .
drwxr-xr-x 3 postgres postgres  4096 Oct 23 14:00 ..
drwxr-xr-x 2 postgres postgres  4096 Oct 23 14:00 conf.d
-rw-r--r-- 1 postgres postgres   315 Oct 23 14:00 environment
-rw-r--r-- 1 postgres postgres   143 Oct 23 14:00 pg_ctl.conf
-rw-r----- 1 postgres postgres  4933 Oct 23 14:00 pg_hba.conf
-rw-r----- 1 postgres postgres  1636 Oct 23 14:00 pg_ident.conf
-rw-r--r-- 1 postgres postgres 28343 Oct 23 16:52 postgresql.conf
-rw-r--r-- 1 postgres postgres   317 Oct 23 14:00 start.conf
postgres@ub20-postgres-6:~$ ls -la /etc/postgresql/13/
total 12
drwxr-xr-x 3 postgres postgres 4096 Oct 23 14:00 .
drwxr-xr-x 3 postgres postgres 4096 Oct 23 14:00 ..
drwxr-xr-x 3 postgres postgres 4096 Oct 23 16:52 main
postgres@ub20-postgres-6:~$ ls -la 13/main/
total 92
drwx------ 19 postgres postgres 4096 Oct 23 17:06 .
drwxr-xr-x  3 postgres postgres 4096 Oct 23 14:00 ..
-rw-------  1 postgres postgres    3 Oct 23 14:00 PG_VERSION
drwx------  6 postgres postgres 4096 Oct 23 14:28 base
drwx------  2 postgres postgres 4096 Oct 23 17:07 global
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_commit_ts
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_dynshmem
drwx------  4 postgres postgres 4096 Oct 23 17:06 pg_logical
drwx------  4 postgres postgres 4096 Oct 23 14:00 pg_multixact
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_notify
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_replslot
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_serial
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_snapshots
drwx------  2 postgres postgres 4096 Oct 23 17:06 pg_stat
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_stat_tmp
drwx------  2 postgres postgres 4096 Oct 23 17:05 pg_subtrans
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_tblspc
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_twophase
drwx------  3 postgres postgres 4096 Oct 23 17:06 pg_wal
drwx------  2 postgres postgres 4096 Oct 23 16:46 pg_xact
-rw-------  1 postgres postgres   88 Oct 23 14:00 postgresql.auto.conf
-rw-------  1 postgres postgres  130 Oct 23 17:06 postmaster.opts
-rw-------  1 postgres postgres  108 Oct 23 17:06 postmaster.pid
postgres@ub20-postgres-6:~$ cat 13/main/postgresql.auto.conf |less
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl stop postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2021-10-23 17:18:19 UTC; 5s ago
    Process: 6145 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 6145 (code=exited, status=0/SUCCESS)

Oct 23 17:06:57 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 17:06:57 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Oct 23 17:18:19 ub20-postgres-6 systemd[1]: postgresql.service: Succeeded.
Oct 23 17:18:19 ub20-postgres-6 systemd[1]: Stopped PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo systemctl start postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 17:18:31 UTC; 2s ago
    Process: 6298 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 6298 (code=exited, status=0/SUCCESS)

Oct 23 17:18:31 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 17:18:31 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
Error: invalid line 426 in /etc/postgresql/13/main/postgresql.conf: log_destination = 'stderr','csvlog'         # Valid values are combinations of
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl stop postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2021-10-23 17:20:21 UTC; 2s ago
    Process: 6298 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 6298 (code=exited, status=0/SUCCESS)

Oct 23 17:18:31 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 17:18:31 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Oct 23 17:20:21 ub20-postgres-6 systemd[1]: postgresql.service: Succeeded.
Oct 23 17:20:21 ub20-postgres-6 systemd[1]: Stopped PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo systemctl start postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 17:20:31 UTC; 3s ago
    Process: 6346 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 6346 (code=exited, status=0/SUCCESS)

Oct 23 17:20:31 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 17:20:31 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 790.7 tps, lat 10.075 ms stddev 7.640
progress: 20.0 s, 815.1 tps, lat 9.818 ms stddev 7.284
progress: 30.0 s, 806.2 tps, lat 9.920 ms stddev 7.452
progress: 40.0 s, 800.3 tps, lat 9.995 ms stddev 7.130
progress: 50.0 s, 809.8 tps, lat 9.881 ms stddev 7.284
progress: 60.0 s, 795.0 tps, lat 10.062 ms stddev 7.261
progress: 70.0 s, 798.7 tps, lat 10.013 ms stddev 7.142
progress: 80.0 s, 809.1 tps, lat 9.888 ms stddev 6.838
progress: 90.0 s, 798.6 tps, lat 10.016 ms stddev 7.105
progress: 100.0 s, 783.1 tps, lat 10.216 ms stddev 7.147
progress: 110.0 s, 785.5 tps, lat 10.185 ms stddev 6.752
progress: 120.0 s, 789.8 tps, lat 10.126 ms stddev 6.685
progress: 130.0 s, 796.9 tps, lat 10.038 ms stddev 6.671
progress: 140.0 s, 812.2 tps, lat 9.851 ms stddev 6.368
progress: 150.0 s, 808.2 tps, lat 9.897 ms stddev 6.540
progress: 160.0 s, 807.4 tps, lat 9.908 ms stddev 6.559
progress: 170.0 s, 807.9 tps, lat 9.900 ms stddev 6.747
progress: 180.0 s, 792.2 tps, lat 10.100 ms stddev 6.502
progress: 190.0 s, 805.1 tps, lat 9.935 ms stddev 6.285
progress: 200.0 s, 809.6 tps, lat 9.879 ms stddev 7.332
progress: 210.0 s, 789.9 tps, lat 10.127 ms stddev 7.682
progress: 220.0 s, 749.1 tps, lat 10.679 ms stddev 8.069
progress: 230.0 s, 768.8 tps, lat 10.404 ms stddev 7.778
progress: 240.0 s, 811.1 tps, lat 9.863 ms stddev 7.347
progress: 250.0 s, 811.8 tps, lat 9.852 ms stddev 7.390
progress: 260.0 s, 772.5 tps, lat 10.356 ms stddev 7.624
progress: 270.0 s, 749.1 tps, lat 10.673 ms stddev 7.819
progress: 280.0 s, 732.0 tps, lat 10.933 ms stddev 8.112
progress: 290.0 s, 767.4 tps, lat 10.424 ms stddev 7.675
progress: 300.0 s, 762.7 tps, lat 10.487 ms stddev 7.764
progress: 310.0 s, 777.2 tps, lat 10.292 ms stddev 7.728
progress: 320.0 s, 791.1 tps, lat 10.111 ms stddev 7.681
progress: 330.0 s, 779.7 tps, lat 10.257 ms stddev 7.568
progress: 340.0 s, 794.5 tps, lat 10.068 ms stddev 7.548
progress: 350.0 s, 803.4 tps, lat 9.959 ms stddev 7.387
progress: 360.0 s, 775.8 tps, lat 10.310 ms stddev 7.999
progress: 370.0 s, 797.8 tps, lat 10.023 ms stddev 7.509
progress: 380.0 s, 801.3 tps, lat 9.988 ms stddev 7.751
progress: 390.0 s, 802.1 tps, lat 9.972 ms stddev 7.461
progress: 400.0 s, 799.6 tps, lat 10.004 ms stddev 7.490
progress: 410.0 s, 815.3 tps, lat 9.813 ms stddev 7.535
progress: 420.0 s, 539.0 tps, lat 14.720 ms stddev 22.838
progress: 430.0 s, 532.5 tps, lat 15.033 ms stddev 23.034
progress: 440.0 s, 528.9 tps, lat 15.109 ms stddev 22.969
progress: 450.0 s, 523.2 tps, lat 15.300 ms stddev 23.637
progress: 460.0 s, 526.0 tps, lat 15.192 ms stddev 22.765
progress: 470.0 s, 534.8 tps, lat 14.950 ms stddev 22.923
progress: 480.0 s, 495.6 tps, lat 16.132 ms stddev 25.465
progress: 490.0 s, 529.0 tps, lat 15.133 ms stddev 23.046
progress: 500.0 s, 527.1 tps, lat 15.174 ms stddev 23.051
progress: 510.0 s, 513.6 tps, lat 15.558 ms stddev 23.817
progress: 520.0 s, 519.9 tps, lat 15.388 ms stddev 23.610
progress: 530.0 s, 503.8 tps, lat 15.889 ms stddev 23.668
progress: 540.0 s, 491.8 tps, lat 16.249 ms stddev 24.248
progress: 550.0 s, 511.6 tps, lat 15.640 ms stddev 23.931
progress: 560.0 s, 485.5 tps, lat 16.477 ms stddev 25.467
progress: 570.0 s, 499.2 tps, lat 16.026 ms stddev 24.427
progress: 580.0 s, 515.4 tps, lat 15.546 ms stddev 23.680
progress: 590.0 s, 516.9 tps, lat 15.453 ms stddev 23.533
progress: 600.0 s, 507.3 tps, lat 15.788 ms stddev 23.962
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 422758
latency average = 11.352 ms
latency stddev = 13.287 ms
tps = 704.566444 (including connections establishing)
tps = 704.570678 (excluding connections establishing)
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ ls -la 13/main/
total 100
drwx------ 20 postgres postgres 4096 Oct 23 17:20 .
drwxr-xr-x  3 postgres postgres 4096 Oct 23 14:00 ..
-rw-------  1 postgres postgres    3 Oct 23 14:00 PG_VERSION
drwx------  6 postgres postgres 4096 Oct 23 14:28 base
-rw-------  1 postgres postgres   44 Oct 23 17:20 current_logfiles
drwx------  2 postgres postgres 4096 Oct 23 17:25 global
drwx------  2 postgres postgres 4096 Oct 23 17:20 log
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_commit_ts
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_dynshmem
drwx------  4 postgres postgres 4096 Oct 23 17:34 pg_logical
drwx------  4 postgres postgres 4096 Oct 23 14:00 pg_multixact
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_notify
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_replslot
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_serial
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_snapshots
drwx------  2 postgres postgres 4096 Oct 23 17:20 pg_stat
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_stat_tmp
drwx------  2 postgres postgres 4096 Oct 23 17:29 pg_subtrans
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_tblspc
drwx------  2 postgres postgres 4096 Oct 23 14:00 pg_twophase
drwx------  3 postgres postgres 4096 Oct 23 17:34 pg_wal
drwx------  2 postgres postgres 4096 Oct 23 17:30 pg_xact
-rw-------  1 postgres postgres   88 Oct 23 14:00 postgresql.auto.conf
-rw-------  1 postgres postgres  130 Oct 23 17:20 postmaster.opts
-rw-------  1 postgres postgres  108 Oct 23 17:20 postmaster.pid
postgres@ub20-postgres-6:~$ ls -la 13/main/log/
total 88
drwx------  2 postgres postgres  4096 Oct 23 17:20 .
drwx------ 20 postgres postgres  4096 Oct 23 17:20 ..
-rw-------  1 postgres postgres 77454 Oct 23 17:30 postgresql-2021-10-23_172029.log
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_172029.log |less
postgres@ub20-postgres-6:~$ man grep
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_172029.log |less
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_172029.log |grep -c 'automatic vacuum'
110
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_172029.log |grep 'automatic vacuum'
2021-10-23 17:20:59.996 UTC [6369] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:21:00.012 UTC [6369] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:21:00.026 UTC [6369] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:21:19.954 UTC [6371] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:21:19.968 UTC [6371] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:21:19.983 UTC [6371] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:21:34.976 UTC [6373] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:21:34.989 UTC [6373] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:21:35.003 UTC [6373] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:21:35.074 UTC [6373] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:21:49.954 UTC [6375] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:21:49.967 UTC [6375] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:21:49.982 UTC [6375] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:22:04.977 UTC [6377] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:22:04.989 UTC [6377] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:22:05.004 UTC [6377] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:22:05.112 UTC [6377] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:22:19.979 UTC [6380] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:22:19.992 UTC [6380] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:22:34.978 UTC [6388] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:22:34.991 UTC [6388] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:22:35.006 UTC [6388] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:22:35.180 UTC [6388] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:22:49.964 UTC [6390] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:22:49.977 UTC [6390] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:22:49.989 UTC [6390] LOG:  automatic vacuum of table "postgres.pg_catalog.pg_statistic": index scans: 0
2021-10-23 17:22:59.958 UTC [6398] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:22:59.971 UTC [6398] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:22:59.988 UTC [6398] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:23:00.188 UTC [6398] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:23:19.974 UTC [6401] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:23:19.987 UTC [6401] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:23:34.975 UTC [6403] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:23:34.989 UTC [6403] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:23:35.003 UTC [6403] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:23:49.981 UTC [6405] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:23:49.994 UTC [6405] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:23:50.012 UTC [6405] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:23:59.966 UTC [6407] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:23:59.979 UTC [6407] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:24:14.987 UTC [6409] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:24:14.999 UTC [6409] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:24:15.359 UTC [6409] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:24:29.980 UTC [6411] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:24:29.993 UTC [6411] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:24:49.988 UTC [6413] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:24:50.002 UTC [6413] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:24:50.024 UTC [6413] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:24:50.460 UTC [6413] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:25:05.028 UTC [6415] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:25:05.042 UTC [6415] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:25:20.029 UTC [6421] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:25:20.043 UTC [6421] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:25:20.500 UTC [6421] LOG:  automatic vacuum of table "postgres.pg_catalog.pg_statistic": index scans: 0
2021-10-23 17:25:20.514 UTC [6421] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:25:35.011 UTC [6423] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:25:35.024 UTC [6423] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:25:50.023 UTC [6425] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:25:50.037 UTC [6425] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:25:50.050 UTC [6425] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:26:05.051 UTC [6427] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:26:05.064 UTC [6427] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:26:05.085 UTC [6427] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:26:15.050 UTC [6429] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:26:15.062 UTC [6429] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:26:35.052 UTC [6431] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:26:35.065 UTC [6431] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:26:35.077 UTC [6431] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:26:50.051 UTC [6433] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:26:50.065 UTC [6433] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:27:05.072 UTC [6435] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:27:05.085 UTC [6435] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:27:20.035 UTC [6437] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:27:20.048 UTC [6437] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:27:35.037 UTC [6439] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:27:35.050 UTC [6439] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:27:35.076 UTC [6439] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:27:35.606 UTC [6439] LOG:  automatic vacuum of table "postgres.pg_catalog.pg_statistic": index scans: 0
2021-10-23 17:27:35.616 UTC [6439] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:27:51.545 UTC [6441] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:27:51.646 UTC [6441] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:28:06.195 UTC [6443] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:28:06.209 UTC [6443] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:28:06.220 UTC [6443] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:28:21.545 UTC [6445] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:28:21.645 UTC [6445] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:28:36.195 UTC [6447] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:28:36.208 UTC [6447] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:28:51.545 UTC [6449] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:28:51.645 UTC [6449] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:29:06.245 UTC [6452] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:29:06.258 UTC [6452] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:29:15.266 UTC [6454] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:29:15.396 UTC [6454] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:29:36.245 UTC [6456] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:29:36.258 UTC [6456] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:29:51.545 UTC [6458] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:29:51.559 UTC [6458] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:30:06.245 UTC [6460] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:30:06.258 UTC [6460] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:30:06.293 UTC [6460] LOG:  automatic vacuum of table "postgres.public.pgbench_history": index scans: 0
2021-10-23 17:30:07.461 UTC [6460] LOG:  automatic vacuum of table "postgres.pg_toast.pg_toast_2619": index scans: 1
2021-10-23 17:30:21.545 UTC [6463] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:30:21.645 UTC [6463] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:30:36.245 UTC [6465] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:30:36.258 UTC [6465] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:30:51.695 UTC [6467] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:30:51.709 UTC [6467] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
2021-10-23 17:30:59.987 UTC [6469] LOG:  automatic vacuum of table "postgres.public.pgbench_branches": index scans: 1
2021-10-23 17:31:00.000 UTC [6469] LOG:  automatic vacuum of table "postgres.public.pgbench_tellers": index scans: 0
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_172029.log |less
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ cp 13/main/log/postgresql-2021-10-23_172029.log ./1.log
postgres@ub20-postgres-6:~$ ls -la
total 124
drwxr-xr-x  4 postgres postgres  4096 Oct 23 18:09 .
drwxr-xr-x 38 root     root      4096 Oct 23 14:00 ..
-rw-------  1 postgres postgres  1083 Oct 23 17:20 .bash_history
drwx------  3 postgres postgres  4096 Oct 23 14:30 .config
-rw-------  1 postgres postgres    59 Oct 23 17:59 .lesshst
-rw-------  1 postgres postgres  5708 Oct 23 17:22 .psql_history
-rw-------  1 postgres postgres 15138 Oct 23 18:09 .viminfo
-rw-------  1 postgres postgres 77454 Oct 23 18:09 1.log
drwxr-xr-x  3 postgres postgres  4096 Oct 23 14:00 13
postgres@ub20-postgres-6:~$ vim 13/main/log/postgresql-2021-10-23_172029.log
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 18:11:16 UTC; 3s ago
    Process: 6894 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 6894 (code=exited, status=0/SUCCESS)

Oct 23 18:11:16 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 18:11:16 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 789.8 tps, lat 10.100 ms stddev 7.500
progress: 20.0 s, 713.3 tps, lat 11.212 ms stddev 7.834
progress: 30.0 s, 760.5 tps, lat 10.516 ms stddev 7.472
progress: 40.0 s, 776.9 tps, lat 10.293 ms stddev 7.343
progress: 50.0 s, 749.1 tps, lat 10.683 ms stddev 7.240
progress: 60.0 s, 750.8 tps, lat 10.655 ms stddev 7.307
progress: 70.0 s, 777.7 tps, lat 10.283 ms stddev 6.928
progress: 80.0 s, 802.0 tps, lat 9.971 ms stddev 6.893
progress: 90.0 s, 814.7 tps, lat 9.823 ms stddev 6.895
progress: 100.0 s, 788.9 tps, lat 10.141 ms stddev 6.943
progress: 110.0 s, 804.1 tps, lat 9.939 ms stddev 6.568
progress: 120.0 s, 803.0 tps, lat 9.966 ms stddev 7.179
progress: 130.0 s, 828.0 tps, lat 9.662 ms stddev 6.298
progress: 140.0 s, 781.2 tps, lat 10.240 ms stddev 7.760
progress: 150.0 s, 811.9 tps, lat 9.853 ms stddev 7.525
progress: 160.0 s, 817.4 tps, lat 9.786 ms stddev 7.610
progress: 170.0 s, 792.3 tps, lat 10.096 ms stddev 7.631
progress: 180.0 s, 753.3 tps, lat 10.615 ms stddev 7.805
progress: 190.0 s, 798.3 tps, lat 10.018 ms stddev 7.326
progress: 200.0 s, 809.0 tps, lat 9.890 ms stddev 7.544
progress: 210.0 s, 807.2 tps, lat 9.913 ms stddev 7.164
progress: 220.0 s, 805.2 tps, lat 9.934 ms stddev 7.569
progress: 230.0 s, 782.8 tps, lat 10.219 ms stddev 7.903
progress: 240.0 s, 789.1 tps, lat 10.134 ms stddev 7.857
progress: 250.0 s, 778.8 tps, lat 10.271 ms stddev 7.835
progress: 260.0 s, 732.5 tps, lat 10.920 ms stddev 8.136
progress: 270.0 s, 739.9 tps, lat 10.802 ms stddev 10.745
progress: 280.0 s, 771.2 tps, lat 10.383 ms stddev 7.944
progress: 290.0 s, 788.7 tps, lat 10.142 ms stddev 7.693
progress: 300.0 s, 795.6 tps, lat 10.052 ms stddev 7.590
progress: 310.0 s, 787.6 tps, lat 10.159 ms stddev 7.836
progress: 320.0 s, 788.3 tps, lat 10.148 ms stddev 7.435
progress: 330.0 s, 737.3 tps, lat 10.844 ms stddev 8.084
progress: 340.0 s, 738.9 tps, lat 10.825 ms stddev 7.846
progress: 350.0 s, 738.2 tps, lat 10.840 ms stddev 8.060
progress: 360.0 s, 761.6 tps, lat 10.501 ms stddev 7.759
progress: 370.0 s, 771.0 tps, lat 10.376 ms stddev 7.566
progress: 380.0 s, 778.4 tps, lat 10.277 ms stddev 7.335
progress: 390.0 s, 767.3 tps, lat 10.425 ms stddev 7.609
progress: 400.0 s, 795.5 tps, lat 10.056 ms stddev 7.485
progress: 410.0 s, 790.5 tps, lat 10.121 ms stddev 7.571
progress: 420.0 s, 787.3 tps, lat 10.158 ms stddev 7.514
progress: 430.0 s, 791.9 tps, lat 10.103 ms stddev 7.650
progress: 440.0 s, 795.8 tps, lat 10.051 ms stddev 7.604
progress: 450.0 s, 549.1 tps, lat 14.568 ms stddev 21.585
progress: 460.0 s, 534.9 tps, lat 14.953 ms stddev 22.567
progress: 470.0 s, 521.3 tps, lat 15.343 ms stddev 23.172
progress: 480.0 s, 520.9 tps, lat 15.361 ms stddev 22.163
progress: 490.0 s, 515.1 tps, lat 15.509 ms stddev 24.269
progress: 500.0 s, 532.5 tps, lat 15.024 ms stddev 23.067
progress: 510.0 s, 531.9 tps, lat 15.007 ms stddev 22.922
progress: 520.0 s, 537.0 tps, lat 14.929 ms stddev 22.288
progress: 530.0 s, 534.1 tps, lat 14.970 ms stddev 23.143
progress: 540.0 s, 536.0 tps, lat 14.903 ms stddev 22.473
progress: 550.0 s, 534.5 tps, lat 14.991 ms stddev 22.815
progress: 560.0 s, 532.4 tps, lat 15.023 ms stddev 23.471
progress: 570.0 s, 528.9 tps, lat 15.125 ms stddev 23.042
progress: 580.0 s, 519.1 tps, lat 15.409 ms stddev 24.277
progress: 590.0 s, 524.2 tps, lat 15.248 ms stddev 22.993
progress: 600.0 s, 530.1 tps, lat 15.086 ms stddev 22.812
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 428256
latency average = 11.206 ms
latency stddev = 12.421 ms
tps = 713.731558 (including connections establishing)
tps = 713.734963 (excluding connections establishing)
postgres@ub20-postgres-6:~$ vim 13/main/log/postgresql-2021-10-23_172029.log
postgres@ub20-postgres-6:~$ ls -la 13/main/log/
total 116
drwx------  2 postgres postgres   4096 Oct 23 18:22 .
drwx------ 20 postgres postgres   4096 Oct 23 18:11 ..
-rw-------  1 postgres postgres      0 Oct 23 18:10 postgresql-2021-10-23_172029.log
-rw-------  1 postgres postgres 105640 Oct 23 18:22 postgresql-2021-10-23_181114.log
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_181114.log |less
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_181114.log |less
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 18:58:08 UTC; 3s ago
    Process: 7368 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 7368 (code=exited, status=0/SUCCESS)

Oct 23 18:58:08 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 18:58:08 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 706.6 tps, lat 11.279 ms stddev 8.294
progress: 20.0 s, 739.1 tps, lat 10.820 ms stddev 7.906
progress: 30.0 s, 727.7 tps, lat 10.996 ms stddev 8.177
progress: 40.0 s, 732.6 tps, lat 10.917 ms stddev 7.874
progress: 50.0 s, 708.3 tps, lat 11.295 ms stddev 7.892
progress: 60.0 s, 674.3 tps, lat 11.862 ms stddev 8.690
progress: 70.0 s, 715.6 tps, lat 11.181 ms stddev 7.630
progress: 80.0 s, 731.5 tps, lat 10.935 ms stddev 7.277
progress: 90.0 s, 741.8 tps, lat 10.774 ms stddev 7.322
progress: 100.0 s, 711.3 tps, lat 11.256 ms stddev 7.527
progress: 110.0 s, 703.7 tps, lat 11.361 ms stddev 7.651
progress: 120.0 s, 746.6 tps, lat 10.720 ms stddev 6.868
progress: 130.0 s, 703.4 tps, lat 11.368 ms stddev 7.351
progress: 140.0 s, 736.4 tps, lat 10.864 ms stddev 7.282
progress: 150.0 s, 736.4 tps, lat 10.865 ms stddev 7.585
progress: 160.0 s, 745.5 tps, lat 10.729 ms stddev 6.417
progress: 170.0 s, 712.2 tps, lat 11.233 ms stddev 7.090
progress: 180.0 s, 744.8 tps, lat 10.740 ms stddev 6.941
progress: 190.0 s, 720.9 tps, lat 11.093 ms stddev 6.498
progress: 200.0 s, 728.4 tps, lat 10.982 ms stddev 7.048
progress: 210.0 s, 727.0 tps, lat 11.006 ms stddev 8.126
progress: 220.0 s, 728.7 tps, lat 10.974 ms stddev 8.301
progress: 230.0 s, 709.1 tps, lat 11.282 ms stddev 8.164
progress: 240.0 s, 740.7 tps, lat 10.800 ms stddev 7.989
progress: 250.0 s, 717.4 tps, lat 11.152 ms stddev 8.168
progress: 260.0 s, 694.7 tps, lat 11.515 ms stddev 8.577
progress: 270.0 s, 695.7 tps, lat 11.495 ms stddev 8.631
progress: 280.0 s, 705.9 tps, lat 11.334 ms stddev 8.381
progress: 290.0 s, 698.1 tps, lat 11.457 ms stddev 10.190
progress: 300.0 s, 739.5 tps, lat 10.820 ms stddev 8.017
progress: 310.0 s, 708.3 tps, lat 11.287 ms stddev 8.143
progress: 320.0 s, 732.0 tps, lat 10.932 ms stddev 7.958
progress: 330.0 s, 740.9 tps, lat 10.796 ms stddev 7.949
progress: 340.0 s, 726.8 tps, lat 11.007 ms stddev 7.917
progress: 350.0 s, 711.5 tps, lat 11.234 ms stddev 8.171
progress: 360.0 s, 711.0 tps, lat 11.256 ms stddev 8.218
progress: 370.0 s, 646.5 tps, lat 12.379 ms stddev 8.851
progress: 380.0 s, 677.4 tps, lat 11.801 ms stddev 8.435
progress: 390.0 s, 724.3 tps, lat 11.050 ms stddev 7.890
progress: 400.0 s, 727.8 tps, lat 10.991 ms stddev 7.666
progress: 410.0 s, 725.4 tps, lat 11.019 ms stddev 8.179
progress: 420.0 s, 724.0 tps, lat 11.057 ms stddev 7.727
progress: 430.0 s, 676.0 tps, lat 11.826 ms stddev 8.040
progress: 440.0 s, 717.4 tps, lat 11.159 ms stddev 8.103
progress: 450.0 s, 722.3 tps, lat 11.072 ms stddev 7.655
progress: 460.0 s, 687.1 tps, lat 11.639 ms stddev 8.245
progress: 470.0 s, 713.4 tps, lat 11.215 ms stddev 8.106
progress: 480.0 s, 726.5 tps, lat 11.012 ms stddev 7.938
progress: 490.0 s, 704.6 tps, lat 11.352 ms stddev 7.529
progress: 500.0 s, 728.8 tps, lat 10.978 ms stddev 7.483
progress: 510.0 s, 704.8 tps, lat 11.350 ms stddev 8.145
progress: 520.0 s, 719.2 tps, lat 11.120 ms stddev 8.323
progress: 530.0 s, 556.4 tps, lat 14.379 ms stddev 19.135
progress: 540.0 s, 486.7 tps, lat 16.438 ms stddev 23.191
progress: 550.0 s, 481.9 tps, lat 16.580 ms stddev 23.522
progress: 560.0 s, 480.3 tps, lat 16.669 ms stddev 24.334
progress: 570.0 s, 488.1 tps, lat 16.370 ms stddev 23.402
progress: 580.0 s, 477.3 tps, lat 16.777 ms stddev 23.047
progress: 590.0 s, 461.1 tps, lat 17.352 ms stddev 24.522
progress: 600.0 s, 483.7 tps, lat 16.516 ms stddev 23.148
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 411962
latency average = 11.650 ms
latency stddev = 10.462 ms
tps = 686.573599 (including connections establishing)
tps = 686.577803 (excluding connections establishing)
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 19:11:08 UTC; 2s ago
    Process: 7538 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 7538 (code=exited, status=0/SUCCESS)

Oct 23 19:11:08 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 19:11:08 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 658.5 tps, lat 12.105 ms stddev 9.019
progress: 20.0 s, 622.0 tps, lat 12.862 ms stddev 9.034
progress: 30.0 s, 645.1 tps, lat 12.403 ms stddev 8.757
progress: 40.0 s, 663.9 tps, lat 12.045 ms stddev 8.352
progress: 50.0 s, 619.1 tps, lat 12.921 ms stddev 8.916
progress: 60.0 s, 657.7 tps, lat 12.166 ms stddev 8.062
progress: 70.0 s, 694.9 tps, lat 11.511 ms stddev 7.784
progress: 80.0 s, 683.6 tps, lat 11.699 ms stddev 7.708
progress: 90.0 s, 687.3 tps, lat 11.641 ms stddev 7.733
progress: 100.0 s, 693.3 tps, lat 11.529 ms stddev 7.808
progress: 110.0 s, 692.5 tps, lat 11.555 ms stddev 7.452
progress: 120.0 s, 684.5 tps, lat 11.690 ms stddev 7.462
progress: 130.0 s, 714.2 tps, lat 11.202 ms stddev 7.094
progress: 140.0 s, 677.5 tps, lat 11.807 ms stddev 7.832
progress: 150.0 s, 647.7 tps, lat 12.353 ms stddev 8.347
progress: 160.0 s, 672.8 tps, lat 11.888 ms stddev 7.685
progress: 170.0 s, 664.8 tps, lat 12.034 ms stddev 8.560
progress: 180.0 s, 644.7 tps, lat 12.403 ms stddev 9.000
progress: 190.0 s, 663.2 tps, lat 12.063 ms stddev 9.112
progress: 200.0 s, 666.8 tps, lat 11.994 ms stddev 8.762
progress: 210.0 s, 634.0 tps, lat 12.617 ms stddev 9.094
progress: 220.0 s, 652.7 tps, lat 12.258 ms stddev 9.163
progress: 230.0 s, 643.0 tps, lat 12.435 ms stddev 8.964
progress: 240.0 s, 593.5 tps, lat 13.479 ms stddev 9.482
progress: 250.0 s, 656.2 tps, lat 12.193 ms stddev 8.940
progress: 260.0 s, 649.7 tps, lat 12.306 ms stddev 8.834
progress: 270.0 s, 623.1 tps, lat 12.845 ms stddev 9.071
progress: 280.0 s, 648.4 tps, lat 12.328 ms stddev 8.835
progress: 290.0 s, 616.5 tps, lat 12.972 ms stddev 10.943
progress: 300.0 s, 639.8 tps, lat 12.516 ms stddev 8.768
progress: 310.0 s, 659.6 tps, lat 12.126 ms stddev 8.467
progress: 320.0 s, 634.7 tps, lat 12.599 ms stddev 8.587
progress: 330.0 s, 619.7 tps, lat 12.907 ms stddev 8.649
progress: 340.0 s, 648.0 tps, lat 12.344 ms stddev 8.470
progress: 350.0 s, 647.1 tps, lat 12.365 ms stddev 8.744
progress: 360.0 s, 652.5 tps, lat 12.260 ms stddev 8.666
progress: 370.0 s, 660.1 tps, lat 12.113 ms stddev 8.403
progress: 380.0 s, 626.8 tps, lat 12.768 ms stddev 8.197
progress: 390.0 s, 655.1 tps, lat 12.208 ms stddev 8.338
progress: 400.0 s, 643.7 tps, lat 12.429 ms stddev 8.476
progress: 410.0 s, 638.3 tps, lat 12.529 ms stddev 8.322
progress: 420.0 s, 624.7 tps, lat 12.806 ms stddev 8.262
progress: 430.0 s, 648.7 tps, lat 12.330 ms stddev 8.767
progress: 440.0 s, 666.4 tps, lat 12.003 ms stddev 8.852
progress: 450.0 s, 677.5 tps, lat 11.808 ms stddev 8.746
progress: 460.0 s, 660.7 tps, lat 12.107 ms stddev 8.530
progress: 470.0 s, 656.0 tps, lat 12.192 ms stddev 9.121
progress: 480.0 s, 646.7 tps, lat 12.369 ms stddev 8.699
progress: 490.0 s, 667.0 tps, lat 11.994 ms stddev 8.721
progress: 500.0 s, 640.9 tps, lat 12.482 ms stddev 8.895
progress: 510.0 s, 644.5 tps, lat 12.409 ms stddev 9.169
progress: 520.0 s, 653.4 tps, lat 12.243 ms stddev 8.780
progress: 530.0 s, 648.1 tps, lat 12.346 ms stddev 9.174
progress: 540.0 s, 603.6 tps, lat 13.249 ms stddev 8.995
progress: 550.0 s, 571.7 tps, lat 13.991 ms stddev 9.976
progress: 560.0 s, 656.6 tps, lat 12.176 ms stddev 8.440
progress: 570.0 s, 651.5 tps, lat 12.287 ms stddev 8.771
progress: 580.0 s, 614.6 tps, lat 13.010 ms stddev 9.241
progress: 590.0 s, 632.2 tps, lat 12.658 ms stddev 9.613
progress: 600.0 s, 643.2 tps, lat 12.427 ms stddev 8.799
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 390054
latency average = 12.304 ms
latency stddev = 8.676 ms
tps = 650.063150 (including connections establishing)
tps = 650.066474 (excluding connections establishing)
postgres@ub20-postgres-6:~$ cp /etc/postgresql/13/main/postgresql.conf ./0130snm.conf
postgres@ub20-postgres-6:~$ ls -la
total 156
drwxr-xr-x  4 postgres postgres  4096 Oct 23 19:30 .
drwxr-xr-x 38 root     root      4096 Oct 23 14:00 ..
-rw-------  1 postgres postgres  2049 Oct 23 19:11 .bash_history
drwx------  3 postgres postgres  4096 Oct 23 14:30 .config
-rw-------  1 postgres postgres    67 Oct 23 18:55 .lesshst
-rw-------  1 postgres postgres  5708 Oct 23 17:22 .psql_history
-rw-------  1 postgres postgres 17381 Oct 23 19:11 .viminfo
-rw-r--r--  1 postgres postgres 28338 Oct 23 19:30 0130snm.conf
-rw-------  1 postgres postgres 77454 Oct 23 18:09 1.log
drwxr-xr-x  3 postgres postgres  4096 Oct 23 14:00 13
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 19:31:06 UTC; 3s ago
    Process: 7848 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 7848 (code=exited, status=0/SUCCESS)

Oct 23 19:31:06 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 19:31:06 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 670.3 tps, lat 11.894 ms stddev 8.798
progress: 20.0 s, 657.9 tps, lat 12.159 ms stddev 8.853
progress: 30.0 s, 679.4 tps, lat 11.776 ms stddev 8.076
progress: 40.0 s, 656.0 tps, lat 12.192 ms stddev 8.355
progress: 50.0 s, 647.5 tps, lat 12.354 ms stddev 8.593
progress: 60.0 s, 662.6 tps, lat 12.071 ms stddev 8.016
progress: 70.0 s, 694.7 tps, lat 11.511 ms stddev 8.204
progress: 80.0 s, 674.4 tps, lat 11.868 ms stddev 8.082
progress: 90.0 s, 721.3 tps, lat 11.087 ms stddev 7.470
progress: 100.0 s, 714.7 tps, lat 11.194 ms stddev 7.500
progress: 110.0 s, 703.0 tps, lat 11.379 ms stddev 7.707
progress: 120.0 s, 719.5 tps, lat 11.118 ms stddev 7.142
progress: 130.0 s, 723.3 tps, lat 11.054 ms stddev 7.137
progress: 140.0 s, 682.3 tps, lat 11.728 ms stddev 7.506
progress: 150.0 s, 703.5 tps, lat 11.368 ms stddev 7.053
progress: 160.0 s, 705.8 tps, lat 11.337 ms stddev 7.271
progress: 170.0 s, 691.8 tps, lat 11.563 ms stddev 7.805
progress: 180.0 s, 672.2 tps, lat 11.898 ms stddev 8.873
progress: 190.0 s, 708.1 tps, lat 11.294 ms stddev 8.119
progress: 200.0 s, 693.2 tps, lat 11.538 ms stddev 8.980
progress: 210.0 s, 675.5 tps, lat 11.845 ms stddev 8.533
progress: 220.0 s, 706.8 tps, lat 11.316 ms stddev 8.169
progress: 230.0 s, 690.4 tps, lat 11.589 ms stddev 8.435
progress: 240.0 s, 742.5 tps, lat 10.771 ms stddev 7.976
progress: 250.0 s, 817.3 tps, lat 9.790 ms stddev 7.536
progress: 260.0 s, 822.4 tps, lat 9.726 ms stddev 7.532
progress: 270.0 s, 820.9 tps, lat 9.747 ms stddev 7.485
progress: 280.0 s, 815.5 tps, lat 9.807 ms stddev 7.620
progress: 290.0 s, 784.8 tps, lat 10.190 ms stddev 9.639
progress: 300.0 s, 743.7 tps, lat 10.760 ms stddev 8.000
progress: 310.0 s, 793.8 tps, lat 10.063 ms stddev 7.518
progress: 320.0 s, 801.3 tps, lat 9.997 ms stddev 7.562
progress: 330.0 s, 810.2 tps, lat 9.872 ms stddev 7.408
progress: 340.0 s, 801.0 tps, lat 9.986 ms stddev 7.519
progress: 350.0 s, 808.9 tps, lat 9.882 ms stddev 7.730
progress: 360.0 s, 813.0 tps, lat 9.842 ms stddev 7.443
progress: 370.0 s, 823.1 tps, lat 9.719 ms stddev 7.259
progress: 380.0 s, 805.6 tps, lat 9.934 ms stddev 7.394
progress: 390.0 s, 817.2 tps, lat 9.787 ms stddev 7.465
progress: 400.0 s, 817.8 tps, lat 9.781 ms stddev 7.337
progress: 410.0 s, 808.9 tps, lat 9.880 ms stddev 7.402
progress: 420.0 s, 821.6 tps, lat 9.742 ms stddev 7.146
progress: 430.0 s, 832.4 tps, lat 9.612 ms stddev 6.861
progress: 440.0 s, 600.8 tps, lat 13.317 ms stddev 19.829
progress: 450.0 s, 527.2 tps, lat 15.174 ms stddev 23.041
progress: 460.0 s, 550.9 tps, lat 14.521 ms stddev 22.250
progress: 470.0 s, 538.1 tps, lat 14.867 ms stddev 22.505
progress: 480.0 s, 541.6 tps, lat 14.768 ms stddev 22.574
progress: 490.0 s, 541.0 tps, lat 14.778 ms stddev 22.525
progress: 500.0 s, 534.3 tps, lat 14.981 ms stddev 23.667
progress: 510.0 s, 542.0 tps, lat 14.755 ms stddev 22.756
progress: 520.0 s, 542.0 tps, lat 14.758 ms stddev 22.259
progress: 530.0 s, 530.3 tps, lat 15.088 ms stddev 23.468
progress: 540.0 s, 555.1 tps, lat 14.411 ms stddev 21.316
progress: 550.0 s, 542.3 tps, lat 14.749 ms stddev 22.699
progress: 560.0 s, 525.5 tps, lat 15.222 ms stddev 23.531
progress: 570.0 s, 519.3 tps, lat 15.405 ms stddev 24.308
progress: 580.0 s, 504.1 tps, lat 15.834 ms stddev 25.284
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 19:43:00 UTC; 2s ago
    Process: 7975 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 7975 (code=exited, status=0/SUCCESS)

Oct 23 19:43:00 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 19:43:00 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 803.6 tps, lat 9.927 ms stddev 7.637
progress: 20.0 s, 804.8 tps, lat 9.937 ms stddev 7.627
progress: 30.0 s, 817.5 tps, lat 9.786 ms stddev 7.221
progress: 40.0 s, 810.3 tps, lat 9.871 ms stddev 7.042
progress: 50.0 s, 804.8 tps, lat 9.940 ms stddev 7.402
progress: 60.0 s, 809.6 tps, lat 9.882 ms stddev 7.291
progress: 70.0 s, 805.4 tps, lat 9.931 ms stddev 7.385
progress: 80.0 s, 816.1 tps, lat 9.800 ms stddev 6.967
progress: 90.0 s, 808.6 tps, lat 9.889 ms stddev 7.224
progress: 100.0 s, 814.6 tps, lat 9.826 ms stddev 6.955
progress: 110.0 s, 817.4 tps, lat 9.778 ms stddev 7.023
progress: 120.0 s, 823.3 tps, lat 9.721 ms stddev 6.918
progress: 130.0 s, 813.4 tps, lat 9.833 ms stddev 6.877
progress: 140.0 s, 813.7 tps, lat 9.829 ms stddev 6.715
progress: 150.0 s, 824.1 tps, lat 9.709 ms stddev 6.518
progress: 160.0 s, 817.5 tps, lat 9.787 ms stddev 6.704
progress: 170.0 s, 826.8 tps, lat 9.675 ms stddev 6.320
progress: 180.0 s, 817.2 tps, lat 9.787 ms stddev 6.174
progress: 190.0 s, 802.6 tps, lat 9.967 ms stddev 6.229
progress: 200.0 s, 813.0 tps, lat 9.839 ms stddev 6.538
progress: 210.0 s, 762.8 tps, lat 10.484 ms stddev 7.715
progress: 220.0 s, 752.7 tps, lat 10.629 ms stddev 8.215
progress: 230.0 s, 822.0 tps, lat 9.731 ms stddev 7.473
progress: 240.0 s, 820.9 tps, lat 9.742 ms stddev 7.254
progress: 250.0 s, 817.0 tps, lat 9.794 ms stddev 7.387
progress: 260.0 s, 815.0 tps, lat 9.815 ms stddev 7.736
progress: 270.0 s, 821.8 tps, lat 9.733 ms stddev 7.694
progress: 280.0 s, 825.1 tps, lat 9.696 ms stddev 7.615
progress: 290.0 s, 813.9 tps, lat 9.820 ms stddev 8.365
progress: 300.0 s, 817.3 tps, lat 9.790 ms stddev 7.648
progress: 310.0 s, 821.0 tps, lat 9.747 ms stddev 7.567
progress: 320.0 s, 809.9 tps, lat 9.874 ms stddev 7.435
progress: 330.0 s, 823.1 tps, lat 9.720 ms stddev 7.599
progress: 340.0 s, 824.7 tps, lat 9.699 ms stddev 7.461
progress: 350.0 s, 811.7 tps, lat 9.856 ms stddev 7.648
progress: 360.0 s, 614.0 tps, lat 13.026 ms stddev 19.221
progress: 370.0 s, 522.7 tps, lat 15.306 ms stddev 24.071
progress: 380.0 s, 518.5 tps, lat 15.429 ms stddev 24.293
progress: 390.0 s, 518.0 tps, lat 15.439 ms stddev 25.297
progress: 400.0 s, 535.1 tps, lat 14.954 ms stddev 23.160
progress: 410.0 s, 534.4 tps, lat 14.969 ms stddev 23.608
progress: 420.0 s, 506.8 tps, lat 15.788 ms stddev 25.253
progress: 430.0 s, 530.1 tps, lat 15.079 ms stddev 23.548
progress: 440.0 s, 532.4 tps, lat 15.028 ms stddev 23.716
progress: 450.0 s, 519.9 tps, lat 15.390 ms stddev 25.020
progress: 460.0 s, 530.7 tps, lat 15.060 ms stddev 24.117
progress: 470.0 s, 534.6 tps, lat 14.963 ms stddev 23.038
progress: 480.0 s, 520.3 tps, lat 15.377 ms stddev 24.467
progress: 490.0 s, 524.9 tps, lat 15.235 ms stddev 23.758
progress: 500.0 s, 508.3 tps, lat 15.742 ms stddev 24.098
progress: 510.0 s, 504.5 tps, lat 15.836 ms stddev 26.052
progress: 520.0 s, 515.0 tps, lat 15.514 ms stddev 24.477
progress: 530.0 s, 489.2 tps, lat 16.350 ms stddev 24.466
progress: 540.0 s, 483.5 tps, lat 16.546 ms stddev 25.971
progress: 550.0 s, 526.0 tps, lat 15.211 ms stddev 23.596
progress: 560.0 s, 506.8 tps, lat 15.785 ms stddev 25.272
progress: 570.0 s, 520.8 tps, lat 15.358 ms stddev 24.194
progress: 580.0 s, 518.9 tps, lat 15.416 ms stddev 24.309
progress: 590.0 s, 520.7 tps, lat 15.361 ms stddev 24.568
progress: 600.1 s, 507.6 tps, lat 15.566 ms stddev 25.019
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 414726
latency average = 11.574 ms
latency stddev = 15.062 ms
tps = 691.063458 (including connections establishing)
tps = 691.066725 (excluding connections establishing)
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ ls -la 13/main/log/post
ls: cannot access '13/main/log/post': No such file or directory
postgres@ub20-postgres-6:~$ ls -la 13/main/log/post*
-rw------- 1 postgres postgres      0 Oct 23 18:10 13/main/log/postgresql-2021-10-23_172029.log
-rw------- 1 postgres postgres 106036 Oct 23 18:58 13/main/log/postgresql-2021-10-23_181114.log
-rw------- 1 postgres postgres 112520 Oct 23 19:11 13/main/log/postgresql-2021-10-23_185806.log
-rw------- 1 postgres postgres 149276 Oct 23 19:31 13/main/log/postgresql-2021-10-23_191106.log
-rw------- 1 postgres postgres  88772 Oct 23 19:42 13/main/log/postgresql-2021-10-23_193104.log
-rw------- 1 postgres postgres 172466 Oct 23 19:53 13/main/log/postgresql-2021-10-23_194258.log
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_194258.log |grep -c 'autova'
0
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_194258.log |less
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_194258.log |grep -c 'automatic vacuum'
273
postgres@ub20-postgres-6:~$ cat 13/main/log/13/main/log/postgresql-2021-10-23_193104.log |grep -c 'automatic vacuum'
cat: 13/main/log/13/main/log/postgresql-2021-10-23_193104.log: No such file or directory
0
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_193104.log |grep -c 'automatic vacuum'
137
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_191106.log |grep -c 'automatic vacuum'
240
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_185806.log |grep -c 'automatic vacuum'
178
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_181114.log |grep -c 'automatic vacuum'
166
postgres@ub20-postgres-6:~$ ls -la
total 156
drwxr-xr-x  4 postgres postgres  4096 Oct 23 19:58 .
drwxr-xr-x 38 root     root      4096 Oct 23 14:00 ..
-rw-------  1 postgres postgres  2294 Oct 23 19:42 .bash_history
drwx------  3 postgres postgres  4096 Oct 23 14:30 .config
-rw-------  1 postgres postgres    67 Oct 23 18:55 .lesshst
-rw-------  1 postgres postgres  5708 Oct 23 17:22 .psql_history
-rw-------  1 postgres postgres 17923 Oct 23 19:58 .viminfo
-rw-r--r--  1 postgres postgres 28338 Oct 23 19:30 0130snm.conf
-rw-------  1 postgres postgres 77454 Oct 23 18:09 1.log
drwxr-xr-x  3 postgres postgres  4096 Oct 23 14:00 13
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_194258.log |less
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_181114.log |grep -c 'index scans: 1'
98
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_185806.log |grep -c 'index scans: 1'
106
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_191106.log |grep -c 'index scans: 1'
139
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_193104.log |grep -c 'index scans: 1'
76
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ vim 0130snm.conf
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 20:12:02 UTC; 4s ago
    Process: 8434 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 8434 (code=exited, status=0/SUCCESS)

Oct 23 20:12:02 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 20:12:02 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 792.1 tps, lat 10.063 ms stddev 7.643
progress: 20.0 s, 805.2 tps, lat 9.935 ms stddev 7.378
progress: 30.0 s, 797.6 tps, lat 10.026 ms stddev 7.456
progress: 40.0 s, 751.4 tps, lat 10.633 ms stddev 7.911
progress: 50.0 s, 739.5 tps, lat 10.831 ms stddev 7.698
progress: 60.0 s, 794.9 tps, lat 10.064 ms stddev 7.539
progress: 70.0 s, 802.8 tps, lat 9.962 ms stddev 6.784
progress: 80.0 s, 805.1 tps, lat 9.939 ms stddev 7.002
progress: 90.0 s, 801.3 tps, lat 9.980 ms stddev 6.680
progress: 100.0 s, 808.1 tps, lat 9.900 ms stddev 6.894
progress: 110.0 s, 811.5 tps, lat 9.857 ms stddev 6.593
progress: 120.0 s, 807.0 tps, lat 9.911 ms stddev 6.953
progress: 130.0 s, 810.0 tps, lat 9.879 ms stddev 6.256
progress: 140.0 s, 816.5 tps, lat 9.796 ms stddev 6.398
progress: 150.0 s, 817.9 tps, lat 9.779 ms stddev 6.522
progress: 160.0 s, 812.7 tps, lat 9.846 ms stddev 6.388
progress: 170.0 s, 814.9 tps, lat 9.813 ms stddev 6.194
progress: 180.0 s, 826.0 tps, lat 9.687 ms stddev 6.148
progress: 190.0 s, 805.2 tps, lat 9.934 ms stddev 6.259
progress: 200.0 s, 829.4 tps, lat 9.645 ms stddev 6.222
progress: 210.0 s, 823.7 tps, lat 9.712 ms stddev 6.163
progress: 220.0 s, 804.9 tps, lat 9.938 ms stddev 6.444
progress: 230.0 s, 818.6 tps, lat 9.772 ms stddev 6.084
progress: 240.0 s, 823.0 tps, lat 9.718 ms stddev 6.859
progress: 250.0 s, 803.6 tps, lat 9.957 ms stddev 7.493
progress: 260.0 s, 806.9 tps, lat 9.908 ms stddev 7.679
progress: 270.0 s, 790.4 tps, lat 10.125 ms stddev 7.555
progress: 280.0 s, 787.0 tps, lat 10.164 ms stddev 7.749
progress: 290.0 s, 798.5 tps, lat 10.017 ms stddev 7.577
progress: 300.0 s, 814.3 tps, lat 9.822 ms stddev 7.370
progress: 310.0 s, 799.7 tps, lat 10.002 ms stddev 7.501
progress: 320.0 s, 797.3 tps, lat 10.033 ms stddev 7.740
progress: 330.0 s, 813.5 tps, lat 9.835 ms stddev 7.575
progress: 340.0 s, 812.9 tps, lat 9.840 ms stddev 7.552
progress: 350.0 s, 803.1 tps, lat 9.959 ms stddev 7.719
progress: 360.0 s, 768.3 tps, lat 10.413 ms stddev 7.761
progress: 370.0 s, 798.5 tps, lat 10.018 ms stddev 7.560
progress: 380.0 s, 812.9 tps, lat 9.840 ms stddev 7.365
progress: 390.0 s, 800.8 tps, lat 9.986 ms stddev 7.563
progress: 400.0 s, 744.4 tps, lat 10.748 ms stddev 11.387
progress: 410.0 s, 513.3 tps, lat 15.588 ms stddev 23.735
progress: 420.0 s, 520.6 tps, lat 15.357 ms stddev 23.137
progress: 430.0 s, 527.1 tps, lat 15.181 ms stddev 22.809
progress: 440.0 s, 503.7 tps, lat 15.881 ms stddev 25.163
progress: 450.0 s, 512.6 tps, lat 15.605 ms stddev 23.860
progress: 460.0 s, 519.8 tps, lat 15.393 ms stddev 23.570
progress: 470.0 s, 526.9 tps, lat 15.180 ms stddev 22.346
progress: 480.0 s, 516.4 tps, lat 15.492 ms stddev 23.740
progress: 490.0 s, 517.2 tps, lat 15.468 ms stddev 22.999
progress: 500.0 s, 501.5 tps, lat 15.948 ms stddev 25.574
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_201200.log |grep -c 'index scans: 1'
64
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_201200.log |grep -c 'automatic vacuum'
144
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_201200.log |less
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 20:27:37 UTC; 2s ago
    Process: 8699 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 8699 (code=exited, status=0/SUCCESS)

Oct 23 20:27:37 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 20:27:37 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 839.7 tps, lat 9.498 ms stddev 7.397
progress: 20.0 s, 831.5 tps, lat 9.619 ms stddev 8.941
progress: 30.0 s, 841.9 tps, lat 9.504 ms stddev 7.100
progress: 40.0 s, 851.7 tps, lat 9.391 ms stddev 7.090
progress: 50.0 s, 842.4 tps, lat 9.493 ms stddev 7.431
progress: 60.0 s, 847.1 tps, lat 9.446 ms stddev 7.040
progress: 70.0 s, 790.0 tps, lat 10.126 ms stddev 7.515
progress: 80.0 s, 803.3 tps, lat 9.957 ms stddev 7.397
progress: 90.0 s, 839.0 tps, lat 9.532 ms stddev 7.155
progress: 100.0 s, 852.2 tps, lat 9.389 ms stddev 7.183
progress: 110.0 s, 854.7 tps, lat 9.356 ms stddev 6.890
progress: 120.0 s, 848.9 tps, lat 9.427 ms stddev 6.839
progress: 130.0 s, 863.1 tps, lat 9.268 ms stddev 6.471
progress: 140.0 s, 852.7 tps, lat 9.380 ms stddev 7.058
progress: 150.0 s, 850.4 tps, lat 9.408 ms stddev 6.627
progress: 160.0 s, 854.6 tps, lat 9.359 ms stddev 6.666
progress: 170.0 s, 850.0 tps, lat 9.410 ms stddev 6.840
progress: 180.0 s, 857.2 tps, lat 9.331 ms stddev 7.454
progress: 190.0 s, 837.7 tps, lat 9.550 ms stddev 7.410
progress: 200.0 s, 840.1 tps, lat 9.523 ms stddev 7.356
progress: 210.0 s, 831.8 tps, lat 9.612 ms stddev 7.527
progress: 220.0 s, 826.0 tps, lat 9.688 ms stddev 7.286
progress: 230.0 s, 833.3 tps, lat 9.598 ms stddev 7.341
progress: 240.0 s, 837.1 tps, lat 9.557 ms stddev 7.404
progress: 250.0 s, 828.8 tps, lat 9.652 ms stddev 7.539
progress: 260.0 s, 821.7 tps, lat 9.728 ms stddev 7.338
progress: 270.0 s, 820.4 tps, lat 9.756 ms stddev 7.359
progress: 280.0 s, 844.0 tps, lat 9.472 ms stddev 7.245
progress: 290.0 s, 798.8 tps, lat 10.018 ms stddev 8.292
progress: 300.0 s, 820.7 tps, lat 9.750 ms stddev 7.667
progress: 310.0 s, 818.3 tps, lat 9.775 ms stddev 7.521
progress: 320.0 s, 807.2 tps, lat 9.906 ms stddev 7.893
progress: 330.0 s, 808.6 tps, lat 9.895 ms stddev 7.690
progress: 340.0 s, 809.7 tps, lat 9.872 ms stddev 7.673
progress: 350.0 s, 813.4 tps, lat 9.840 ms stddev 7.570
progress: 360.0 s, 820.3 tps, lat 9.751 ms stddev 7.501
progress: 370.0 s, 819.2 tps, lat 9.768 ms stddev 7.557
progress: 380.0 s, 822.9 tps, lat 9.720 ms stddev 7.263
progress: 390.0 s, 743.2 tps, lat 10.762 ms stddev 7.742
progress: 400.0 s, 780.7 tps, lat 10.248 ms stddev 7.701
progress: 410.0 s, 803.6 tps, lat 9.953 ms stddev 7.293
progress: 420.0 s, 803.8 tps, lat 9.950 ms stddev 7.384
progress: 430.0 s, 806.0 tps, lat 9.917 ms stddev 7.381
progress: 440.0 s, 796.0 tps, lat 10.058 ms stddev 7.185
progress: 450.0 s, 796.0 tps, lat 10.050 ms stddev 7.366
progress: 460.0 s, 801.0 tps, lat 9.986 ms stddev 7.485
progress: 470.0 s, 627.5 tps, lat 12.748 ms stddev 17.254
progress: 480.0 s, 513.1 tps, lat 15.590 ms stddev 25.125
progress: 490.0 s, 533.5 tps, lat 14.995 ms stddev 22.490
progress: 500.0 s, 524.1 tps, lat 15.264 ms stddev 23.319
progress: 510.0 s, 540.3 tps, lat 14.802 ms stddev 21.732
progress: 520.0 s, 527.2 tps, lat 15.172 ms stddev 23.652
^C
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_20 |less
postgresql-2021-10-23_201200.log  postgresql-2021-10-23_202735.log
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_20 |less
postgresql-2021-10-23_201200.log  postgresql-2021-10-23_202735.log
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_202735.log |less
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_202735.log |grep -c 'automatic vacuum'
243
postgres@ub20-postgres-6:~$ cat 13/main/log/postgresql-2021-10-23_202735.log |grep -c 'index scans: 1'
123
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 20:40:50 UTC; 3s ago
    Process: 8938 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 8938 (code=exited, status=0/SUCCESS)

Oct 23 20:40:50 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 20:40:50 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 770.1 tps, lat 10.352 ms stddev 7.933
progress: 20.0 s, 778.6 tps, lat 10.277 ms stddev 7.684
progress: 30.0 s, 790.6 tps, lat 10.117 ms stddev 7.234
progress: 40.0 s, 800.4 tps, lat 9.997 ms stddev 7.325
progress: 50.0 s, 794.6 tps, lat 10.066 ms stddev 7.074
progress: 60.0 s, 799.0 tps, lat 10.011 ms stddev 7.149
progress: 70.0 s, 800.5 tps, lat 9.992 ms stddev 7.156
progress: 80.0 s, 776.5 tps, lat 10.300 ms stddev 7.164
progress: 90.0 s, 766.0 tps, lat 10.446 ms stddev 7.212
progress: 100.0 s, 794.7 tps, lat 10.062 ms stddev 6.961
progress: 110.0 s, 788.6 tps, lat 10.149 ms stddev 6.779
progress: 120.0 s, 783.9 tps, lat 10.201 ms stddev 6.892
progress: 130.0 s, 786.3 tps, lat 10.173 ms stddev 6.911
progress: 140.0 s, 786.8 tps, lat 10.169 ms stddev 6.786
progress: 150.0 s, 786.0 tps, lat 10.176 ms stddev 6.815
progress: 160.0 s, 788.0 tps, lat 10.153 ms stddev 6.454
progress: 170.0 s, 791.4 tps, lat 10.108 ms stddev 6.269
progress: 180.0 s, 774.8 tps, lat 10.324 ms stddev 6.570
progress: 190.0 s, 770.4 tps, lat 10.383 ms stddev 6.575
progress: 200.0 s, 774.2 tps, lat 10.333 ms stddev 6.365
progress: 210.0 s, 788.0 tps, lat 10.151 ms stddev 6.304
progress: 220.0 s, 703.6 tps, lat 11.364 ms stddev 8.139
progress: 230.0 s, 720.1 tps, lat 11.106 ms stddev 8.134
progress: 240.0 s, 775.5 tps, lat 10.319 ms stddev 7.764
progress: 250.0 s, 791.6 tps, lat 10.105 ms stddev 7.623
progress: 260.0 s, 782.6 tps, lat 10.222 ms stddev 7.830
progress: 270.0 s, 785.5 tps, lat 10.182 ms stddev 7.860
progress: 280.0 s, 784.2 tps, lat 10.203 ms stddev 7.977
progress: 290.0 s, 777.3 tps, lat 10.290 ms stddev 8.737
progress: 300.0 s, 784.9 tps, lat 10.192 ms stddev 7.653
progress: 310.0 s, 782.6 tps, lat 10.220 ms stddev 7.572
progress: 320.0 s, 777.9 tps, lat 10.285 ms stddev 7.925
progress: 330.0 s, 794.1 tps, lat 10.073 ms stddev 7.320
progress: 340.0 s, 796.9 tps, lat 10.038 ms stddev 7.411
progress: 350.0 s, 773.1 tps, lat 10.345 ms stddev 7.717
progress: 360.0 s, 751.9 tps, lat 10.639 ms stddev 7.792
progress: 370.0 s, 785.9 tps, lat 10.179 ms stddev 7.597
progress: 380.0 s, 772.8 tps, lat 10.351 ms stddev 7.782
progress: 390.0 s, 757.3 tps, lat 10.562 ms stddev 7.828
progress: 400.0 s, 796.1 tps, lat 10.050 ms stddev 7.381
progress: 410.0 s, 796.2 tps, lat 10.041 ms stddev 6.908
progress: 420.0 s, 795.9 tps, lat 10.056 ms stddev 7.159
progress: 430.0 s, 637.5 tps, lat 12.548 ms stddev 17.009
progress: 440.0 s, 521.1 tps, lat 15.351 ms stddev 22.979
progress: 450.0 s, 525.1 tps, lat 15.229 ms stddev 22.421
progress: 460.0 s, 515.8 tps, lat 15.511 ms stddev 23.497
progress: 470.0 s, 521.4 tps, lat 15.342 ms stddev 22.878
progress: 480.0 s, 524.5 tps, lat 15.252 ms stddev 22.588
progress: 490.0 s, 517.6 tps, lat 15.450 ms stddev 23.827
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 20:50:42 UTC; 4s ago
    Process: 9113 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9113 (code=exited, status=0/SUCCESS)

Oct 23 20:50:42 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 20:50:42 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 791.4 tps, lat 10.079 ms stddev 7.439
progress: 20.0 s, 792.4 tps, lat 10.096 ms stddev 7.615
progress: 30.0 s, 789.6 tps, lat 10.130 ms stddev 7.376
progress: 40.0 s, 790.4 tps, lat 10.115 ms stddev 7.213
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 20:52:14 UTC; 2s ago
    Process: 9183 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9183 (code=exited, status=0/SUCCESS)

Oct 23 20:52:14 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 20:52:14 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 778.0 tps, lat 10.252 ms stddev 7.941
progress: 20.0 s, 792.8 tps, lat 10.090 ms stddev 7.561
progress: 30.0 s, 804.6 tps, lat 9.942 ms stddev 7.315
progress: 40.0 s, 809.7 tps, lat 9.879 ms stddev 7.279
progress: 50.0 s, 811.1 tps, lat 9.863 ms stddev 7.361
progress: 60.0 s, 819.2 tps, lat 9.760 ms stddev 6.900
progress: 70.0 s, 811.6 tps, lat 9.858 ms stddev 6.878
progress: 80.0 s, 817.1 tps, lat 9.792 ms stddev 7.244
progress: 90.0 s, 802.0 tps, lat 9.973 ms stddev 7.005
progress: 100.0 s, 801.2 tps, lat 9.980 ms stddev 6.868
progress: 110.0 s, 809.8 tps, lat 9.883 ms stddev 6.588
progress: 120.0 s, 807.3 tps, lat 9.908 ms stddev 6.926
progress: 130.0 s, 810.3 tps, lat 9.870 ms stddev 6.644
progress: 140.0 s, 804.0 tps, lat 9.948 ms stddev 6.869
progress: 150.0 s, 811.2 tps, lat 9.861 ms stddev 6.613
progress: 160.0 s, 800.7 tps, lat 9.994 ms stddev 6.836
progress: 170.0 s, 804.0 tps, lat 9.949 ms stddev 6.593
progress: 180.0 s, 756.6 tps, lat 10.566 ms stddev 7.717
progress: 190.0 s, 754.7 tps, lat 10.603 ms stddev 7.961
progress: 200.0 s, 792.1 tps, lat 10.099 ms stddev 7.496
progress: 210.0 s, 810.1 tps, lat 9.875 ms stddev 6.464
progress: 220.0 s, 811.2 tps, lat 9.862 ms stddev 6.189
progress: 230.0 s, 804.1 tps, lat 9.947 ms stddev 7.188
progress: 240.0 s, 801.7 tps, lat 9.979 ms stddev 7.717
progress: 250.0 s, 813.0 tps, lat 9.839 ms stddev 7.368
progress: 260.0 s, 807.6 tps, lat 9.903 ms stddev 7.670
progress: 270.0 s, 805.7 tps, lat 9.928 ms stddev 7.659
progress: 280.0 s, 810.6 tps, lat 9.867 ms stddev 7.764
progress: 290.0 s, 822.0 tps, lat 9.732 ms stddev 7.283
progress: 300.0 s, 793.7 tps, lat 10.080 ms stddev 8.574
progress: 310.0 s, 816.7 tps, lat 9.794 ms stddev 7.500
progress: 320.0 s, 780.0 tps, lat 10.256 ms stddev 8.140
progress: 330.0 s, 809.7 tps, lat 9.878 ms stddev 7.328
progress: 340.0 s, 808.5 tps, lat 9.898 ms stddev 7.526
progress: 350.0 s, 811.6 tps, lat 9.853 ms stddev 7.490
progress: 360.0 s, 809.9 tps, lat 9.871 ms stddev 7.591
progress: 370.0 s, 804.9 tps, lat 9.946 ms stddev 7.473
progress: 380.0 s, 819.4 tps, lat 9.761 ms stddev 7.147
progress: 390.0 s, 667.0 tps, lat 11.871 ms stddev 16.130
progress: 400.0 s, 527.5 tps, lat 15.174 ms stddev 23.716
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
^C
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:00:40 UTC; 2s ago
    Process: 9357 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9357 (code=exited, status=0/SUCCESS)

Oct 23 21:00:40 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:00:40 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 802.9 tps, lat 9.935 ms stddev 7.689
progress: 20.0 s, 807.1 tps, lat 9.911 ms stddev 7.452
progress: 30.0 s, 808.6 tps, lat 9.890 ms stddev 7.515
progress: 40.0 s, 802.8 tps, lat 9.965 ms stddev 7.862
progress: 50.0 s, 809.3 tps, lat 9.880 ms stddev 7.316
progress: 60.0 s, 802.8 tps, lat 9.968 ms stddev 7.333
progress: 70.0 s, 820.0 tps, lat 9.757 ms stddev 7.318
progress: 80.0 s, 823.7 tps, lat 9.711 ms stddev 7.175
progress: 90.0 s, 809.3 tps, lat 9.885 ms stddev 7.301
progress: 100.0 s, 811.7 tps, lat 9.854 ms stddev 7.045
progress: 110.0 s, 803.7 tps, lat 9.952 ms stddev 7.227
progress: 120.0 s, 791.5 tps, lat 10.109 ms stddev 7.204
progress: 130.0 s, 814.6 tps, lat 9.819 ms stddev 6.980
progress: 140.0 s, 805.6 tps, lat 9.919 ms stddev 7.189
progress: 150.0 s, 809.7 tps, lat 9.891 ms stddev 6.869
progress: 160.0 s, 815.7 tps, lat 9.805 ms stddev 7.054
progress: 170.0 s, 816.8 tps, lat 9.793 ms stddev 6.632
progress: 180.0 s, 817.0 tps, lat 9.791 ms stddev 6.898
progress: 190.0 s, 809.8 tps, lat 9.877 ms stddev 6.387
progress: 200.0 s, 810.0 tps, lat 9.874 ms stddev 6.476
progress: 210.0 s, 810.8 tps, lat 9.869 ms stddev 6.711
progress: 220.0 s, 811.3 tps, lat 9.859 ms stddev 6.444
progress: 230.0 s, 805.4 tps, lat 9.932 ms stddev 6.396
progress: 240.0 s, 816.4 tps, lat 9.796 ms stddev 6.279
progress: 250.0 s, 805.0 tps, lat 9.938 ms stddev 6.244
progress: 260.0 s, 796.1 tps, lat 10.045 ms stddev 6.346
progress: 270.0 s, 797.2 tps, lat 10.039 ms stddev 6.527
progress: 280.0 s, 788.1 tps, lat 10.148 ms stddev 6.288
progress: 290.0 s, 790.5 tps, lat 10.121 ms stddev 6.368
progress: 300.0 s, 759.9 tps, lat 10.525 ms stddev 7.218
progress: 310.0 s, 772.7 tps, lat 10.347 ms stddev 7.959
progress: 320.0 s, 779.3 tps, lat 10.266 ms stddev 7.459
progress: 330.0 s, 772.4 tps, lat 10.358 ms stddev 7.971
progress: 340.0 s, 764.2 tps, lat 10.469 ms stddev 7.729
progress: 350.0 s, 777.4 tps, lat 10.288 ms stddev 7.946
progress: 360.0 s, 788.1 tps, lat 10.149 ms stddev 7.827
progress: 370.0 s, 797.6 tps, lat 10.032 ms stddev 7.817
progress: 380.0 s, 809.2 tps, lat 9.884 ms stddev 7.505
progress: 390.0 s, 795.7 tps, lat 10.052 ms stddev 7.483
progress: 400.0 s, 805.6 tps, lat 9.930 ms stddev 7.818
progress: 410.0 s, 799.7 tps, lat 10.003 ms stddev 7.612
progress: 420.0 s, 748.7 tps, lat 10.685 ms stddev 7.884
progress: 430.0 s, 795.9 tps, lat 10.049 ms stddev 7.445
progress: 440.0 s, 563.7 tps, lat 14.192 ms stddev 20.746
progress: 450.0 s, 505.7 tps, lat 15.810 ms stddev 24.155
progress: 460.0 s, 500.4 tps, lat 15.990 ms stddev 25.284
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:10:09 UTC; 3s ago
    Process: 9543 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9543 (code=exited, status=0/SUCCESS)

Oct 23 21:10:09 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:10:09 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 810.6 tps, lat 9.840 ms stddev 7.366
progress: 20.0 s, 802.5 tps, lat 9.968 ms stddev 7.448
progress: 30.0 s, 809.5 tps, lat 9.880 ms stddev 7.031
progress: 40.0 s, 804.4 tps, lat 9.941 ms stddev 7.111
progress: 50.0 s, 754.3 tps, lat 10.608 ms stddev 7.516
progress: 60.0 s, 767.9 tps, lat 10.418 ms stddev 7.444
progress: 70.0 s, 818.1 tps, lat 9.776 ms stddev 7.104
progress: 80.0 s, 809.4 tps, lat 9.884 ms stddev 7.005
progress: 90.0 s, 807.4 tps, lat 9.908 ms stddev 6.880
progress: 100.0 s, 808.6 tps, lat 9.890 ms stddev 6.755
progress: 110.0 s, 815.9 tps, lat 9.805 ms stddev 6.539
progress: 120.0 s, 806.2 tps, lat 9.922 ms stddev 6.548
progress: 130.0 s, 819.2 tps, lat 9.765 ms stddev 6.565
progress: 140.0 s, 808.2 tps, lat 9.899 ms stddev 6.481
progress: 150.0 s, 814.8 tps, lat 9.818 ms stddev 6.555
progress: 160.0 s, 791.6 tps, lat 10.105 ms stddev 6.606
progress: 170.0 s, 809.4 tps, lat 9.883 ms stddev 6.626
progress: 180.0 s, 816.7 tps, lat 9.795 ms stddev 6.372
progress: 190.0 s, 821.5 tps, lat 9.733 ms stddev 6.478
progress: 200.0 s, 814.5 tps, lat 9.823 ms stddev 6.543
progress: 210.0 s, 817.9 tps, lat 9.782 ms stddev 6.317
progress: 220.0 s, 809.1 tps, lat 9.885 ms stddev 6.368
progress: 230.0 s, 816.3 tps, lat 9.800 ms stddev 6.407
progress: 240.0 s, 815.3 tps, lat 9.810 ms stddev 7.340
progress: 250.0 s, 811.2 tps, lat 9.862 ms stddev 7.388
progress: 260.0 s, 803.8 tps, lat 9.950 ms stddev 7.465
progress: 270.0 s, 800.9 tps, lat 9.989 ms stddev 7.452
progress: 280.0 s, 812.2 tps, lat 9.848 ms stddev 7.592
progress: 290.0 s, 804.1 tps, lat 9.948 ms stddev 7.813
progress: 300.0 s, 793.8 tps, lat 10.077 ms stddev 8.639
progress: 310.0 s, 816.1 tps, lat 9.798 ms stddev 7.508
progress: 320.0 s, 807.4 tps, lat 9.913 ms stddev 7.506
progress: 330.0 s, 818.9 tps, lat 9.769 ms stddev 7.531
progress: 340.0 s, 823.2 tps, lat 9.716 ms stddev 7.245
progress: 350.0 s, 812.2 tps, lat 9.849 ms stddev 7.707
progress: 360.0 s, 802.0 tps, lat 9.975 ms stddev 7.436
progress: 370.0 s, 772.1 tps, lat 10.358 ms stddev 7.814
progress: 380.0 s, 747.4 tps, lat 10.703 ms stddev 7.891
progress: 390.0 s, 810.5 tps, lat 9.868 ms stddev 7.488
progress: 400.0 s, 804.7 tps, lat 9.943 ms stddev 7.628
progress: 410.0 s, 539.6 tps, lat 14.826 ms stddev 22.906
progress: 420.0 s, 517.2 tps, lat 15.465 ms stddev 23.716
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:17:56 UTC; 2s ago
    Process: 9697 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9697 (code=exited, status=0/SUCCESS)

Oct 23 21:17:56 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:17:56 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 788.4 tps, lat 10.116 ms stddev 7.643
progress: 20.0 s, 805.2 tps, lat 9.932 ms stddev 7.459
progress: 30.0 s, 800.8 tps, lat 9.986 ms stddev 7.365
progress: 40.0 s, 808.5 tps, lat 9.898 ms stddev 7.302
progress: 50.0 s, 805.5 tps, lat 9.929 ms stddev 7.202
progress: 60.0 s, 813.4 tps, lat 9.834 ms stddev 7.116
progress: 70.0 s, 807.3 tps, lat 9.912 ms stddev 7.204
progress: 80.0 s, 822.6 tps, lat 9.719 ms stddev 7.024
progress: 90.0 s, 827.5 tps, lat 9.667 ms stddev 6.777
progress: 100.0 s, 821.9 tps, lat 9.736 ms stddev 7.070
progress: 110.0 s, 829.6 tps, lat 9.641 ms stddev 6.543
progress: 120.0 s, 827.1 tps, lat 9.671 ms stddev 7.106
progress: 130.0 s, 817.3 tps, lat 9.787 ms stddev 6.658
progress: 140.0 s, 830.9 tps, lat 9.628 ms stddev 6.665
progress: 150.0 s, 817.2 tps, lat 9.789 ms stddev 6.471
progress: 160.0 s, 823.1 tps, lat 9.719 ms stddev 6.795
progress: 170.0 s, 819.4 tps, lat 9.761 ms stddev 6.871
progress: 180.0 s, 822.1 tps, lat 9.731 ms stddev 6.641
progress: 190.0 s, 822.2 tps, lat 9.730 ms stddev 6.415
progress: 200.0 s, 824.0 tps, lat 9.707 ms stddev 6.279
progress: 210.0 s, 798.4 tps, lat 10.015 ms stddev 6.616
progress: 220.0 s, 758.9 tps, lat 10.544 ms stddev 8.061
progress: 230.0 s, 816.1 tps, lat 9.799 ms stddev 7.320
progress: 240.0 s, 818.7 tps, lat 9.775 ms stddev 7.426
progress: 250.0 s, 816.5 tps, lat 9.797 ms stddev 7.410
progress: 260.0 s, 819.3 tps, lat 9.761 ms stddev 7.542
progress: 270.0 s, 825.0 tps, lat 9.695 ms stddev 7.278
progress: 280.0 s, 820.4 tps, lat 9.755 ms stddev 7.625
progress: 290.0 s, 815.2 tps, lat 9.812 ms stddev 7.361
progress: 300.0 s, 798.1 tps, lat 10.018 ms stddev 7.767
progress: 310.0 s, 818.3 tps, lat 9.776 ms stddev 7.698
progress: 320.0 s, 775.7 tps, lat 10.230 ms stddev 10.612
progress: 330.0 s, 534.0 tps, lat 14.968 ms stddev 23.342
progress: 340.0 s, 536.1 tps, lat 14.931 ms stddev 23.580
progress: 350.0 s, 535.5 tps, lat 14.935 ms stddev 23.381
progress: 360.0 s, 516.7 tps, lat 15.480 ms stddev 25.259
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:25:56 UTC; 2s ago
    Process: 9852 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 9852 (code=exited, status=0/SUCCESS)

Oct 23 21:25:56 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:25:56 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 803.0 tps, lat 9.935 ms stddev 7.636
progress: 20.0 s, 809.8 tps, lat 9.877 ms stddev 7.554
progress: 30.0 s, 795.9 tps, lat 10.050 ms stddev 8.560
progress: 40.0 s, 808.8 tps, lat 9.886 ms stddev 7.550
progress: 50.0 s, 736.0 tps, lat 10.869 ms stddev 7.862
progress: 60.0 s, 807.1 tps, lat 9.912 ms stddev 7.512
progress: 70.0 s, 814.6 tps, lat 9.820 ms stddev 7.220
progress: 80.0 s, 825.2 tps, lat 9.694 ms stddev 7.016
progress: 90.0 s, 817.2 tps, lat 9.787 ms stddev 6.833
progress: 100.0 s, 827.5 tps, lat 9.671 ms stddev 6.788
progress: 110.0 s, 828.4 tps, lat 9.655 ms stddev 6.621
progress: 120.0 s, 827.2 tps, lat 9.670 ms stddev 6.629
progress: 130.0 s, 793.5 tps, lat 10.079 ms stddev 7.027
progress: 140.0 s, 821.5 tps, lat 9.738 ms stddev 6.582
progress: 150.0 s, 832.3 tps, lat 9.611 ms stddev 6.547
progress: 160.0 s, 823.7 tps, lat 9.712 ms stddev 6.417
progress: 170.0 s, 832.8 tps, lat 9.605 ms stddev 6.307
progress: 180.0 s, 821.0 tps, lat 9.743 ms stddev 6.660
progress: 190.0 s, 827.9 tps, lat 9.662 ms stddev 6.302
progress: 200.0 s, 829.4 tps, lat 9.645 ms stddev 6.318
progress: 210.0 s, 824.8 tps, lat 9.697 ms stddev 6.994
progress: 220.0 s, 825.5 tps, lat 9.689 ms stddev 7.188
progress: 230.0 s, 816.9 tps, lat 9.794 ms stddev 7.368
progress: 240.0 s, 828.7 tps, lat 9.652 ms stddev 7.003
progress: 250.0 s, 827.1 tps, lat 9.669 ms stddev 7.207
progress: 260.0 s, 820.6 tps, lat 9.753 ms stddev 7.553
progress: 270.0 s, 824.7 tps, lat 9.694 ms stddev 7.464
progress: 280.0 s, 825.5 tps, lat 9.694 ms stddev 7.510
progress: 290.0 s, 813.4 tps, lat 9.834 ms stddev 8.920
progress: 300.0 s, 818.4 tps, lat 9.771 ms stddev 7.335
progress: 310.0 s, 835.0 tps, lat 9.584 ms stddev 7.404
progress: 320.0 s, 830.4 tps, lat 9.633 ms stddev 7.399
progress: 330.0 s, 827.1 tps, lat 9.671 ms stddev 7.290
progress: 340.0 s, 836.0 tps, lat 9.569 ms stddev 7.463
progress: 350.0 s, 826.1 tps, lat 9.683 ms stddev 7.636
progress: 360.0 s, 773.2 tps, lat 10.339 ms stddev 8.006
progress: 370.0 s, 764.4 tps, lat 10.470 ms stddev 8.166
progress: 380.0 s, 809.9 tps, lat 9.875 ms stddev 7.341
progress: 390.0 s, 805.0 tps, lat 9.937 ms stddev 7.406
progress: 400.0 s, 524.7 tps, lat 15.248 ms stddev 24.322
progress: 410.0 s, 514.5 tps, lat 15.546 ms stddev 24.904
progress: 420.0 s, 519.2 tps, lat 15.406 ms stddev 23.848
progress: 430.0 s, 516.0 tps, lat 15.480 ms stddev 23.971
progress: 440.0 s, 526.6 tps, lat 15.213 ms stddev 22.920
progress: 450.0 s, 522.8 tps, lat 15.263 ms stddev 23.224
progress: 460.0 s, 529.5 tps, lat 15.143 ms stddev 23.046
progress: 470.0 s, 529.1 tps, lat 15.123 ms stddev 23.094
progress: 480.0 s, 523.6 tps, lat 15.249 ms stddev 23.376
progress: 490.0 s, 514.7 tps, lat 15.571 ms stddev 24.401
progress: 500.0 s, 515.2 tps, lat 15.525 ms stddev 24.324
progress: 510.0 s, 528.8 tps, lat 15.126 ms stddev 22.964
progress: 520.0 s, 516.6 tps, lat 15.483 ms stddev 24.246
progress: 530.0 s, 533.1 tps, lat 14.945 ms stddev 23.170
progress: 540.0 s, 533.6 tps, lat 15.031 ms stddev 23.553
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:36:46 UTC; 3s ago
    Process: 10042 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 10042 (code=exited, status=0/SUCCESS)

Oct 23 21:36:46 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:36:46 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 811.5 tps, lat 9.825 ms stddev 7.693
progress: 20.0 s, 816.5 tps, lat 9.800 ms stddev 7.368
progress: 30.0 s, 746.0 tps, lat 10.719 ms stddev 7.815
progress: 40.0 s, 738.3 tps, lat 10.833 ms stddev 7.921
progress: 50.0 s, 797.5 tps, lat 10.033 ms stddev 7.107
progress: 60.0 s, 806.1 tps, lat 9.925 ms stddev 7.201
progress: 70.0 s, 804.2 tps, lat 9.942 ms stddev 6.908
progress: 80.0 s, 811.0 tps, lat 9.869 ms stddev 7.123
progress: 90.0 s, 790.7 tps, lat 10.112 ms stddev 7.277
progress: 100.0 s, 825.9 tps, lat 9.685 ms stddev 6.744
progress: 110.0 s, 825.3 tps, lat 9.690 ms stddev 6.966
progress: 120.0 s, 830.2 tps, lat 9.640 ms stddev 6.732
progress: 130.0 s, 835.4 tps, lat 9.575 ms stddev 6.634
progress: 140.0 s, 828.2 tps, lat 9.661 ms stddev 6.597
progress: 150.0 s, 840.6 tps, lat 9.513 ms stddev 6.504
progress: 160.0 s, 826.6 tps, lat 9.679 ms stddev 6.795
progress: 170.0 s, 822.3 tps, lat 9.728 ms stddev 7.118
progress: 180.0 s, 828.3 tps, lat 9.658 ms stddev 7.736
progress: 190.0 s, 823.0 tps, lat 9.721 ms stddev 7.331
progress: 200.0 s, 816.6 tps, lat 9.795 ms stddev 7.232
progress: 210.0 s, 828.0 tps, lat 9.660 ms stddev 6.315
progress: 220.0 s, 821.6 tps, lat 9.737 ms stddev 6.436
progress: 230.0 s, 818.2 tps, lat 9.772 ms stddev 6.909
progress: 240.0 s, 814.9 tps, lat 9.821 ms stddev 7.631
progress: 250.0 s, 821.7 tps, lat 9.732 ms stddev 7.454
progress: 260.0 s, 818.9 tps, lat 9.768 ms stddev 7.508
progress: 270.0 s, 822.6 tps, lat 9.727 ms stddev 7.343
progress: 280.0 s, 810.0 tps, lat 9.869 ms stddev 7.631
progress: 290.0 s, 818.6 tps, lat 9.777 ms stddev 7.485
progress: 300.0 s, 826.3 tps, lat 9.682 ms stddev 7.265
progress: 310.0 s, 822.5 tps, lat 9.725 ms stddev 7.424
progress: 320.0 s, 826.6 tps, lat 9.678 ms stddev 7.426
progress: 330.0 s, 831.0 tps, lat 9.624 ms stddev 7.409
progress: 340.0 s, 816.4 tps, lat 9.799 ms stddev 7.310
progress: 350.0 s, 765.1 tps, lat 10.454 ms stddev 8.329
progress: 360.0 s, 796.9 tps, lat 10.036 ms stddev 7.877
progress: 370.0 s, 817.3 tps, lat 9.790 ms stddev 7.142
progress: 380.0 s, 778.9 tps, lat 10.184 ms stddev 9.722
progress: 390.0 s, 526.7 tps, lat 15.190 ms stddev 23.671
progress: 400.0 s, 529.1 tps, lat 15.114 ms stddev 22.759
progress: 410.0 s, 517.3 tps, lat 15.457 ms stddev 24.860
progress: 420.0 s, 526.6 tps, lat 15.205 ms stddev 23.687
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:45:22 UTC; 2s ago
    Process: 10200 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 10200 (code=exited, status=0/SUCCESS)

Oct 23 21:45:22 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:45:22 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 804.0 tps, lat 9.920 ms stddev 7.851
progress: 20.0 s, 808.7 tps, lat 9.887 ms stddev 7.338
progress: 30.0 s, 812.2 tps, lat 9.843 ms stddev 7.272
progress: 40.0 s, 797.6 tps, lat 10.027 ms stddev 7.290
progress: 50.0 s, 793.5 tps, lat 10.091 ms stddev 7.129
progress: 60.0 s, 810.1 tps, lat 9.873 ms stddev 7.003
progress: 70.0 s, 812.2 tps, lat 9.852 ms stddev 6.944
progress: 80.0 s, 817.3 tps, lat 9.786 ms stddev 6.968
progress: 90.0 s, 800.3 tps, lat 9.994 ms stddev 7.207
progress: 100.0 s, 818.9 tps, lat 9.769 ms stddev 6.821
progress: 110.0 s, 810.7 tps, lat 9.866 ms stddev 6.911
progress: 120.0 s, 811.2 tps, lat 9.857 ms stddev 6.706
progress: 130.0 s, 799.8 tps, lat 10.005 ms stddev 6.612
progress: 140.0 s, 812.7 tps, lat 9.843 ms stddev 6.881
progress: 150.0 s, 771.3 tps, lat 10.371 ms stddev 7.039
progress: 160.0 s, 821.9 tps, lat 9.730 ms stddev 6.488
progress: 170.0 s, 816.7 tps, lat 9.798 ms stddev 6.275
progress: 180.0 s, 825.2 tps, lat 9.694 ms stddev 6.292
progress: 190.0 s, 782.6 tps, lat 10.222 ms stddev 7.062
progress: 200.0 s, 831.6 tps, lat 9.618 ms stddev 6.442
progress: 210.0 s, 823.9 tps, lat 9.706 ms stddev 6.258
progress: 220.0 s, 810.8 tps, lat 9.867 ms stddev 6.911
progress: 230.0 s, 820.4 tps, lat 9.748 ms stddev 6.858
progress: 240.0 s, 817.8 tps, lat 9.781 ms stddev 7.524
progress: 250.0 s, 820.6 tps, lat 9.745 ms stddev 7.635
progress: 260.0 s, 819.0 tps, lat 9.772 ms stddev 7.809
progress: 270.0 s, 821.3 tps, lat 9.738 ms stddev 7.433
progress: 280.0 s, 825.0 tps, lat 9.695 ms stddev 7.272
progress: 290.0 s, 803.4 tps, lat 9.954 ms stddev 7.839
progress: 300.0 s, 813.8 tps, lat 9.833 ms stddev 7.525
progress: 310.0 s, 813.5 tps, lat 9.834 ms stddev 7.593
progress: 320.0 s, 825.7 tps, lat 9.688 ms stddev 7.475
progress: 330.0 s, 816.2 tps, lat 9.800 ms stddev 7.586
progress: 340.0 s, 813.5 tps, lat 9.831 ms stddev 7.500
progress: 350.0 s, 813.7 tps, lat 9.834 ms stddev 7.479
progress: 360.0 s, 816.6 tps, lat 9.792 ms stddev 7.289
progress: 370.0 s, 816.4 tps, lat 9.798 ms stddev 7.527
progress: 380.0 s, 810.3 tps, lat 9.874 ms stddev 7.772
progress: 390.0 s, 798.1 tps, lat 10.024 ms stddev 7.601
progress: 400.0 s, 794.1 tps, lat 10.071 ms stddev 9.087
progress: 410.0 s, 529.1 tps, lat 15.117 ms stddev 23.300
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 21:54:12 UTC; 4s ago
    Process: 10340 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 10340 (code=exited, status=0/SUCCESS)

Oct 23 21:54:12 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 21:54:12 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 786.2 tps, lat 10.145 ms stddev 7.653
progress: 20.0 s, 801.0 tps, lat 9.986 ms stddev 7.736
progress: 30.0 s, 810.0 tps, lat 9.877 ms stddev 7.155
progress: 40.0 s, 796.3 tps, lat 10.047 ms stddev 7.262
progress: 50.0 s, 802.7 tps, lat 9.964 ms stddev 7.097
progress: 60.0 s, 819.5 tps, lat 9.761 ms stddev 6.969
progress: 70.0 s, 801.3 tps, lat 9.983 ms stddev 6.999
progress: 80.0 s, 794.7 tps, lat 10.066 ms stddev 7.104
progress: 90.0 s, 795.2 tps, lat 10.056 ms stddev 6.910
progress: 100.0 s, 794.5 tps, lat 10.072 ms stddev 7.167
progress: 110.0 s, 793.3 tps, lat 10.080 ms stddev 6.899
progress: 120.0 s, 801.7 tps, lat 9.979 ms stddev 6.728
progress: 130.0 s, 802.1 tps, lat 9.969 ms stddev 6.561
progress: 140.0 s, 821.0 tps, lat 9.742 ms stddev 6.221
progress: 150.0 s, 815.4 tps, lat 9.817 ms stddev 6.511
progress: 160.0 s, 822.3 tps, lat 9.728 ms stddev 6.289
progress: 170.0 s, 813.8 tps, lat 9.828 ms stddev 6.496
progress: 180.0 s, 806.9 tps, lat 9.910 ms stddev 7.073
progress: 190.0 s, 803.4 tps, lat 9.958 ms stddev 7.218
progress: 200.0 s, 820.0 tps, lat 9.759 ms stddev 7.398
progress: 210.0 s, 822.6 tps, lat 9.719 ms stddev 7.458
progress: 220.0 s, 821.7 tps, lat 9.738 ms stddev 7.538
progress: 230.0 s, 812.4 tps, lat 9.846 ms stddev 7.472
progress: 240.0 s, 815.1 tps, lat 9.813 ms stddev 7.568
progress: 250.0 s, 765.9 tps, lat 10.443 ms stddev 7.930
progress: 260.0 s, 767.5 tps, lat 10.422 ms stddev 8.197
progress: 270.0 s, 824.7 tps, lat 9.691 ms stddev 7.367
progress: 280.0 s, 823.8 tps, lat 9.720 ms stddev 7.474
progress: 290.0 s, 805.8 tps, lat 9.926 ms stddev 9.114
progress: 300.0 s, 827.6 tps, lat 9.667 ms stddev 7.520
progress: 310.0 s, 812.9 tps, lat 9.837 ms stddev 7.322
progress: 320.0 s, 798.2 tps, lat 10.022 ms stddev 7.779
progress: 330.0 s, 810.7 tps, lat 9.867 ms stddev 7.449
progress: 340.0 s, 798.0 tps, lat 10.025 ms stddev 7.534
progress: 350.0 s, 793.3 tps, lat 10.082 ms stddev 7.706
progress: 360.0 s, 816.0 tps, lat 9.801 ms stddev 7.348
progress: 370.0 s, 796.9 tps, lat 10.042 ms stddev 7.549
progress: 380.0 s, 794.8 tps, lat 10.063 ms stddev 7.867
progress: 390.0 s, 797.2 tps, lat 9.996 ms stddev 7.693
progress: 400.0 s, 512.5 tps, lat 15.575 ms stddev 24.099
progress: 410.0 s, 509.3 tps, lat 15.720 ms stddev 23.208
progress: 420.0 s, 516.3 tps, lat 15.494 ms stddev 23.840
progress: 430.0 s, 518.4 tps, lat 15.432 ms stddev 23.964
progress: 440.0 s, 535.8 tps, lat 14.930 ms stddev 22.962
progress: 450.0 s, 532.6 tps, lat 14.999 ms stddev 22.932
progress: 460.0 s, 520.6 tps, lat 15.349 ms stddev 24.800
progress: 470.0 s, 545.4 tps, lat 14.679 ms stddev 22.385
progress: 480.0 s, 532.4 tps, lat 15.022 ms stddev 23.099
progress: 490.0 s, 511.7 tps, lat 15.635 ms stddev 23.816
progress: 500.0 s, 525.2 tps, lat 15.221 ms stddev 22.634
progress: 510.0 s, 516.9 tps, lat 15.499 ms stddev 22.804
progress: 520.0 s, 521.7 tps, lat 15.316 ms stddev 22.855
progress: 530.0 s, 521.2 tps, lat 15.335 ms stddev 23.868
progress: 540.0 s, 518.6 tps, lat 15.427 ms stddev 22.744
progress: 550.0 s, 502.3 tps, lat 15.916 ms stddev 25.465
progress: 560.0 s, 514.5 tps, lat 15.543 ms stddev 23.417
progress: 570.0 s, 488.6 tps, lat 16.368 ms stddev 24.039
progress: 580.0 s, 520.9 tps, lat 15.354 ms stddev 23.645
progress: 590.0 s, 518.7 tps, lat 15.446 ms stddev 24.054
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 22:06:44 UTC; 4s ago
    Process: 10504 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 10504 (code=exited, status=0/SUCCESS)

Oct 23 22:06:44 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 22:06:44 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 823.3 tps, lat 9.688 ms stddev 7.455
progress: 20.0 s, 818.2 tps, lat 9.773 ms stddev 7.387
progress: 30.0 s, 818.7 tps, lat 9.774 ms stddev 7.282
progress: 40.0 s, 828.7 tps, lat 9.653 ms stddev 7.019
progress: 50.0 s, 828.4 tps, lat 9.657 ms stddev 7.072
progress: 60.0 s, 818.5 tps, lat 9.772 ms stddev 7.152
progress: 70.0 s, 829.2 tps, lat 9.648 ms stddev 6.917
progress: 80.0 s, 825.8 tps, lat 9.686 ms stddev 6.736
progress: 90.0 s, 817.7 tps, lat 9.781 ms stddev 6.744
progress: 100.0 s, 818.0 tps, lat 9.781 ms stddev 6.805
progress: 110.0 s, 828.5 tps, lat 9.656 ms stddev 6.867
progress: 120.0 s, 822.6 tps, lat 9.719 ms stddev 6.571
progress: 130.0 s, 770.4 tps, lat 10.383 ms stddev 6.993
progress: 140.0 s, 836.7 tps, lat 9.566 ms stddev 6.745
progress: 150.0 s, 829.3 tps, lat 9.646 ms stddev 6.341
progress: 160.0 s, 822.5 tps, lat 9.725 ms stddev 6.409
progress: 170.0 s, 834.7 tps, lat 9.582 ms stddev 6.277
progress: 180.0 s, 818.9 tps, lat 9.761 ms stddev 6.727
progress: 190.0 s, 820.7 tps, lat 9.754 ms stddev 7.495
progress: 200.0 s, 828.5 tps, lat 9.657 ms stddev 6.616
progress: 210.0 s, 819.4 tps, lat 9.760 ms stddev 7.318
progress: 220.0 s, 816.4 tps, lat 9.797 ms stddev 7.512
progress: 230.0 s, 809.6 tps, lat 9.880 ms stddev 7.358
progress: 240.0 s, 813.4 tps, lat 9.836 ms stddev 7.462
progress: 250.0 s, 814.4 tps, lat 9.826 ms stddev 7.665
progress: 260.0 s, 816.5 tps, lat 9.795 ms stddev 7.599
progress: 270.0 s, 821.4 tps, lat 9.738 ms stddev 7.392
progress: 280.0 s, 824.5 tps, lat 9.703 ms stddev 7.265
progress: 290.0 s, 794.2 tps, lat 10.066 ms stddev 8.861
progress: 300.0 s, 813.7 tps, lat 9.834 ms stddev 7.367
progress: 310.0 s, 822.4 tps, lat 9.728 ms stddev 7.434
progress: 320.0 s, 812.8 tps, lat 9.841 ms stddev 7.309
progress: 330.0 s, 808.0 tps, lat 9.897 ms stddev 7.489
progress: 340.0 s, 816.2 tps, lat 9.794 ms stddev 7.437
progress: 350.0 s, 828.4 tps, lat 9.667 ms stddev 7.440
progress: 360.0 s, 823.9 tps, lat 9.697 ms stddev 7.398
progress: 370.0 s, 815.2 tps, lat 9.822 ms stddev 7.568
progress: 380.0 s, 813.9 tps, lat 9.831 ms stddev 7.550
progress: 390.0 s, 735.6 tps, lat 10.871 ms stddev 13.013
progress: 400.0 s, 526.8 tps, lat 15.179 ms stddev 23.837
progress: 410.0 s, 523.7 tps, lat 15.284 ms stddev 24.436
progress: 420.0 s, 529.4 tps, lat 15.111 ms stddev 23.160
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 22:14:47 UTC; 2s ago
    Process: 10658 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 10658 (code=exited, status=0/SUCCESS)

Oct 23 22:14:47 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 22:14:47 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 811.2 tps, lat 9.832 ms stddev 7.464
progress: 20.0 s, 815.9 tps, lat 9.799 ms stddev 6.934
progress: 30.0 s, 822.7 tps, lat 9.729 ms stddev 7.175
progress: 40.0 s, 822.6 tps, lat 9.725 ms stddev 6.947
progress: 50.0 s, 827.3 tps, lat 9.664 ms stddev 7.088
progress: 60.0 s, 831.5 tps, lat 9.623 ms stddev 7.226
progress: 70.0 s, 827.9 tps, lat 9.663 ms stddev 7.045
progress: 80.0 s, 820.8 tps, lat 9.745 ms stddev 7.100
progress: 90.0 s, 836.5 tps, lat 9.564 ms stddev 6.335
progress: 100.0 s, 830.0 tps, lat 9.633 ms stddev 6.928
progress: 110.0 s, 828.5 tps, lat 9.660 ms stddev 6.643
progress: 120.0 s, 835.7 tps, lat 9.570 ms stddev 6.814
progress: 130.0 s, 832.2 tps, lat 9.612 ms stddev 6.582
progress: 140.0 s, 833.6 tps, lat 9.597 ms stddev 6.644
progress: 150.0 s, 842.7 tps, lat 9.492 ms stddev 6.670
progress: 160.0 s, 834.6 tps, lat 9.583 ms stddev 6.572
progress: 170.0 s, 840.0 tps, lat 9.524 ms stddev 6.195
progress: 180.0 s, 837.8 tps, lat 9.546 ms stddev 6.850
progress: 190.0 s, 842.0 tps, lat 9.500 ms stddev 6.599
progress: 200.0 s, 823.0 tps, lat 9.720 ms stddev 7.620
progress: 210.0 s, 849.5 tps, lat 9.412 ms stddev 7.051
progress: 220.0 s, 837.2 tps, lat 9.555 ms stddev 7.496
progress: 230.0 s, 843.9 tps, lat 9.484 ms stddev 7.431
progress: 240.0 s, 843.0 tps, lat 9.489 ms stddev 7.281
progress: 250.0 s, 825.7 tps, lat 9.688 ms stddev 7.790
progress: 260.0 s, 829.7 tps, lat 9.638 ms stddev 7.302
progress: 270.0 s, 847.7 tps, lat 9.437 ms stddev 7.206
progress: 280.0 s, 769.0 tps, lat 10.401 ms stddev 8.027
progress: 290.0 s, 742.6 tps, lat 10.774 ms stddev 9.718
progress: 300.0 s, 839.1 tps, lat 9.527 ms stddev 7.687
progress: 310.0 s, 839.7 tps, lat 9.531 ms stddev 7.645
progress: 320.0 s, 840.0 tps, lat 9.521 ms stddev 7.344
progress: 330.0 s, 844.6 tps, lat 9.469 ms stddev 7.343
progress: 340.0 s, 835.6 tps, lat 9.576 ms stddev 7.509
progress: 350.0 s, 830.6 tps, lat 9.629 ms stddev 7.221
progress: 360.0 s, 846.5 tps, lat 9.452 ms stddev 7.256
progress: 370.0 s, 824.1 tps, lat 9.707 ms stddev 7.287
progress: 380.0 s, 826.4 tps, lat 9.678 ms stddev 7.349
progress: 390.1 s, 822.1 tps, lat 9.643 ms stddev 7.892
progress: 400.1 s, 515.3 tps, lat 15.530 ms stddev 26.060
progress: 410.1 s, 547.1 tps, lat 14.618 ms stddev 23.166
progress: 420.0 s, 541.2 tps, lat 14.908 ms stddev 23.794
progress: 430.1 s, 537.9 tps, lat 14.749 ms stddev 23.274
progress: 440.0 s, 542.8 tps, lat 14.865 ms stddev 23.197
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 22:24:11 UTC; 5s ago
    Process: 10832 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 10832 (code=exited, status=0/SUCCESS)

Oct 23 22:24:11 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 22:24:11 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 855.4 tps, lat 9.321 ms stddev 7.349
progress: 20.0 s, 863.4 tps, lat 9.263 ms stddev 7.085
progress: 30.0 s, 825.2 tps, lat 9.698 ms stddev 7.222
progress: 40.0 s, 845.4 tps, lat 9.461 ms stddev 7.100
progress: 50.0 s, 863.6 tps, lat 9.263 ms stddev 7.043
progress: 60.0 s, 860.6 tps, lat 9.295 ms stddev 6.753
progress: 70.0 s, 851.7 tps, lat 9.393 ms stddev 6.815
progress: 80.0 s, 866.4 tps, lat 9.234 ms stddev 6.886
progress: 90.0 s, 867.4 tps, lat 9.219 ms stddev 6.800
progress: 100.0 s, 851.6 tps, lat 9.394 ms stddev 6.851
progress: 110.0 s, 860.4 tps, lat 9.297 ms stddev 6.845
progress: 120.0 s, 858.4 tps, lat 9.320 ms stddev 6.608
progress: 130.0 s, 858.1 tps, lat 9.323 ms stddev 6.545
progress: 140.0 s, 857.5 tps, lat 9.327 ms stddev 6.910
progress: 150.0 s, 850.7 tps, lat 9.403 ms stddev 6.968
progress: 160.0 s, 854.6 tps, lat 9.361 ms stddev 7.143
progress: 170.0 s, 852.0 tps, lat 9.389 ms stddev 7.385
progress: 180.0 s, 855.5 tps, lat 9.350 ms stddev 7.220
progress: 190.0 s, 853.9 tps, lat 9.367 ms stddev 7.287
progress: 200.0 s, 856.3 tps, lat 9.342 ms stddev 7.612
progress: 210.0 s, 849.6 tps, lat 9.415 ms stddev 7.378
progress: 220.0 s, 864.8 tps, lat 9.252 ms stddev 7.283
progress: 230.0 s, 854.5 tps, lat 9.358 ms stddev 7.443
progress: 240.0 s, 848.8 tps, lat 9.425 ms stddev 7.297
progress: 250.0 s, 841.5 tps, lat 9.503 ms stddev 7.312
progress: 260.0 s, 851.3 tps, lat 9.402 ms stddev 7.169
progress: 270.0 s, 843.2 tps, lat 9.485 ms stddev 7.486
progress: 280.0 s, 858.4 tps, lat 9.317 ms stddev 7.310
progress: 290.0 s, 858.4 tps, lat 9.315 ms stddev 7.246
progress: 300.0 s, 812.4 tps, lat 9.849 ms stddev 7.694
progress: 310.0 s, 852.3 tps, lat 9.387 ms stddev 7.407
progress: 320.0 s, 855.6 tps, lat 9.345 ms stddev 7.459
progress: 330.0 s, 856.6 tps, lat 9.344 ms stddev 7.528
progress: 340.0 s, 821.9 tps, lat 9.731 ms stddev 7.342
progress: 350.0 s, 831.9 tps, lat 9.614 ms stddev 7.406
progress: 360.0 s, 858.1 tps, lat 9.324 ms stddev 7.374
progress: 370.0 s, 850.1 tps, lat 9.410 ms stddev 7.167
progress: 380.0 s, 850.5 tps, lat 9.404 ms stddev 7.345
progress: 390.0 s, 862.4 tps, lat 9.274 ms stddev 7.120
progress: 400.0 s, 832.1 tps, lat 9.612 ms stddev 7.533
progress: 410.0 s, 825.2 tps, lat 9.698 ms stddev 7.466
progress: 420.0 s, 784.4 tps, lat 10.197 ms stddev 7.549
progress: 430.0 s, 769.9 tps, lat 10.308 ms stddev 9.297
progress: 440.0 s, 545.1 tps, lat 14.678 ms stddev 22.464
progress: 450.0 s, 550.9 tps, lat 14.528 ms stddev 22.368
progress: 460.0 s, 552.1 tps, lat 14.475 ms stddev 22.480
progress: 470.0 s, 557.0 tps, lat 14.350 ms stddev 22.163
progress: 480.0 s, 558.0 tps, lat 14.340 ms stddev 21.622
progress: 490.0 s, 561.9 tps, lat 14.223 ms stddev 21.417
progress: 500.0 s, 537.4 tps, lat 14.905 ms stddev 22.488
progress: 510.0 s, 541.1 tps, lat 14.782 ms stddev 22.381
progress: 520.0 s, 529.4 tps, lat 15.110 ms stddev 23.497
progress: 530.0 s, 538.1 tps, lat 14.886 ms stddev 21.543
progress: 540.0 s, 530.6 tps, lat 15.039 ms stddev 23.256
progress: 550.0 s, 543.4 tps, lat 14.735 ms stddev 22.776
progress: 560.0 s, 550.1 tps, lat 14.527 ms stddev 22.559
progress: 570.0 s, 540.4 tps, lat 14.829 ms stddev 23.045
progress: 580.0 s, 555.7 tps, lat 14.386 ms stddev 22.404
progress: 590.0 s, 550.4 tps, lat 14.536 ms stddev 22.133
progress: 600.0 s, 533.5 tps, lat 14.991 ms stddev 23.231
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 457200
latency average = 10.498 ms
latency stddev = 12.206 ms
tps = 761.926042 (including connections establishing)
tps = 761.929711 (excluding connections establishing)
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 23:02:11 UTC; 3s ago
    Process: 11373 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 11373 (code=exited, status=0/SUCCESS)

Oct 23 23:02:11 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 23:02:11 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 863.8 tps, lat 9.231 ms stddev 7.345
progress: 20.0 s, 877.4 tps, lat 9.119 ms stddev 6.917
progress: 30.0 s, 876.8 tps, lat 9.125 ms stddev 7.161
progress: 40.0 s, 878.2 tps, lat 9.107 ms stddev 6.775
progress: 50.0 s, 870.5 tps, lat 9.191 ms stddev 6.861
progress: 60.0 s, 887.1 tps, lat 9.018 ms stddev 6.626
progress: 70.0 s, 858.7 tps, lat 9.305 ms stddev 6.837
progress: 80.0 s, 873.5 tps, lat 9.163 ms stddev 6.978
progress: 90.0 s, 877.0 tps, lat 9.126 ms stddev 6.704
progress: 100.0 s, 877.4 tps, lat 9.115 ms stddev 6.529
progress: 110.0 s, 875.7 tps, lat 9.134 ms stddev 6.743
progress: 120.0 s, 876.5 tps, lat 9.126 ms stddev 6.715
progress: 130.0 s, 866.7 tps, lat 9.231 ms stddev 6.744
progress: 140.0 s, 881.1 tps, lat 9.076 ms stddev 6.384
progress: 150.0 s, 864.9 tps, lat 9.250 ms stddev 6.732
progress: 160.0 s, 852.7 tps, lat 9.381 ms stddev 7.152
progress: 170.0 s, 866.9 tps, lat 9.229 ms stddev 6.767
progress: 180.0 s, 886.4 tps, lat 9.025 ms stddev 7.004
progress: 190.0 s, 872.9 tps, lat 9.163 ms stddev 7.134
progress: 200.0 s, 872.2 tps, lat 9.171 ms stddev 7.128
progress: 210.0 s, 886.2 tps, lat 9.028 ms stddev 7.051
progress: 220.0 s, 873.6 tps, lat 9.156 ms stddev 7.427
progress: 230.0 s, 865.0 tps, lat 9.246 ms stddev 7.181
progress: 240.0 s, 874.5 tps, lat 9.149 ms stddev 7.198
progress: 250.0 s, 878.3 tps, lat 9.100 ms stddev 6.719
progress: 260.0 s, 877.3 tps, lat 9.126 ms stddev 7.332
progress: 270.0 s, 857.9 tps, lat 9.322 ms stddev 7.249
progress: 280.0 s, 839.0 tps, lat 9.533 ms stddev 7.351
progress: 290.0 s, 885.2 tps, lat 9.036 ms stddev 7.043
progress: 300.0 s, 884.0 tps, lat 9.049 ms stddev 7.158
progress: 310.0 s, 870.5 tps, lat 9.187 ms stddev 7.382
progress: 320.0 s, 882.3 tps, lat 9.071 ms stddev 7.046
progress: 330.0 s, 873.2 tps, lat 9.159 ms stddev 7.167
progress: 340.0 s, 877.3 tps, lat 9.120 ms stddev 7.064
progress: 350.0 s, 867.5 tps, lat 9.220 ms stddev 7.309
progress: 360.0 s, 870.9 tps, lat 9.182 ms stddev 7.162
progress: 370.0 s, 862.6 tps, lat 9.276 ms stddev 7.188
progress: 380.0 s, 880.1 tps, lat 9.083 ms stddev 6.957
progress: 390.0 s, 879.1 tps, lat 9.104 ms stddev 7.046
progress: 400.0 s, 875.4 tps, lat 9.137 ms stddev 6.965
progress: 410.0 s, 711.1 tps, lat 11.244 ms stddev 15.685
progress: 420.0 s, 577.9 tps, lat 13.847 ms stddev 22.233
progress: 430.0 s, 571.0 tps, lat 14.004 ms stddev 21.831
progress: 440.0 s, 543.5 tps, lat 14.729 ms stddev 24.714
progress: 450.0 s, 575.2 tps, lat 13.903 ms stddev 22.126
progress: 460.0 s, 556.8 tps, lat 14.363 ms stddev 22.383
^C
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 23:12:12 UTC; 3s ago
    Process: 11559 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 11559 (code=exited, status=0/SUCCESS)

Oct 23 23:12:12 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 23:12:12 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 789.6 tps, lat 10.099 ms stddev 7.549
progress: 20.0 s, 820.2 tps, lat 9.751 ms stddev 7.574
progress: 30.0 s, 802.9 tps, lat 9.966 ms stddev 8.896
progress: 40.0 s, 831.0 tps, lat 9.628 ms stddev 7.202
progress: 50.0 s, 815.6 tps, lat 9.806 ms stddev 7.029
progress: 60.0 s, 825.5 tps, lat 9.687 ms stddev 7.040
progress: 70.0 s, 832.5 tps, lat 9.611 ms stddev 7.016
progress: 80.0 s, 852.6 tps, lat 9.384 ms stddev 6.400
progress: 90.0 s, 844.8 tps, lat 9.468 ms stddev 6.916
progress: 100.0 s, 842.0 tps, lat 9.498 ms stddev 6.723
progress: 110.0 s, 847.9 tps, lat 9.436 ms stddev 6.825
progress: 120.0 s, 842.9 tps, lat 9.487 ms stddev 6.356
progress: 130.0 s, 845.9 tps, lat 9.458 ms stddev 6.662
progress: 140.0 s, 853.9 tps, lat 9.366 ms stddev 6.465
progress: 150.0 s, 844.7 tps, lat 9.475 ms stddev 7.058
progress: 160.0 s, 861.5 tps, lat 9.283 ms stddev 6.288
progress: 170.0 s, 819.7 tps, lat 9.761 ms stddev 7.380
progress: 180.0 s, 852.6 tps, lat 9.380 ms stddev 7.276
progress: 190.0 s, 850.0 tps, lat 9.411 ms stddev 7.166
progress: 200.0 s, 847.7 tps, lat 9.432 ms stddev 7.543
progress: 210.0 s, 853.8 tps, lat 9.374 ms stddev 7.298
progress: 220.0 s, 849.5 tps, lat 9.415 ms stddev 7.176
progress: 230.0 s, 858.3 tps, lat 9.321 ms stddev 7.050
progress: 240.0 s, 865.5 tps, lat 9.241 ms stddev 7.260
progress: 250.0 s, 848.3 tps, lat 9.429 ms stddev 7.680
progress: 260.0 s, 860.4 tps, lat 9.298 ms stddev 7.171
progress: 270.0 s, 847.2 tps, lat 9.441 ms stddev 7.306
progress: 280.0 s, 856.6 tps, lat 9.339 ms stddev 7.280
progress: 290.0 s, 856.9 tps, lat 9.333 ms stddev 7.162
progress: 300.0 s, 803.2 tps, lat 9.958 ms stddev 8.890
progress: 310.0 s, 798.0 tps, lat 10.027 ms stddev 7.620
progress: 320.0 s, 823.5 tps, lat 9.714 ms stddev 7.078
progress: 330.0 s, 852.1 tps, lat 9.383 ms stddev 7.598
progress: 340.0 s, 860.9 tps, lat 9.289 ms stddev 7.379
progress: 350.0 s, 853.4 tps, lat 9.379 ms stddev 7.359
progress: 360.0 s, 844.8 tps, lat 9.469 ms stddev 7.550
progress: 370.0 s, 852.8 tps, lat 9.371 ms stddev 7.311
progress: 380.0 s, 857.7 tps, lat 9.332 ms stddev 7.441
progress: 390.0 s, 858.1 tps, lat 9.322 ms stddev 7.433
progress: 400.0 s, 862.3 tps, lat 9.275 ms stddev 6.827
progress: 410.0 s, 859.6 tps, lat 9.309 ms stddev 7.290
progress: 420.0 s, 863.7 tps, lat 9.260 ms stddev 7.119
progress: 430.0 s, 850.6 tps, lat 9.404 ms stddev 7.077
progress: 440.0 s, 851.0 tps, lat 9.403 ms stddev 7.065
progress: 450.0 s, 858.3 tps, lat 9.320 ms stddev 7.044
progress: 460.1 s, 720.2 tps, lat 11.020 ms stddev 15.135
progress: 470.1 s, 554.7 tps, lat 14.418 ms stddev 22.974
progress: 480.1 s, 548.2 tps, lat 14.598 ms stddev 21.830
progress: 490.1 s, 568.9 tps, lat 14.054 ms stddev 21.684
progress: 500.1 s, 553.8 tps, lat 14.441 ms stddev 22.431
progress: 510.1 s, 556.3 tps, lat 14.379 ms stddev 22.842
progress: 520.1 s, 561.0 tps, lat 14.257 ms stddev 22.204
progress: 530.1 s, 544.1 tps, lat 14.705 ms stddev 23.293
progress: 540.1 s, 559.5 tps, lat 14.294 ms stddev 22.345
progress: 550.1 s, 534.9 tps, lat 14.981 ms stddev 24.614
progress: 560.1 s, 565.2 tps, lat 14.132 ms stddev 22.250
progress: 570.1 s, 557.2 tps, lat 14.357 ms stddev 22.569
progress: 580.1 s, 556.5 tps, lat 14.372 ms stddev 22.844
progress: 590.1 s, 555.8 tps, lat 14.398 ms stddev 23.088
progress: 600.1 s, 554.2 tps, lat 14.428 ms stddev 22.399
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 464659
latency average = 10.330 ms
latency stddev = 11.654 ms
tps = 774.316644 (including connections establishing)
tps = 774.320332 (excluding connections establishing)
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
postgres@ub20-postgres-6:~$ logout
Student@ub20-postgres-6:~$ sudo systemctl restart postgresql
Student@ub20-postgres-6:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2021-10-23 23:23:22 UTC; 3s ago
    Process: 11757 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 11757 (code=exited, status=0/SUCCESS)

Oct 23 23:23:22 ub20-postgres-6 systemd[1]: Starting PostgreSQL RDBMS...
Oct 23 23:23:22 ub20-postgres-6 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-6:~$ sudo -u postgres -i
postgres@ub20-postgres-6:~$ pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 829.7 tps, lat 9.613 ms stddev 7.100
progress: 20.0 s, 833.6 tps, lat 9.599 ms stddev 7.159
progress: 30.0 s, 833.5 tps, lat 9.593 ms stddev 7.265
progress: 40.0 s, 827.9 tps, lat 9.666 ms stddev 6.942
progress: 50.0 s, 833.2 tps, lat 9.600 ms stddev 7.017
progress: 60.0 s, 834.5 tps, lat 9.585 ms stddev 6.904
progress: 70.0 s, 839.3 tps, lat 9.527 ms stddev 6.885
progress: 80.0 s, 835.1 tps, lat 9.579 ms stddev 6.863
progress: 90.0 s, 827.8 tps, lat 9.665 ms stddev 6.867
progress: 100.0 s, 829.9 tps, lat 9.639 ms stddev 6.698
progress: 110.0 s, 806.3 tps, lat 9.919 ms stddev 6.947
progress: 120.0 s, 826.4 tps, lat 9.678 ms stddev 6.673
progress: 130.0 s, 830.2 tps, lat 9.638 ms stddev 6.763
progress: 140.0 s, 844.1 tps, lat 9.480 ms stddev 6.413
progress: 150.0 s, 836.2 tps, lat 9.563 ms stddev 6.340
progress: 160.0 s, 828.2 tps, lat 9.660 ms stddev 6.510
progress: 170.0 s, 829.4 tps, lat 9.647 ms stddev 6.563
progress: 180.0 s, 830.1 tps, lat 9.638 ms stddev 6.507
progress: 190.0 s, 824.7 tps, lat 9.697 ms stddev 6.710
progress: 200.0 s, 825.1 tps, lat 9.696 ms stddev 6.302
progress: 210.0 s, 836.2 tps, lat 9.566 ms stddev 6.591
progress: 220.0 s, 827.9 tps, lat 9.660 ms stddev 6.435
progress: 230.0 s, 830.0 tps, lat 9.639 ms stddev 6.375
progress: 240.0 s, 838.3 tps, lat 9.539 ms stddev 6.318
progress: 250.0 s, 829.6 tps, lat 9.647 ms stddev 7.309
progress: 260.0 s, 830.1 tps, lat 9.634 ms stddev 7.283
progress: 270.0 s, 800.0 tps, lat 10.001 ms stddev 7.568
progress: 280.0 s, 807.5 tps, lat 9.904 ms stddev 7.506
progress: 290.0 s, 827.6 tps, lat 9.665 ms stddev 7.426
progress: 300.0 s, 822.8 tps, lat 9.719 ms stddev 8.519
progress: 310.0 s, 827.4 tps, lat 9.671 ms stddev 7.346
progress: 320.0 s, 838.8 tps, lat 9.532 ms stddev 7.156
progress: 330.0 s, 837.5 tps, lat 9.554 ms stddev 7.296
progress: 340.0 s, 844.2 tps, lat 9.475 ms stddev 7.152
progress: 350.0 s, 823.7 tps, lat 9.709 ms stddev 7.132
progress: 360.0 s, 834.8 tps, lat 9.585 ms stddev 7.362
progress: 370.0 s, 834.5 tps, lat 9.583 ms stddev 7.071
progress: 380.0 s, 833.5 tps, lat 9.602 ms stddev 7.437
progress: 390.0 s, 834.5 tps, lat 9.582 ms stddev 7.413
progress: 400.0 s, 670.4 tps, lat 11.932 ms stddev 17.026
progress: 410.0 s, 526.6 tps, lat 15.193 ms stddev 24.400
progress: 420.0 s, 518.8 tps, lat 15.420 ms stddev 24.017
progress: 430.0 s, 523.3 tps, lat 15.284 ms stddev 24.469
progress: 440.0 s, 517.1 tps, lat 15.474 ms stddev 25.150
progress: 450.0 s, 528.4 tps, lat 15.135 ms stddev 24.281
progress: 460.0 s, 535.0 tps, lat 14.948 ms stddev 23.692
progress: 470.0 s, 535.0 tps, lat 14.949 ms stddev 24.002
progress: 480.0 s, 540.4 tps, lat 14.810 ms stddev 23.463
progress: 490.0 s, 530.7 tps, lat 15.070 ms stddev 24.282
progress: 500.0 s, 530.7 tps, lat 15.068 ms stddev 23.619
progress: 510.0 s, 522.4 tps, lat 15.320 ms stddev 24.627
progress: 520.0 s, 539.7 tps, lat 14.827 ms stddev 23.382
^C
postgres@ub20-postgres-6:~$

```
#### 9. построить график по получившимся значениям
##### Результат:

Ну тут и так ясно что в районе 400+- секунды идет просадка нет смысла строить график

#### 10. так чтобы получить максимально ровное значение tps (отредактировано) 
##### Результат:

```
postgres@ub20-postgres-6:~$ vim /etc/postgresql/13/main/postgresql.conf
                                        # requires track_counts to also be on.
log_autovacuum_min_duration = 0         # -1 disables, 0 logs all actions and
                                        # their durations, > 0 logs only
                                        # actions running at least this number
                                        # of milliseconds.
autovacuum_max_workers = 10             # max number of autovacuum subprocesses
                                        # (change requires restart)
autovacuum_naptime = 10         # time between autovacuum runs
autovacuum_vacuum_threshold = 25        # min number of row updates before
                                        # vacuum
autovacuum_vacuum_insert_threshold = 25 # min number of row inserts
                                        # before vacuum; -1 disables insert
                                        # vacuums
autovacuum_analyze_threshold = 25       # min number of row updates before
                                        # analyze
autovacuum_vacuum_scale_factor = 0.01   # fraction of table size before vacuum
autovacuum_vacuum_insert_scale_factor = 0.08    # fraction of inserts over table
                                        # size before insert vacuum
autovacuum_analyze_scale_factor = 0.1   # fraction of table size before analyze
#autovacuum_freeze_max_age = 200000000  # maximum XID age before forced vacuum
                                        # (change requires restart)
#autovacuum_multixact_freeze_max_age = 400000000        # maximum multixact age
                                        # before forced vacuum
                                        # (change requires restart)
autovacuum_vacuum_cost_delay = 10ms     # default vacuum cost delay for
                                        # autovacuum, in milliseconds;
                                        # -1 means use vacuum_cost_delay
autovacuum_vacuum_cost_limit = 2000     # default vacuum cost limit for
                                        # autovacuum, -1 means use

```
перепробовал кучу комбинаций, но явно нужно что-то более рациональное чем подбор методом научного тыка

Понял несколько тенденций но это очень не обоснованно на мой взгляд, так что нет смысла сюда писать.
uptime 23:54:42 up  9:56,  2 users,  load average: 0.00, 0.20, 3.51

