�������� �������
������ � �������� �������� ���������� � PostgreSQL

����:
��������� �������� � Google Cloud Platform �� ������ Google Compute Engine (IaaS)
��������� ��������� ������� ������� ��������� � PostgreSQL � �������� ����������� ������ ������� read commited � repeatable read

1 ������� ����� ������ � Google Cloud Platform, �������� postgres2021-, ��� yyyymmdd ���, ����� � ���� ������ �������� (��� ������� ������ ���� ��������� �� ������ GCP)
������: ������ ������ postgres2021-19810421

2 ���� ����������� ������� � ����� ������� ������������ ifti@yandex.ru � ����� Project Editor
���� ������ ����� ����� web  ��������� IAM
������: ������ ����� editor �� ������ � IAM

3 ����� ������� ������� ����������� ������ Compute Engine � ���������� �����������
�������� ���� ssh ���� � GCE metadata
������:
"������� ��: NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
ub20-postgres-1  us-central1-c  e2-medium                  10.128.0.3                TERMINATED"
��������� 2 ����� �������� � �������������� 
������ �������� ����� CLI (SDK) {������������� ��� �����������}
������ �������� ����� web  ���������

4 ����� ��������� ssh (������ ������), �� ��������� ��� ssh-add
��������� PostgreSQL
������: �������� ������ ��� Student
��������� ������
psql (PostgreSQL) 14.0 (Ubuntu 14.0-1.pgdg20.04+1)

5 ����� ������ ssh (������ ������)
������: +

6 ��������� ����� psql �� ��� ������������ postgres
������: +

7: ��������� auto commit
������:
\set AUTOCOMMIT OFF

8: ������� � ������ ������ ����� ������� � ��������� �� ������� create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
������: +

9: ���������� ������� ������� ��������: show transaction isolation level
������:
"iso=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
"
10: ������ ����� ���������� � ����� ������� � ��������� (�� �����) ������� ��������
������:
begin;


11 � ������ ������ �������� ����� ������ insert into persons(first_name, second_name) values('sergey', 'sergeev');
������: +

12: ������� select * from persons �� ������ ������
������ �� �� ����� ������ � ���� �� �� ������?
--
���, ��������� �������:
iso=*# select * from persons; �������� 2 ������
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

��� ��� ���������� � ������ ������ �� ���������, � ��������� ������ ��������� read_commited ������������� ����������� ������ ���������� � ����� ����������� ��� ������ ���������� 
������ ����� ��������� ���������� ���������� (�������) � ��� �� � ������ ������ ���������� ��� �� ���������."

13: ��������� ������ ���������� - commit;
������:

14: ������� select * from persons �� ������ ������
������:
"iso=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
"

15: ������ �� �� ����� ������ � ���� �� �� ������?
��, ������ ������ ����������� � ���������� �� ������ ������ �������� �������� �� ������ ����������, ������� ����� ���������� ������� (�� ������ ����� ������, �� ������ ����), 
��� ��� ������ ���������� ������� ��������� (�����������). ������������� ������ �������� �� ������� ������ ��������� ����������. (read_commited)


16: ��������� ���������� �� ������ ������
������: +
"iso=*# commit;
COMMIT
"

17: ������ ����� �� ��� repeatable read ��������� - set transaction isolation level repeatable read;
������: �������� ��������� � ����� �������
"iso=# set transaction isolation level repeatable read;
SET
iso=*# begin;
WARNING:  there is already a transaction in progress
BEGIN
"

18: � ������ ������ �������� ����� ������ insert into persons(first_name, second_name) values('sveta', 'svetova');
������:
"iso=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1"


19: ������� select * from persons �� ������ ������
������: 
"iso=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
"

20: ������ �� �� ����� ������ � ���� �� �� ������?
���, ������ ��������������� ������ �� ������ ���������� �� �������� �� ���� ���������� postgres  ������� ��������� ����������


21: ��������� ������ ���������� - commit;
������: +

22: ������� select * from persons �� ������ ������
������: ��������� �����

23: ������ �� �� ����� ������ � ���� �� �� ������?
���. ���� ������ ���������� ������� ���������, �� �� ����� ��������� ������ � �������, ��� ��� �� ������ ��������� repeatable read 
�� ���� ��������� �� ����� ������ ������ ���������� ����� � ������ (������������� ������ ���������� �� ������ ���������=repeatable read)


24: ��������� ������ ����������
������:

25: ������� select * from persons �� ������ ������
������: +
������ �� �� ����� ������ � ���� �� �� ������?
��, ������ ������ ����������� � ���������� �� ������ ������ �������� �������� �� ������ ���������� ������� ����� ���������� ���������� �� ������ ������ 
(�� ������ ����� ������, �� ������ ����, ��� ������ ����� � ����������), ��� ��� ������ ���������� ������� ��������� (�����������). 
��� ��� ��� ������ ������(�����) ����� �������� ������ �����������, �� ��� ��� ��������

26:���������� ����������� ������ �� �� �������� ��
������:
��������� ������ postgres, ����� ��������� ��
sudo systemctl stop postgresql
sudo systemctl status postgresql
sudo shutdown now
����� 2 ������ �������� ��� �� �����������:
gcloud compute instances list
NAME             ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
ub20-postgres-1  us-central1-c  e2-medium                  10.128.0.3                TERMINATED
