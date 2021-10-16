# Домашнее задание (3)
## Установка и настройка PostgteSQL в контейнере Docker
### Цель:
1. установить PostgreSQL в Docker контейнере
2. настроить контейнер для внешнего подключения

### Задание и ход выполнения по этапам: 
#### 1. сделать в GCE инстанс с Ubuntu 20.04
##### Результат:
```
Created.
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
ub20-postgres-4  us-central1-c  e2-medium                  10.128.0.6   35.184.68.27  RUNNING
```
#### 2. поставить на нем Docker Engine
##### Результат:
```
Student@ub20-postgres-4:~$ curl -fsSL https://get.docker.com -o get-docker.sh
Student@ub20-postgres-4:~$ sudo sh get-docker.sh
# Executing docker install script, commit: 93d2499759296ac1f9c510605fef85052a2c32be
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | gpg --dearmor --yes -o /usr/share/keyrings/docker-archive-keyring.gpg
+ sh -c echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu focal stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends  docker-ce-cli docker-scan-plugin docker-ce >/dev/null
+ version_gte 20.10
+ [ -z  ]
+ return 0
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce-rootless-extras >/dev/null
+ sh -c docker version
Client: Docker Engine - Community
 Version:           20.10.9
 API version:       1.41
 Go version:        go1.16.8
 Git commit:        c2ea9bc
 Built:             Mon Oct  4 16:08:29 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.9
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.8
  Git commit:       79ea9d3
  Built:            Mon Oct  4 16:06:37 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.11
  GitCommit:        5b46e404f6b9f661a205e28d59c982d3634148f8
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================

Student@ub20-postgres-4:~$ rm get-docker.sh
Student@ub20-postgres-4:~$ sudo usermod -aG docker $USER
```
 Проверяем что докер откликается на запросы и заодно смотрим версию
``` 
Student@ub20-postgres-4:~$ sudo docker --version
Docker version 20.10.9, build c2ea9bc
```

3 сделать каталог /var/lib/postgres
Результат:
Создали и убедились что каталог пустой
```
Student@ub20-postgres-4:~$ sudo mkdir /var/lib/postgres
Student@ub20-postgres-4:~$ sudo ls -la /var/lib/postgres
total 8
drwxr-xr-x  2 root root 4096 Oct 16 13:27 .
drwxr-xr-x 40 root root 4096 Oct 16 13:27 ..
```

#### 4. развернуть контейнер с PostgreSQL 13 смонтировав в него /var/lib/postgres
##### Результат:
Сразу создадим сеть
```
Student@ub20-postgres-4:~$ sudo docker network create pg-net
e2b23c2733ee039073a4f31f5fca3e8f6461db33b9207300834be400edb94800
```
Теперь создадим сам контейнер
```
Student@ub20-postgres-4:~$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:13
Unable to find image 'postgres:13' locally
13: Pulling from library/postgres
7d63c13d9b9b: Pull complete
cad0f9d5f5fe: Pull complete
ff74a7a559cb: Pull complete
c43dfd845683: Pull complete
e554331369f5: Pull complete
d25d54a3ac3a: Pull complete
bbc6df00588c: Pull complete
d4deb2e86480: Pull complete
d4132927c0d9: Pull complete
3d03efa70ed1: Pull complete
645312b7d892: Pull complete
3cc7074f2000: Pull complete
5d6e98ee16de: Pull complete
Digest: sha256:5d6e513ae9b936e5cd7bff5f1eb658d17512a49e2bb1902ac4c85b7ac8627308
Status: Downloaded newer image for postgres:13
5f9c3cf02689aa29e834a6be4abb5b7b95a2c46ed3b1f4d59ee2c21040087717

```
Видим что докер скачал необходимые базовые образы и завершил сборку

Теперь проверим, что контейнер запущен
```
Student@ub20-postgres-4:~$ sudo docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
5f9c3cf02689   postgres:13   "docker-entrypoint.s…"   18 seconds ago   Up 15 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
```
И убедимся, что в нашем каталоге появились файлы:
```
Student@ub20-postgres-4:~$ sudo ls -la /var/lib/postgres
total 132
drwx------ 19 systemd-coredump root              4096 Oct 16 13:33 .
drwxr-xr-x 40 root             root              4096 Oct 16 13:27 ..
-rw-------  1 systemd-coredump systemd-coredump     3 Oct 16 13:33 PG_VERSION
drwx------  5 systemd-coredump systemd-coredump  4096 Oct 16 13:33 base
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 global
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_commit_ts
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_dynshmem
-rw-------  1 systemd-coredump systemd-coredump  4782 Oct 16 13:33 pg_hba.conf
-rw-------  1 systemd-coredump systemd-coredump  1636 Oct 16 13:33 pg_ident.conf
drwx------  4 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_logical
drwx------  4 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_multixact
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_notify
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_replslot
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_serial
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_snapshots
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_stat
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_stat_tmp
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_subtrans
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_tblspc
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_twophase
drwx------  3 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_wal
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_xact
-rw-------  1 systemd-coredump systemd-coredump    88 Oct 16 13:33 postgresql.auto.conf
-rw-------  1 systemd-coredump systemd-coredump 28156 Oct 16 13:33 postgresql.conf
-rw-------  1 systemd-coredump systemd-coredump    36 Oct 16 13:33 postmaster.opts
-rw-------  1 systemd-coredump systemd-coredump    94 Oct 16 13:33 postmaster.pid

```

#### 5. развернуть контейнер с клиентом postgres
##### Результат:
Выполним команду:
```
Student@ub20-postgres-4:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-docker -U postgres
```
Введем пароль и посмотрим, сто в результате клиент подключился 
```
Password for user postgres:
psql (13.4 (Debian 13.4-4.pgdg110+1))
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg-docker" (address "172.18.0.2") at port "5432".

```

#### 6. подключиться из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
##### Результат: 

Так как мы уже подключены к клиенту, а создавать таблицы в базе postgres  не хотим. то создадим свою БД и проверим что она благополучно добавилась:
```
postgres=# create database dockertestdb;
CREATE DATABASE
postgres=# \l+
                                                                    List of databases
     Name     |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   |  Size   | Tablespace |                Description
--------------+----------+----------+------------+------------+-----------------------+---------+------------+--------------------------------------------
 dockertestdb | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       | 7753 kB | pg_default |
 postgres     | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       | 7901 kB | pg_default | default administrative connection database
 template0    | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7753 kB | pg_default | unmodifiable empty database
              |          |          |            |            | postgres=CTc/postgres |         |            |
 template1    | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7753 kB | pg_default | default template for new databases
              |          |          |            |            | postgres=CTc/postgres |         |            |
(5 rows)
```
затем перейдем в неё, создадим таблицу и добавим в неё 3 строки:
```
postgres=# \c dockertestdb
You are now connected to database "dockertestdb" as user "postgres".
dockertestdb=# CREATE TABLE test (i serial, amount int);
CREATE TABLE
dockertestdb=# INSERT INTO test(amount) VALUES (100);
INSERT 0 1
dockertestdb=# INSERT INTO test(amount) VALUES (120);
INSERT 0 1
dockertestdb=# INSERT INTO test(amount) VALUES (160);
INSERT 0 1
```
Убедимся в существовании таблицы и проверим её содержимое:
```
dockertestdb=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

dockertestdb=# select * from test;
 i | amount
---+--------
 1 |    100
 2 |    120
 3 |    160
(3 rows)
```

#### 7. подключиться к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP
##### Результат:
Чтобы подключение стало возможным:

- Добавил разрешения на подключение к серверу по порту 5432 в web интрефейсе GCP (назвал правило pg-for-dc-test, разрешает с любого хоста на любой хост в подсети доступ по порту 5432)
{надо же было сеть в web интерфейсе чуть не в самый низ запихать}
- Посмотрел в SDK (CLI) ip адрес сервера EXTERNAL_IP  
```
gcloud compute instances list
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
ub20-postgres-4  us-central1-c  e2-medium                  10.128.0.6   35.184.68.27  RUNNING
```
- Запустил ubuntu на виртуалке на своем ПК, зашел в терминал, установил 13й postgres 
```
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-13
```
Проверил коннект до сервера (доступен ли порт)
```
postgres@ws2021a2:~$ nc -vz 35.184.68.27 5432
Connection to 35.184.68.27 5432 port [tcp/postgresql] succeeded!
```
Подключился клиентом
```
student@ws2021a2:~$ sudo -u postgres -i psql -p 5432 -U postgres -h 35.184.68.27 -d postgres -W
Password: 
psql (13.4 (Ubuntu 13.4-4.pgdg20.04+1))
Type "help" for help.

postgres=# \conninfo 
You are connected to database "postgres" as user "postgres" on host "35.184.68.27" at port "5432".
postgres=# \l
                                  List of databases
     Name     |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
--------------+----------+----------+------------+------------+-----------------------
 dockertestdb | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0    | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
              |          |          |            |            | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
              |          |          |            |            | postgres=CTc/postgres
(4 rows)

```

#### 8. удалить контейнер с сервером
##### Результат: 
Найдем id контейнера
```
Student@ub20-postgres-4:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED             STATUS             PORTS                                       NAMES
49d3459fbb54   postgres:13   "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
```

Остановим его и затем удалим:
```
Student@ub20-postgres-4:~$ sudo docker stop 49d3459fbb54
49d3459fbb54
Student@ub20-postgres-4:~$ sudo docker rm 49d3459fbb54
49d3459fbb54
```

Проверим что наша директория с базой не пустая:
```
Student@ub20-postgres-4:~$ sudo ls -la /var/lib/postgres
total 128
drwx------ 19 systemd-coredump root              4096 Oct 16 20:31 .
drwxr-xr-x 40 root             root              4096 Oct 16 13:27 ..
-rw-------  1 systemd-coredump systemd-coredump     3 Oct 16 13:33 PG_VERSION
drwx------  6 systemd-coredump systemd-coredump  4096 Oct 16 19:28 base
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 19:20 global
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_commit_ts
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_dynshmem
-rw-------  1 systemd-coredump systemd-coredump  4782 Oct 16 13:33 pg_hba.conf
-rw-------  1 systemd-coredump systemd-coredump  1636 Oct 16 13:33 pg_ident.conf
drwx------  4 systemd-coredump systemd-coredump  4096 Oct 16 20:31 pg_logical
drwx------  4 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_multixact
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_notify
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_replslot
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_serial
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_snapshots
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 20:31 pg_stat
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 20:31 pg_stat_tmp
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_subtrans
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_tblspc
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_twophase
drwx------  3 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_wal
drwx------  2 systemd-coredump systemd-coredump  4096 Oct 16 13:33 pg_xact
-rw-------  1 systemd-coredump systemd-coredump    88 Oct 16 13:33 postgresql.auto.conf
-rw-------  1 systemd-coredump systemd-coredump 28156 Oct 16 13:33 postgresql.conf
-rw-------  1 systemd-coredump systemd-coredump    36 Oct 16 19:19 postmaster.opts

```

#### 9. создать его заново
##### Результат:
```
Student@ub20-postgres-4:~$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:13
e113a6933ff29253aa805fe9f50f81d37d44f5af7e1e32dac6d01838d3248c20
```
Контейнер создан и запущен

#### 10. подключиться снова из контейнера с клиентом к контейнеру с сервером и проверить, что данные остались на месте
Результат:
```
Student@ub20-postgres-4:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-docker -p 5432 -U postgres
Password for user postgres:
psql (13.4 (Debian 13.4-4.pgdg110+1))
Type "help" for help.

postgres=# \l
                                  List of databases
     Name     |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
--------------+----------+----------+------------+------------+-----------------------
 dockertestdb | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres     | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0    | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
              |          |          |            |            | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
              |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \c dockertestdb
You are now connected to database "dockertestdb" as user "postgres".
dockertestdb=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

dockertestdb=# select * from test;
 i | amount
---+--------
 1 |    100
 2 |    120
 3 |    160
(3 rows)

```

#### 12. оставляйте в ЛК ДЗ комментарии, что и как вы делали и как боролись с проблемами

#### Кроме того:
Создал еще одно правило для CGP fireWall средствами SDK (CLI)
```
gcloud compute --project=postgres2021-19810421 firewall-rules create for-pg-non-st-port --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:5436 --source-ranges=0.0.0.0/0 --target-tags=ub20-postgres-4
```
Основное отличие данного правила в том, что оно распространяется на хосты помеченные тегом ub20-postgres-4 ну и порт конечно другой (5436)
Создал еще один каталог (/opt/postgres/data) и еще один контейнер с базой (назвал pg-docker2)
```
sudo docker run --name pg-docker2 --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5436:5432 -v /opt/postgres/data:/var/lib/postgresql/data postgres:13
```
с указанным портом на стороне хоста и портом 5432 на стороне контейнера и подключился к нему с внешнего узла (со своей домашней виртуалки)
```
postgres@ws2021a2:~$ psql -p 5436 -U postgres -h 35.184.68.27 -d postgres -W
Password: 
psql (13.4 (Ubuntu 13.4-4.pgdg20.04+1))
Type "help" for help.

postgres=# \l
                                     List of databases
       Name        |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-------------------+----------+----------+------------+------------+-----------------------
 anothercontainier | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres          | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0         | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                   |          |          |            |            | postgres=CTc/postgres
 template1         | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                   |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \q

```
Если в CGP web интерфейсе отключить правило на firewall  или снять с хоста метку, то доступ к контейнеру с внешнего хоста пропадает.
Таким образом можно подключеться по разным портам к разным инстансам postgres, размещенным в разных контейнерах на одном хосте и при этом не мешающим работать друг другу,
как с того же хоста так и с удаленного. 

Существует особенность, при пробрасывании разных портов, обращение (подключение) к приложению в контейнере локально по имени контейнера требует 
указания того порта на котором приложение ожидает подключения внутри контейнера, 
если же при подключении к контейнеру используется ip адрес хоста, то необходимо указывать порт который мы смапили для самого хоста носителя докер контейнеров.
Если указать внешний по отношению к контейнеру порт и, при этом, указать имя контейнера в качастве хоста, то получим ошибку:
```
Student@ub20-postgres-4:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-docker2 -p 5436 -U postgres
psql: error: connection to server at "pg-docker2" (172.18.0.3), port 5436 failed: Connection refused
        Is the server running on that host and accepting TCP/IP connections?
Student@ub20-postgres-4:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-docker2 -p 5432 -U postgres
Password for user postgres:
psql (13.4 (Debian 13.4-4.pgdg110+1))
Type "help" for help.

postgres=# \l
                                     List of databases
       Name        |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-------------------+----------+----------+------------+------------+-----------------------
 anothercontainier | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres          | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0         | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                   |          |          |            |            | postgres=CTc/postgres
 template1         | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                   |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \q

Student@ub20-postgres-4:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h 35.184.68.27 -p 5436 -U postgres
Password for user postgres:
psql (13.4 (Debian 13.4-4.pgdg110+1))
Type "help" for help.

postgres=# \l
                                     List of databases
       Name        |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-------------------+----------+----------+------------+------------+-----------------------
 anothercontainier | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres          | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0         | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                   |          |          |            |            | postgres=CTc/postgres
 template1         | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                   |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \q

```

#### PS останавливаем контейнеры и ВМ.
```
gcloud compute instances list
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
ub20-postgres-4  us-central1-c  e2-medium                  10.128.0.6                TERMINATED
```
