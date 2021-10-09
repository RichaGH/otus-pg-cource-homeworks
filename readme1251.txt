Домашнее задание
Работа с уровнями изоляции транзакции в PostgreSQL

Цель:
научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS)
научиться управлять уровнем изолции транзации в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

1 создать новый проект в Google Cloud Platform, например postgres2021-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально на уровне GCP)
готово: Создан проект postgres2021-19810421

2 дать возможность доступа к этому проекту пользователю ifti@yandex.ru с ролью Project Editor
Роль Едитор выдал через web  интерфейс IAM
готово: выданы права editor на проект в IAM

3 далее создать инстанс виртуальной машины Compute Engine с дефолтными параметрами
добавить свой ssh ключ в GCE metadata
готово:
"создана ВМ: NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
ub20-postgres-1  us-central1-c  e2-medium                  10.128.0.3                TERMINATED"
Добавлены 2 ключа основной и дополнительный 
Первый добавлен через CLI (SDK) {автоматически при подключении}
Второй добавлен через web  интерфейс

4 зайти удаленным ssh (первая сессия), не забывайте про ssh-add
поставить PostgreSQL
готово: постгрес ставил поз Student
установил версия
psql (PostgreSQL) 14.0 (Ubuntu 14.0-1.pgdg20.04+1)

5 зайти вторым ssh (вторая сессия)
готово: +

6 запустить везде psql из под пользователя postgres
готово: +

7: выключить auto commit
готово:
\set AUTOCOMMIT OFF

8: сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
готово: +

9: посмотреть текущий уровень изоляции: show transaction isolation level
готово:
"iso=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
"
10: начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
готово:
begin;


11 в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
готово: +

12: сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
--
НЕТ, результат запроса:
iso=*# select * from persons; Содержит 2 строки
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

Так как транзакция в первой сессии не завершена, а настройка уровня иззоляции read_commited подразумевает доступность данных измененных в одних транзакциях для других транзакций 
только после успешного завершения транзакции (коммита) У нас же в первой сессии транзакция еще не завершена."

13: завершить первую транзакцию - commit;
готово:

14: сделать select * from persons во второй сессии
готово:
"iso=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
"

15: видите ли вы новую запись и если да то почему?
Да, теперь данные обновленные в транзакции из первой сессии доступны запросам из других транзакций, начатым после завершения коммита (не только нашей второй, но вообще всем), 
так как первая транзакция успешно завершена (закоммичена). Неповторяемое чтение возможно на текущем уровне иззоляции транзакций. (read_commited)


16: завершите транзакцию во второй сессии
готово: +
"iso=*# commit;
COMMIT
"

17: начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
готово: операция выполнена в обоих сессиях
"iso=# set transaction isolation level repeatable read;
SET
iso=*# begin;
WARNING:  there is already a transaction in progress
BEGIN
"

18: в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
готово:
"iso=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1"


19: сделать select * from persons во второй сессии
готово: 
"iso=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
"

20: видите ли вы новую запись и если да то почему?
Нет, чтение незакоммиченных данных из других транзакций не доступно на всех допустимых postgres  уровнях иззоляции транзакций


21: завершить первую транзакцию - commit;
Готово: +

22: сделать select * from persons во второй сессии
готово: результат тотже

23: видите ли вы новую запись и если да то почему?
Нет. Хотя первая транзакция успешно завершена, мы не видим четвертой строки в выборке, так как на уровне иззоляции repeatable read 
ни одна транзация не может видеть данные измененные после её начала (Неповторяемое чтение невозможно на уровне иззоляции=repeatable read)


24: завершить вторую транзакцию
готово:

25: сделать select * from persons во второй сессии
готово: +
видите ли вы новую запись и если да то почему?
Да, теперь данные обновленные в транзакции из первой сессии доступны запросам из других транзакций начатых после завершения транзакции из первой сессии 
(не только нашей второй, но вообще всем, что начаты после её завершения), так как первая транзакция успешно завершена (закоммичена). 
Так как наш запрос сделан(начат) после фиксации данных транзакцией, то они ему доступны

26:остановите виртуальную машину но не удаляйте ее
готово:
остановил сервис postgres, затем остановил ВМ
sudo systemctl stop postgresql
sudo systemctl status postgresql
sudo shutdown now
Через 2 минуты проверил что ВМ остановлена:
gcloud compute instances list
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
ub20-postgres-1  us-central1-c  e2-medium                  10.128.0.3                TERMINATED
