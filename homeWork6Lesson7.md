# Домашнее задание
## Работа с журналами

### Цель:
- уметь работать с журналами и контрольными точками
- уметь настраивать параметры журналов

### Задание и ход выполнения по этапам: 

#### 1. Настройте выполнение контрольной точки раз в 30 секунд.
##### Результат:
```
postgres=# ALTER SYSTEM SET checkpoint_timeout=30;
ALTER SYSTEM

postgres=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show checkpoint_timeout;
 checkpoint_timeout
--------------------
 30s
(1 row)

```

#### 2. 10 минут c помощью утилиты pgbench подавайте нагрузку.
##### Результат:
Готово.
```
postgres=# SELECT pg_current_wal_insert_lsn(),pg_current_wal_lsn();
 pg_current_wal_insert_lsn | pg_current_wal_lsn
---------------------------+--------------------
 0/2243CD8                 | 0/2243CD8
(1 row)

postgres=# \! pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 729.8 tps, lat 10.918 ms stddev 7.735
progress: 20.0 s, 723.9 tps, lat 11.051 ms stddev 7.732
progress: 30.0 s, 730.5 tps, lat 10.953 ms stddev 7.989
progress: 40.0 s, 705.8 tps, lat 11.329 ms stddev 7.683
progress: 50.0 s, 707.3 tps, lat 11.313 ms stddev 7.773
progress: 60.0 s, 717.4 tps, lat 11.150 ms stddev 7.998
progress: 70.0 s, 703.7 tps, lat 11.367 ms stddev 8.046
progress: 80.0 s, 748.9 tps, lat 10.681 ms stddev 7.152
progress: 90.0 s, 770.8 tps, lat 10.377 ms stddev 7.096
progress: 100.0 s, 743.8 tps, lat 10.756 ms stddev 7.114
progress: 110.0 s, 777.9 tps, lat 10.284 ms stddev 6.670
progress: 120.0 s, 779.6 tps, lat 10.262 ms stddev 6.757
progress: 130.0 s, 760.6 tps, lat 10.516 ms stddev 6.903
progress: 140.0 s, 772.5 tps, lat 10.352 ms stddev 6.602
progress: 150.0 s, 796.5 tps, lat 10.048 ms stddev 6.788
progress: 160.0 s, 792.4 tps, lat 10.094 ms stddev 6.478
progress: 170.0 s, 800.3 tps, lat 9.996 ms stddev 6.300
progress: 180.0 s, 797.4 tps, lat 10.031 ms stddev 6.576
progress: 190.0 s, 810.1 tps, lat 9.875 ms stddev 6.308
progress: 200.0 s, 800.0 tps, lat 9.998 ms stddev 6.252
progress: 210.0 s, 806.1 tps, lat 9.922 ms stddev 6.821
progress: 220.0 s, 801.3 tps, lat 9.985 ms stddev 6.782
progress: 230.0 s, 807.4 tps, lat 9.905 ms stddev 7.264
progress: 240.0 s, 799.9 tps, lat 10.000 ms stddev 7.560
progress: 250.0 s, 798.7 tps, lat 10.017 ms stddev 7.398
progress: 260.0 s, 817.0 tps, lat 9.791 ms stddev 7.668
progress: 270.0 s, 802.4 tps, lat 9.970 ms stddev 7.502
progress: 280.0 s, 809.6 tps, lat 9.877 ms stddev 7.395
progress: 290.0 s, 813.0 tps, lat 9.840 ms stddev 7.393
progress: 300.0 s, 814.5 tps, lat 9.822 ms stddev 7.740
progress: 310.0 s, 796.4 tps, lat 10.043 ms stddev 7.809
progress: 320.0 s, 822.0 tps, lat 9.730 ms stddev 7.465
progress: 330.0 s, 811.4 tps, lat 9.863 ms stddev 7.190
progress: 340.0 s, 796.0 tps, lat 10.050 ms stddev 7.582
progress: 350.0 s, 802.2 tps, lat 9.966 ms stddev 7.283
progress: 360.0 s, 803.9 tps, lat 9.954 ms stddev 7.513
progress: 370.0 s, 765.3 tps, lat 10.450 ms stddev 7.963
progress: 380.0 s, 769.0 tps, lat 10.404 ms stddev 7.733
progress: 390.0 s, 802.4 tps, lat 9.963 ms stddev 7.345
progress: 400.0 s, 810.2 tps, lat 9.876 ms stddev 7.316
progress: 410.0 s, 789.7 tps, lat 10.132 ms stddev 7.342
progress: 420.0 s, 828.9 tps, lat 9.651 ms stddev 7.170
progress: 430.0 s, 829.9 tps, lat 9.639 ms stddev 7.029
progress: 440.0 s, 815.2 tps, lat 9.811 ms stddev 7.228
progress: 450.0 s, 814.0 tps, lat 9.755 ms stddev 7.298
progress: 460.0 s, 535.0 tps, lat 14.942 ms stddev 22.632
progress: 470.0 s, 526.2 tps, lat 15.189 ms stddev 23.071
progress: 480.0 s, 535.4 tps, lat 14.942 ms stddev 23.048
progress: 490.0 s, 533.0 tps, lat 15.003 ms stddev 23.447
progress: 500.0 s, 536.1 tps, lat 14.929 ms stddev 22.305
progress: 510.0 s, 539.6 tps, lat 14.843 ms stddev 22.702
progress: 520.0 s, 532.0 tps, lat 15.015 ms stddev 22.141
progress: 530.0 s, 528.3 tps, lat 15.146 ms stddev 22.970
progress: 540.0 s, 528.6 tps, lat 15.155 ms stddev 23.067
progress: 550.0 s, 527.1 tps, lat 15.146 ms stddev 23.900
progress: 560.0 s, 513.7 tps, lat 15.557 ms stddev 23.557
progress: 570.0 s, 510.9 tps, lat 15.670 ms stddev 23.929
progress: 580.0 s, 520.2 tps, lat 15.371 ms stddev 24.198
progress: 590.0 s, 519.7 tps, lat 15.393 ms stddev 23.860
progress: 600.0 s, 518.3 tps, lat 15.437 ms stddev 24.736
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 432022
latency average = 11.109 ms
latency stddev = 12.107 ms
tps = 719.987069 (including connections establishing)
tps = 719.991453 (excluding connections establishing)

postgres=# SELECT pg_current_wal_insert_lsn(),pg_current_wal_lsn();
 pg_current_wal_insert_lsn | pg_current_wal_lsn
---------------------------+--------------------
 0/2070D2C8                | 0/2070D2C8
(1 row)

```

#### 3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
##### Результат:
```
postgres=# SELECT '0/2243398'::pg_lsn - '0/2070D2C8'::pg_lsn;
  ?column?
------------
 -508337968
(1 row)
```
В wal файлах это заняло бы около 484МБ, но так как после завершения контрольной точки журналы записанные до её начала не нужны, то файлы с ними не хранятся (мы ничего никуда не реплицируем, задержек нет все лишнее удаляется)
остается только последние 4 файла (размером в 16 Мб):
```
postgres=# \! ls -la ./13/main/pg_wal
total 65548
drwx------  3 postgres postgres     4096 Oct 26 19:27 .
drwx------ 19 postgres postgres     4096 Oct 26 19:07 ..
-rw-------  1 postgres postgres 16777216 Oct 26 19:29 000000010000000000000020
-rw-------  1 postgres postgres 16777216 Oct 26 19:26 000000010000000000000021
-rw-------  1 postgres postgres 16777216 Oct 26 19:27 000000010000000000000022
-rw-------  1 postgres postgres 16777216 Oct 26 19:27 000000010000000000000023
drwx------  2 postgres postgres     4096 Oct 26 18:59 archive_status

```
При этом перед запуском у pg_bench у нас LSN  указывал на файл `000000010000000000000002`
```
postgres=# SELECT file_name, upper(to_hex(file_offset)) file_offset
postgres-# FROM pg_walfile_name_offset('0/2243398');
        file_name         | file_offset
--------------------------+-------------
 000000010000000000000002 | 243398
(1 row)

```
А сразу после на файл `000000010000000000000020`
```
postgres=# SELECT file_name, upper(to_hex(file_offset)) file_offset
postgres-# FROM pg_walfile_name_offset('0/2070D2C8');
        file_name         | file_offset
--------------------------+-------------
 000000010000000000000020 | 70D2C8
(1 row)

```
Таким образом за время исполнения теста мы сгенегировали приблизительно 20 hex-2 hex=1E hex->30 Dec файла журнала по 16 Мб каждый и того 480Mb (но тут надо учетывать хвосты в конечных файлах)
итого: "наших контрольных точек 20" логов 484 Мб делим среднее равно около = 24Мб на 1 Контрольную точку

#### 4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
##### Результат:

```
postgres=# select * from pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 127
checkpoints_req       | 0
checkpoint_write_time | 328092
checkpoint_sync_time  | 162
buffers_checkpoint    | 44574
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 4600
buffers_backend_fsync | 0
buffers_alloc         | 5219
stats_reset           | 2021-10-26 18:59:36.254308+00
```
Команду CHECKPOINT  мы не использовали, 
Отдельно включил запись точек в лог (здесь видно что когда система не под нагрузкой то контрольные точки могут не создаваться по расписанию, но когда нагрузка идет то они создаются регулярно)
Существует еще ситуация когда достигается max_wal_size тогда сброс контрольной точки произойдет не по таймауту, а "по требованию" 

```
postgres=# \! tail -n 200 /var/log/postgresql/postgresql-13-main.log
...
2021-10-26 22:15:02.137 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:15:17.115 UTC [3417] LOG:  checkpoint complete: wrote 1962 buffers (12.0%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.955 s, sync=0.006 s, total=14.979 s; sync files=14, longest=0.004 s, average=0.001 s; distance=23760 kB, estimate=23760 kB
2021-10-26 22:15:32.130 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:15:47.015 UTC [3417] LOG:  checkpoint complete: wrote 1845 buffers (11.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.866 s, sync=0.004 s, total=14.885 s; sync files=7, longest=0.004 s, average=0.001 s; distance=20122 kB, estimate=23396 kB
2021-10-26 22:20:02.229 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:20:17.033 UTC [3417] LOG:  checkpoint complete: wrote 1951 buffers (11.9%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.758 s, sync=0.008 s, total=14.805 s; sync files=18, longest=0.004 s, average=0.001 s; distance=22863 kB, estimate=23343 kB
2021-10-26 22:20:32.045 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:20:47.025 UTC [3417] LOG:  checkpoint complete: wrote 1941 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.953 s, sync=0.008 s, total=14.981 s; sync files=6, longest=0.005 s, average=0.002 s; distance=23625 kB, estimate=23625 kB
2021-10-26 22:21:02.037 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:21:17.030 UTC [3417] LOG:  checkpoint complete: wrote 2073 buffers (12.7%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.956 s, sync=0.009 s, total=14.994 s; sync files=14, longest=0.005 s, average=0.001 s; distance=24088 kB, estimate=24088 kB
2021-10-26 22:21:32.045 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:21:47.021 UTC [3417] LOG:  checkpoint complete: wrote 1926 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.954 s, sync=0.006 s, total=14.977 s; sync files=7, longest=0.003 s, average=0.001 s; distance=24255 kB, estimate=24255 kB
2021-10-26 22:22:02.033 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:22:17.022 UTC [3417] LOG:  checkpoint complete: wrote 2056 buffers (12.5%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.955 s, sync=0.006 s, total=14.990 s; sync files=14, longest=0.004 s, average=0.001 s; distance=24542 kB, estimate=24542 kB
2021-10-26 22:22:32.037 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:22:47.034 UTC [3417] LOG:  checkpoint complete: wrote 1933 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.958 s, sync=0.007 s, total=14.998 s; sync files=8, longest=0.003 s, average=0.001 s; distance=24227 kB, estimate=24510 kB
2021-10-26 22:23:02.049 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:23:17.042 UTC [3417] LOG:  checkpoint complete: wrote 2029 buffers (12.4%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.955 s, sync=0.006 s, total=14.994 s; sync files=14, longest=0.003 s, average=0.001 s; distance=24045 kB, estimate=24464 kB
2021-10-26 22:23:32.057 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:23:47.036 UTC [3417] LOG:  checkpoint complete: wrote 1937 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.954 s, sync=0.009 s, total=14.980 s; sync files=8, longest=0.006 s, average=0.002 s; distance=24328 kB, estimate=24450 kB
2021-10-26 22:24:02.049 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:24:17.044 UTC [3417] LOG:  checkpoint complete: wrote 2038 buffers (12.4%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.956 s, sync=0.006 s, total=14.996 s; sync files=13, longest=0.003 s, average=0.001 s; distance=24586 kB, estimate=24586 kB
2021-10-26 22:24:32.057 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:24:47.055 UTC [3417] LOG:  checkpoint complete: wrote 1935 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.960 s, sync=0.005 s, total=14.999 s; sync files=9, longest=0.003 s, average=0.001 s; distance=24597 kB, estimate=24597 kB
2021-10-26 22:25:02.069 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:25:17.059 UTC [3417] LOG:  checkpoint complete: wrote 2049 buffers (12.5%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.956 s, sync=0.005 s, total=14.991 s; sync files=13, longest=0.003 s, average=0.001 s; distance=25117 kB, estimate=25117 kB
2021-10-26 22:25:32.073 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:25:47.064 UTC [3417] LOG:  checkpoint complete: wrote 1932 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.958 s, sync=0.005 s, total=14.992 s; sync files=8, longest=0.003 s, average=0.001 s; distance=24878 kB, estimate=25093 kB
2021-10-26 22:26:02.077 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:26:17.069 UTC [3417] LOG:  checkpoint complete: wrote 2053 buffers (12.5%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.957 s, sync=0.006 s, total=14.993 s; sync files=13, longest=0.003 s, average=0.001 s; distance=25024 kB, estimate=25086 kB
2021-10-26 22:26:32.081 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:26:47.077 UTC [3417] LOG:  checkpoint complete: wrote 1933 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.962 s, sync=0.006 s, total=14.997 s; sync files=8, longest=0.004 s, average=0.001 s; distance=24613 kB, estimate=25039 kB
2021-10-26 22:27:02.089 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:27:17.071 UTC [3417] LOG:  checkpoint complete: wrote 2044 buffers (12.5%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.952 s, sync=0.010 s, total=14.983 s; sync files=11, longest=0.004 s, average=0.001 s; distance=24577 kB, estimate=24993 kB
2021-10-26 22:27:32.085 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:27:47.026 UTC [3417] LOG:  checkpoint complete: wrote 1929 buffers (11.8%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.907 s, sync=0.007 s, total=14.942 s; sync files=8, longest=0.004 s, average=0.001 s; distance=24608 kB, estimate=24954 kB
2021-10-26 22:28:02.041 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:28:17.032 UTC [3417] LOG:  checkpoint complete: wrote 2413 buffers (14.7%); 0 WAL file(s) added, 0 removed, 2 recycled; write=14.951 s, sync=0.009 s, total=14.992 s; sync files=13, longest=0.006 s, average=0.001 s; distance=20578 kB, estimate=24517 kB
2021-10-26 22:28:32.045 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:28:47.026 UTC [3417] LOG:  checkpoint complete: wrote 1871 buffers (11.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.959 s, sync=0.006 s, total=14.982 s; sync files=8, longest=0.004 s, average=0.001 s; distance=20606 kB, estimate=24126 kB
2021-10-26 22:29:02.041 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:29:17.029 UTC [3417] LOG:  checkpoint complete: wrote 1880 buffers (11.5%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.951 s, sync=0.009 s, total=14.989 s; sync files=11, longest=0.004 s, average=0.001 s; distance=20619 kB, estimate=23775 kB
2021-10-26 22:29:32.041 UTC [3417] LOG:  checkpoint starting: time
2021-10-26 22:29:47.074 UTC [3417] LOG:  checkpoint complete: wrote 1868 buffers (11.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=14.993 s, sync=0.007 s, total=15.034 s; sync files=8, longest=0.004 s, average=0.001 s; distance=20703 kB, estimate=23468 kB

```

#### 5. Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
##### Результат:

Проверяем что у нас был настроен синхронный режим:
```
postgres=# show synchronous_commit ;
 synchronous_commit
--------------------
 on
(1 row)
```
Значит результаты для синхронного режима у нас есть
осталось переключиться в асинхронный:
```
postgres=# ALTER SYSTEM SET synchronous_commit = off;
ALTER SYSTEM
postgres=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show synchronous_commit ;
 synchronous_commit
--------------------
 off
(1 row)

```

Запустим тест
```
postgres=# \! pgbench -c8 -P 10 -T 600 -U postgres postgres
starting vacuum...end.
progress: 10.0 s, 1904.7 tps, lat 4.187 ms stddev 1.300
progress: 20.0 s, 1940.4 tps, lat 4.122 ms stddev 1.228
progress: 30.0 s, 1919.2 tps, lat 4.168 ms stddev 1.258
progress: 40.0 s, 1897.2 tps, lat 4.216 ms stddev 1.361
progress: 50.0 s, 1925.9 tps, lat 4.153 ms stddev 1.324
progress: 60.0 s, 1894.1 tps, lat 4.223 ms stddev 1.318
progress: 70.0 s, 1899.7 tps, lat 4.210 ms stddev 1.280
progress: 80.0 s, 1895.8 tps, lat 4.219 ms stddev 1.290
progress: 90.0 s, 1916.1 tps, lat 4.174 ms stddev 1.274
progress: 100.0 s, 1903.3 tps, lat 4.202 ms stddev 1.325
progress: 110.0 s, 1924.9 tps, lat 4.155 ms stddev 1.280
progress: 120.0 s, 1909.7 tps, lat 4.188 ms stddev 1.668
progress: 130.0 s, 941.9 tps, lat 8.492 ms stddev 23.246
progress: 140.0 s, 947.7 tps, lat 8.440 ms stddev 23.244
progress: 150.0 s, 931.9 tps, lat 8.571 ms stddev 23.554
progress: 160.0 s, 931.2 tps, lat 8.548 ms stddev 23.544
progress: 170.0 s, 942.7 tps, lat 8.499 ms stddev 23.510
progress: 180.0 s, 945.2 tps, lat 8.478 ms stddev 23.428
progress: 190.0 s, 952.6 tps, lat 8.383 ms stddev 23.108
progress: 200.0 s, 939.1 tps, lat 8.519 ms stddev 23.326
progress: 210.0 s, 922.2 tps, lat 8.658 ms stddev 23.524
progress: 220.0 s, 907.4 tps, lat 8.817 ms stddev 23.859
progress: 230.0 s, 932.4 tps, lat 8.579 ms stddev 23.494
progress: 240.0 s, 931.7 tps, lat 8.600 ms stddev 23.464
progress: 250.0 s, 957.4 tps, lat 8.327 ms stddev 23.008
progress: 260.0 s, 940.8 tps, lat 8.504 ms stddev 23.475
progress: 270.0 s, 919.1 tps, lat 8.704 ms stddev 23.742
progress: 280.0 s, 930.7 tps, lat 8.608 ms stddev 23.624
progress: 290.0 s, 921.9 tps, lat 8.677 ms stddev 23.631
progress: 300.0 s, 927.2 tps, lat 8.613 ms stddev 23.513
progress: 310.0 s, 955.6 tps, lat 8.358 ms stddev 23.097
progress: 320.0 s, 951.0 tps, lat 8.411 ms stddev 23.157
progress: 330.0 s, 929.8 tps, lat 8.589 ms stddev 23.454
progress: 340.0 s, 931.9 tps, lat 8.599 ms stddev 23.543
progress: 350.0 s, 947.0 tps, lat 8.446 ms stddev 23.306
progress: 360.0 s, 934.3 tps, lat 8.532 ms stddev 23.365
progress: 370.0 s, 939.6 tps, lat 8.557 ms stddev 23.423
progress: 380.0 s, 928.9 tps, lat 8.596 ms stddev 23.478
progress: 390.0 s, 923.1 tps, lat 8.680 ms stddev 23.825
progress: 400.0 s, 909.8 tps, lat 8.763 ms stddev 23.727
progress: 410.0 s, 936.3 tps, lat 8.558 ms stddev 23.411
progress: 420.0 s, 956.0 tps, lat 8.381 ms stddev 23.183
progress: 430.0 s, 956.0 tps, lat 8.368 ms stddev 23.177
progress: 440.0 s, 933.8 tps, lat 8.536 ms stddev 23.453
progress: 450.0 s, 939.2 tps, lat 8.518 ms stddev 23.391
progress: 460.0 s, 931.3 tps, lat 8.618 ms stddev 23.899
progress: 470.0 s, 964.4 tps, lat 8.280 ms stddev 23.051
progress: 480.0 s, 950.2 tps, lat 8.420 ms stddev 23.249
progress: 490.0 s, 958.9 tps, lat 8.356 ms stddev 23.097
progress: 500.0 s, 959.7 tps, lat 8.321 ms stddev 23.026
progress: 510.0 s, 957.6 tps, lat 8.368 ms stddev 23.202
progress: 520.0 s, 940.5 tps, lat 8.490 ms stddev 23.423
progress: 530.0 s, 954.1 tps, lat 8.356 ms stddev 23.101
progress: 540.0 s, 945.2 tps, lat 8.493 ms stddev 23.325
progress: 550.0 s, 946.6 tps, lat 8.435 ms stddev 23.265
progress: 560.0 s, 938.3 tps, lat 8.539 ms stddev 23.398
progress: 570.0 s, 934.3 tps, lat 8.562 ms stddev 23.386
progress: 580.0 s, 947.5 tps, lat 8.442 ms stddev 23.361
progress: 590.0 s, 939.0 tps, lat 8.519 ms stddev 23.499
progress: 600.0 s, 939.9 tps, lat 8.497 ms stddev 23.275
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 680347
latency average = 7.053 ms
latency stddev = 19.171 ms
tps = 1133.876207 (including connections establishing)
tps = 1133.882734 (excluding connections establishing)

```
Сравним результаты:

      для синхронного режима было так:
```

tps = 719.987069 (including connections establishing)
tps = 719.991453 (excluding connections establishing)
```
итого разница в приблизительно в полтора раза скорость выше в асинхронном режиме:
1133/719=1.57 

#### 6. Создайте новый кластер с включенной контрольной суммой страниц. 
##### Результат:
Создаем второй кластер с включенной проверкой CRC
```
postgres@ub20-postgres-7:~$ mkdir nck
postgres@ub20-postgres-7:~/nck$ /usr/lib/postgresql/13/bin/initdb -D /var/lib/postgresql/nck --data-checksums
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "C.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are enabled.

fixing permissions on existing directory /var/lib/postgresql/nck ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Etc/UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/lib/postgresql/13/bin/pg_ctl -D /var/lib/postgresql/nck -l logfile start

```
правим конфиг (меняем порт на 5433)
запускаем:
```
postgres@ub20-postgres-7:~$ /usr/lib/postgresql/13/bin/pg_ctl start -D /var/lib/postgresql/nck -l /var/lib/postgresql/nck/lllog.log
waiting for server to start.... done
server started
postgres@ub20-postgres-7:~$ psql -p 5433
psql (13.4 (Ubuntu 13.4-4.pgdg20.04+1))
Type "help" for help.

postgres=#
postgres=# SHOW data_checksums;
 data_checksums
----------------
 on
(1 row)


```

#### 6-1. Создайте таблицу. Вставьте несколько значений. 
##### Результат:

```
postgres=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
CREATE TABLE
INSERT 0 1
INSERT 0 1

```

#### 6-2. Выключите кластер. Измените пару байт в таблице. 
##### Результат:

```
postgres=# \q
postgres@ub20-postgres-7:~$ /usr/lib/postgresql/13/bin/pg_ctl stop -D /var/lib/postgresql/nck -l /var/lib/postgresql/nck/lllog.log
waiting for server to shut down.... done
server stopped
postgres@ub20-postgres-7:~$ vim /var/lib/postgresql/nck/base/13414/16386
...

```
готово.

#### 6-3. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?
##### Результат:

```
postgres@ub20-postgres-7:~$ /usr/lib/postgresql/13/bin/pg_ctl start -D /var/lib/postgresql/nck -l /var/lib/postgresql/nck/lllog.log
waiting for server to start.... done
server started
postgres@ub20-postgres-7:~$ psql -p 5433
psql (13.4 (Ubuntu 13.4-4.pgdg20.04+1))
Type "help" for help.

postgres=# select * from persons;
WARNING:  page verification failed, calculated checksum 6605 but expected 39851
ERROR:  invalid page in block 0 of relation base/13414/16386

```

Мы получили сообщение о том что контрольная сумма файла не совпадает с ожидаемой. (Самое время задуматься о целоствости данных :) )
Можно проигнорировать ошибку так:
```
postgres=# SET ignore_checksum_failure = on;
SET
postgres=# show ignore_checksum_failure;
 ignore_checksum_failure
-------------------------
 on
(1 row)

```
Но если данные реально испорчены то получится так:
```
postgres=# select * from persons;
WARNING:  page verification failed, calculated checksum 6605 but expected 39851
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Failed.

```
Можно идти восстанавливаться из бакапа...
или ковырять walы в надежде что-то восстановить
