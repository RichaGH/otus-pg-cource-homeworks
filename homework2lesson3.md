# Домашнее задание
## Установка и настройка PostgreSQL
### Цель:
1. создавать дополнительный диск для уже существующей виртуальной машины,
2. размечать его и делать на нем файловую систему
3. переносить содержимое базы данных PostgreSQL на дополнительный диск
4. переносить содержимое БД PostgreSQL между виртуальными машинами

### Задание и ход выполнения по этапам: 
#### 1. создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a
##### Результат:

*Created*

NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS  
ub20-postgres-2  us-central1-c  e2-medium                  10.128.0.4   35.193.253.238  RUNNING

#### 2. поставьте на нее PostgreSQL через sudo apt
##### Результат:
```
sudo apt update && sudo apt upgrade -y -q  
sudo shutdown -r now  
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -  
sudo apt-get update  
sudo apt-get -y install postgresql  
sudo systemctl status postgresql  
```

#### 3 проверьте что кластер запущен через sudo -u postgres pg_lsclusters
##### Результат:
```
Student@ub20-postgres-2:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```
#### 4 зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым postgres=# create table test(c1 text); postgres=# insert into test values('1'); \q
##### Результат:
```
Student@ub20-postgres-2:~$ sudo -u postgres -i psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# create database test1;
CREATE DATABASE
postgres=# \c test1
You are now connected to database "test1" as user "postgres".

test1=# create table test(c1 text);
CREATE TABLE
test1=# insert into test values('1');
INSERT 0 1
test1=# \q
```
#### 5 остановите postgres например через sudo -u postgres pg_ctlcluster 13 main stop
##### Результат: Но:
тут нас предупредят, что для управления кластером следует использовать утилиту systemctl и если мы так сделаем то systemd unit будет помечен как failed

При этом статус службы отался Active: active и я воспользовался systemctl для остановки postgres
```
Student@ub20-postgres-2:~$ sudo -u postgres pg_ctlcluster 14 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@14-main
Student@ub20-postgres-2:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2021-10-12 16:40:40 UTC; 15min ago
   Main PID: 3439 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 4705)
     Memory: 0B
     CGroup: /system.slice/postgresql.service

Oct 12 16:40:40 ub20-postgres-2 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 16:40:40 ub20-postgres-2 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-2:~$ sudo systemctl stop postgresql
Student@ub20-postgres-2:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2021-10-12 16:56:07 UTC; 2s ago
   Main PID: 3439 (code=exited, status=0/SUCCESS)

Oct 12 16:40:40 ub20-postgres-2 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 16:40:40 ub20-postgres-2 systemd[1]: Finished PostgreSQL RDBMS.
Oct 12 16:56:07 ub20-postgres-2 systemd[1]: postgresql.service: Succeeded.
Oct 12 16:56:07 ub20-postgres-2 systemd[1]: Stopped PostgreSQL RDBMS.
```

#### 6 создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
##### Результат:
в гугловй SDK (CLI) запускаем команду по созданию диска и получаем результат
```
Cloud SDK>gcloud beta compute disks create pg-data-d1 --project=postgres2021-19810421 --type=pd-standard --size=10GB --zone=us-central1-c
WARNING: You have selected a disk size of under [200GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created 
NAME        ZONE           SIZE_GB  TYPE         STATUS
pg-data-d1  us-central1-c  10       pd-standard  READY

New disks are unformatted. You must format and mount a disk before it
can be used. You can find instructions on how to do this at:

https://cloud.google.com/compute/docs/disks/add-persistent-disk#formatting
```
#### 7 добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
##### Результат: 
Добавил диск через редактирование ВМ в web-интерфейсе

#### 8 проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
##### Результат:
Выполнил в КС команды просмотр блочных устройств (slblk) нашел как называется диск, определился с форматом диска, создал раздел , проверил выравнивание, создал ФС, создал каталог под точку монтирования и примонтировал туда диск.
```
 lsblk
 sudo parted -s -a optimal -- /dev/sdb mklabel gpt
 sudo parted -s -a optimal -- /dev/sdb mkpart primary 0% 100%
 sudo parted -s -- /dev/sdb align-check optimal 1
 sudo mkfs.ext4 -L DataPGC /dev/sdb1
 sudo mkdir -p /mnt/data
 sudo mount -o defaults /dev/sdb1 /mnt/data
 ```
проверил что диск подключился в `/mnt/data`
```
Student@ub20-postgres-2:~$ sudo df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/root      ext4      9.6G  2.0G  7.6G  21% /
devtmpfs       devtmpfs  2.0G     0  2.0G   0% /dev
tmpfs          tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs          tmpfs     393M  932K  392M   1% /run
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/loop0     squashfs   56M   56M     0 100% /snap/core18/2128
/dev/loop2     squashfs  249M  249M     0 100% /snap/google-cloud-sdk/199
/dev/loop1     squashfs   62M   62M     0 100% /snap/core20/1081
/dev/loop3     squashfs   68M   68M     0 100% /snap/lxd/21545
/dev/loop4     squashfs   33M   33M     0 100% /snap/snapd/13170
/dev/sda15     vfat      105M  5.2M  100M   5% /boot/efi
tmpfs          tmpfs     393M     0  393M   0% /run/user/1002
/dev/sdb1      ext4      9.8G   37M  9.3G   1% /mnt/data
```

#### 9 сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
##### Результат:

`sudo chown -R postgres:postgres /mnt/data/`
готово

#### 10 перенесите содержимое /var/lib/postgres/13 в /mnt/data - mv /var/lib/postgresql/13 /mnt/data
##### Результат:
```
Student@ub20-postgres-2:~$ sudo mv /var/lib/postgresql/14 /mnt/data
Student@ub20-postgres-2:~$ sudo ls -la /mnt/data/
total 28
drwxr-xr-x 4 postgres postgres  4096 Oct 12 17:44 .
drwxr-xr-x 3 root     root      4096 Oct 12 17:30 ..
drwxr-xr-x 3 postgres postgres  4096 Oct 12 16:40 14
drwx------ 2 postgres postgres 16384 Oct 12 17:26 lost+found
```

#### 11 попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 13 main start
##### Результат: готово - неработает:
```
Student@ub20-postgres-2:~$ sudo -u postgres pg_ctlcluster 14 main start
Error: /var/lib/postgresql/14/main is not accessible or does not exist
Student@ub20-postgres-2:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2021-10-12 17:48:59 UTC; 1min 35s ago
    Process: 5010 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 5010 (code=exited, status=0/SUCCESS)

Oct 12 17:47:53 ub20-postgres-2 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 17:47:53 ub20-postgres-2 systemd[1]: Finished PostgreSQL RDBMS.
Oct 12 17:48:59 ub20-postgres-2 systemd[1]: postgresql.service: Succeeded.
Oct 12 17:48:59 ub20-postgres-2 systemd[1]: Stopped PostgreSQL RDBMS.
```
#### 12 напишите получилось или нет и почему
##### Результат:
Пишет что не доступен дата каталог. (ну его нет мы же переместили данные, а конфиг еще не исправили)

#### 13 задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
##### Результат:
Провел поиск по каталогу с конфигами файлов, содержащих пути к перемещенному каталогу:
```
Student@ub20-postgres-2:~$ sudo -u postgres -i grep '/var/lib/postgresql/14' /etc/postgresql/14/main/*
grep: /etc/postgresql/14/main/conf.d: Is a directory
/etc/postgresql/14/main/postgresql.conf:data_directory = '/var/lib/postgresql/14/main'         # use data in another directory
```
обнаружил, что есть строка в конфиге /etc/postgresql/14/main/postgresql.conf, содержащая параметр в котором явно указан путь к перемещенному каталогу. В других файлах в каталоге данный путь не упоминается

Запустил редактор текстовых файлов с правами пользователя postgres
`sudo -u postgres -i vim /etc/postgresql/14/main/postgresql.conf`
и изменил
`data_directory = '/var/lib/postgresql/14/main'`
на 
`data_directory = '/mnt/data/14/main'`

#### 14 попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 13 main start
##### Результат:
```
Student@ub20-postgres-2:~$ sudo -u postgres pg_ctlcluster 14 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
```
#### 15 напишите получилось или нет и почему
##### Результат:
Получилось, так как параметр определяет путь к файлам БД и му указали где искать эти файлы.

Однако процесс был запущен не в сервисном режиме и мы теперь не может получить его статус через systemctl  и должны грепать в ps
чтобы понять работает он или нет

так выглядит статус рекомендованного нам предыдущей командой сервиса:
```
Student@ub20-postgres-2:~$ sudo systemctl status postgresql@14-main
? postgresql@14-main.service - PostgreSQL Cluster 14-main
     Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)
     Active: failed (Result: protocol) since Tue 2021-10-12 17:47:53 UTC; 44min ago
    Process: 5009 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 14-main start (code=exited, status=1/FAILURE)

Oct 12 17:47:53 ub20-postgres-2 systemd[1]: Starting PostgreSQL Cluster 14-main...
Oct 12 17:47:53 ub20-postgres-2 postgresql@14-main[5009]: Error: /var/lib/postgresql/14/main is not accessible or does not exist
Oct 12 17:47:53 ub20-postgres-2 systemd[1]: postgresql@14-main.service: Can't open PID file /run/postgresql/14-main.pid (yet?) after start: Operation not permitted
Oct 12 17:47:53 ub20-postgres-2 systemd[1]: postgresql@14-main.service: Failed with result 'protocol'.
Oct 12 17:47:53 ub20-postgres-2 systemd[1]: Failed to start PostgreSQL Cluster 14-main.
```
Так выглядят "грепнутые" процессы здесь видно что процесс запущен и работает:
```
Student@ub20-postgres-2:~$ ps -elf |grep postg
0 S postgres    5222       1  0  80   0 - 54416 -      18:21 ?        00:00:00 /usr/lib/postgresql/14/bin/postgres -D /mnt/data/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
1 S postgres    5224    5222  0  80   0 - 54451 -      18:21 ?        00:00:00 postgres: 14/main: checkpointer
1 S postgres    5225    5222  0  80   0 - 54416 -      18:21 ?        00:00:00 postgres: 14/main: background writer
1 S postgres    5226    5222  0  80   0 - 54416 -      18:21 ?        00:00:00 postgres: 14/main: walwriter
1 S postgres    5227    5222  0  80   0 - 54584 -      18:21 ?        00:00:00 postgres: 14/main: autovacuum launcher
1 S postgres    5228    5222  0  80   0 - 18099 -      18:21 ?        00:00:00 postgres: 14/main: stats collector
1 S postgres    5229    5222  0  80   0 - 54553 -      18:21 ?        00:00:00 postgres: 14/main: logical replication launcher
0 S Student     5711     911  0  80   0 -  2042 pipe_r 18:33 pts/0    00:00:00 grep --color=auto postg
```
Так выглятит статус для postgresql.service 
```
Student@ub20-postgres-2:~$ sudo systemctl status postgresql.service
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2021-10-12 17:48:59 UTC; 45min ago
    Process: 5010 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 5010 (code=exited, status=0/SUCCESS)

Oct 12 17:47:53 ub20-postgres-2 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 17:47:53 ub20-postgres-2 systemd[1]: Finished PostgreSQL RDBMS.
Oct 12 17:48:59 ub20-postgres-2 systemd[1]: postgresql.service: Succeeded.
Oct 12 17:48:59 ub20-postgres-2 systemd[1]: Stopped PostgreSQL RDBMS.
```

#### 16 зайдите через через psql и проверьте содержимое ранее созданной таблицы
##### Результат:
```
Student@ub20-postgres-2:~$ sudo -u postgres -i psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges
-----------+----------+----------+---------+---------+-----------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 test1     | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
(4 rows)

postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

test1=# select * from test;
 c1
----
 1
(1 row)
```

Все на месте.

#### 17 задание со звездочкой \* : не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
##### Результат: работает
##### как получилось:
#### 17.1 Остановим postgres
```
Student@ub20-postgres-2:~$ sudo -u postgres pg_ctlcluster 14 main stop
```
и проверим что процессы завершились
```
Student@ub20-postgres-2:~$ ps -elf | grep postgre
0 R Student     5794     911  0  80   0 -  2042 -      18:53 pts/0    00:00:00 grep --color=auto postgre
```
#### 17.2 отмонтируем диск с данными
```
Student@ub20-postgres-2:~$ sudo umount /dev/sdb1
```
и проверим что он не подмонтирован никуда ()
```
Student@ub20-postgres-2:~$ sudo df -h |grep sdb1
```
получили пустой вывод - ок точек монторования нет

#### 17.3 отключим диск от ВМ.
отключаем диск от инстанса в гугл SDK (CLI) командой вида:
```
gcloud compute instances detach-disk my-instance --disk=my-disk
```
В моем случае это:
```
gcloud compute instances detach-disk ub20-postgres-2 --disk=pg-data-d1
```
Вернемся на хост и Убедимся что диска больше нет в списке
```
Student@ub20-postgres-2:~$ sudo lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0     7:0    0  55.4M  1 loop /snap/core18/2128
loop1     7:1    0  61.8M  1 loop /snap/core20/1081
loop2     7:2    0 248.8M  1 loop /snap/google-cloud-sdk/199
loop3     7:3    0  67.3M  1 loop /snap/lxd/21545
loop4     7:4    0  32.3M  1 loop /snap/snapd/13170
sda       8:0    0    10G  0 disk
+-sda1    8:1    0   9.9G  0 part /
+-sda14   8:14   0     4M  0 part
L-sda15   8:15   0   106M  0 part /boot/efi
```
#### 17.4 Настроим еще один хост по аналогии с ub20-postgres-2
Назовем его ub20-postgres-3 (опущу шаги так как только-что далали такойже хост)

#### 17.5 подключаем диск к инстансу в гугл SDK (CLI) командой:
```
gcloud compute instances attach-disk ub20-postgres-3 --disk pg-data-d1
```
Перейдем на хост ub20-postgres-3 
Проверим что диск появился сразу с разделом
```
Student@ub20-postgres-3:~$ lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0     7:0    0  55.4M  1 loop /snap/core18/2128
loop1     7:1    0  61.8M  1 loop /snap/core20/1081
loop2     7:2    0 248.8M  1 loop /snap/google-cloud-sdk/199
loop3     7:3    0  32.3M  1 loop /snap/snapd/13170
loop4     7:4    0  67.3M  1 loop /snap/lxd/21545
sda       8:0    0    10G  0 disk
+-sda1    8:1    0   9.9G  0 part /
+-sda14   8:14   0     4M  0 part
L-sda15   8:15   0   106M  0 part /boot/efi
sdb       8:16   0    10G  0 disk
L-sdb1    8:17   0    10G  0 part
```

Нам осталось создать каталог под точку монтирования и примонтировать туда диск
```
Student@ub20-postgres-3:~$ sudo mkdir /mnt/data
Student@ub20-postgres-3:~$ sudo mount /dev/sdb1 /mnt/data
```
Проверим наличие данных и права на каталогах
```
Student@ub20-postgres-3:~$ sudo ls -la /mnt/data
total 28
drwxr-xr-x 4 postgres postgres  4096 Oct 12 17:44 .
drwxr-xr-x 3 root     root      4096 Oct 12 19:28 ..
drwxr-xr-x 3 postgres postgres  4096 Oct 12 16:40 14
drwx------ 2 postgres postgres 16384 Oct 12 17:26 lost+found
```
да каталог на месте и права у пользователя postgres (чудо возможно так как пользователь при создании получает один и тот же ид не зависимо от хоста на котором проводилась установка)

#### 17.6 останавливаем сервис postgresql
```
Student@ub20-postgres-3:~$ sudo systemctl stop postgresql
Student@ub20-postgres-3:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2021-10-12 19:42:21 UTC; 1min 6s ago
    Process: 4827 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 4827 (code=exited, status=0/SUCCESS)

Oct 12 19:41:31 ub20-postgres-3 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 19:41:31 ub20-postgres-3 systemd[1]: Finished PostgreSQL RDBMS.
Oct 12 19:42:21 ub20-postgres-3 systemd[1]: postgresql.service: Succeeded.
Oct 12 19:42:21 ub20-postgres-3 systemd[1]: Stopped PostgreSQL RDBMS.
```
вносим наш новый путь в конфиг 
```
Student@ub20-postgres-3:~$ sudo -u postgres -i vim /etc/postgresql/14/main/postgresql.conf
```
#было
```
#data_directory = '/var/lib/postgresql/14/main'         # use data in another directory
```
стало
```
data_directory = '/mnt/data/14/main'            # use data in another directory
```
запускаем сервис и проверяем что база "та самая"
```
Student@ub20-postgres-3:~$ sudo systemctl start postgresql
Student@ub20-postgres-3:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2021-10-12 19:45:59 UTC; 5s ago
    Process: 4882 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 4882 (code=exited, status=0/SUCCESS)

Oct 12 19:45:59 ub20-postgres-3 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 19:45:59 ub20-postgres-3 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-3:~$ sudo -u postgres -i psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# \l+
                                                                List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+---------+---------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 8529 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8377 kB | pg_default | unmodifiable empty database
           |          |          |         |         | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8529 kB | pg_default | default template for new databases
           |          |          |         |         | postgres=CTc/postgres |         |            |
 test1     | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 8401 kB | pg_default |
(4 rows)

postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

test1=# select * from test;
 c1
----
 1
(1 row)

test1=# \q
```

#### 17.7 если хотим чтобы после перезагрузки диск автоматом подмапился то 
добавляем в /etc/fstab строчку
```
LABEL=DataPGC   /mnt/data       ext4    defaults        0 2
```
- где DataPGC метка нашего диска
для того чтобы проверить что мы не ошиблись останавливаем сервис постгреса 
`sudo systemctl stop postgresql`
и отмонтируем диск 
`sudo umount /dev/sdb1`
и затем монтируем все что есть в fstab командой 
`sudo mount -a`
и проверяем что все на месте
```
Student@ub20-postgres-3:~$ sudo df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       9.6G  2.0G  7.6G  21% /
devtmpfs        2.0G     0  2.0G   0% /dev
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           393M  932K  392M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/loop0       56M   56M     0 100% /snap/core18/2128
/dev/loop1       62M   62M     0 100% /snap/core20/1081
/dev/loop2      249M  249M     0 100% /snap/google-cloud-sdk/199
/dev/loop3       33M   33M     0 100% /snap/snapd/13170
/dev/loop4       68M   68M     0 100% /snap/lxd/21545
/dev/sda15      105M  5.2M  100M   5% /boot/efi
tmpfs           393M     0  393M   0% /run/user/1002
/dev/sdb1       9.8G   86M  9.2G   1% /mnt/data
```
#### 17.8 Можно стартовать postgresql обратно или перезагрузить хост чтобы проверить стартует ли сервис корректно автоматом
Проверил: После перезагрузки все работает
```
Student@ub20-postgres-3:~$ sudo systemctl status postgresql
? postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2021-10-12 20:12:39 UTC; 1min 30s ago
    Process: 846 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 846 (code=exited, status=0/SUCCESS)

Oct 12 20:12:39 ub20-postgres-3 systemd[1]: Starting PostgreSQL RDBMS...
Oct 12 20:12:39 ub20-postgres-3 systemd[1]: Finished PostgreSQL RDBMS.
Student@ub20-postgres-3:~$ sudo -u postgres -i psql
psql (14.0 (Ubuntu 14.0-1.pgdg20.04+1))
Type "help" for help.

postgres=# \l+
                                                                List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+---------+---------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 8529 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8377 kB | pg_default | unmodifiable empty database
           |          |          |         |         | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8529 kB | pg_default | default template for new databases
           |          |          |         |         | postgres=CTc/postgres |         |            |
 test1     | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 8553 kB | pg_default |
(4 rows)

postgres=# \q
Student@ub20-postgres-3:~$ uptime
 20:14:53 up 2 min,  1 user,  load average: 0.17, 0.22, 0.09
Student@ub20-postgres-3:~$
```

#### 18 проверяем что все ВМ остановлены

NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
ub20-postgres-1  us-central1-c  e2-medium                  10.128.0.3                TERMINATED
ub20-postgres-2  us-central1-c  e2-medium                  10.128.0.4                TERMINATED
ub20-postgres-3  us-central1-c  e2-medium                  10.128.0.5                TERMINATED

### PS 
#### Название проекта:

postgres2021-19810421

#### Заметки
На самом деле в `/mnt` обычно не монтируют на постоянной, основе, но так как было это не прод проект не стал заморачиваться с каталогом под точку монтирования, для тестовых целей это не важно.

Теперь важно не забыть про `/etc/fstab` и перед отключением диска удалить запись о диске в нем.

