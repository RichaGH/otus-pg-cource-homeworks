# Домашнее задание
## Секционирование таблицы

### Цель:
научиться секционировать таблицы.
Секционировать большую таблицу из демо базы flights

#### 1. Выбор таблицы:
##### Результат:
- После загрузки базы demo с сайта https://postgrespro.com/education/demodb
- Посмотрим список таблиц:
```
demo=# \dt+
                                                List of relations
  Schema  |      Name       | Type  |  Owner   | Persistence | Access method |  Size  |        Description
----------+-----------------+-------+----------+-------------+---------------+--------+---------------------------
 bookings | aircrafts_data  | table | postgres | permanent   | heap          | 16 kB  | Aircrafts (internal data)
 bookings | airports_data   | table | postgres | permanent   | heap          | 56 kB  | Airports (internal data)
 bookings | boarding_passes | table | postgres | permanent   | heap          | 455 MB | Boarding passes
 bookings | bookings        | table | postgres | permanent   | heap          | 105 MB | Bookings
 bookings | flights         | table | postgres | permanent   | heap          | 21 MB  | Flights
 bookings | seats           | table | postgres | permanent   | heap          | 96 kB  | Seats
 bookings | ticket_flights  | table | postgres | permanent   | heap          | 547 MB | Flight segment
 bookings | tickets         | table | postgres | permanent   | heap          | 386 MB | Tickets
(8 rows)

```

- возьмем для наших целей первую большую таблицу:
```
boarding_passes 
```

#### 2. Подготовка к секционированию:
#### 2.0 Замечания о способе секционирования:
##### Результат:

- Объективно: выбор способа секционирования зависит от версии PostgreSQL так как в 9ке например физически нет декларативоного секционирования, а также от целей секционирования.
- Так как одной из наших целей является тренирока навыков необходимых для декларативного секционирования, то выберем именно этот способ 

- Кроме того, очень важным замечанием является критерий необходимости бесперебойного функционирования БД, так как если мы можем остановить работу приложений (отключить пользователей от БД), задача упрощается. Мы же расмотрим только случай когда прерывать работу приложения нельзя.
#### 2.1 Выбор критериев секционирования:
##### Результат:
- Итак: надо определить зачем мы хотим секционировать таблицу.
- Предположим, что она слишком велика и мы хотим за счет секционирования ускорить работу:
- Первый вариант: - это размещение всех часто используемых данных в одной "активной секции" и вынос всех остальных в другую, обращения к которой будут пренебрежимо редкими.
- осталось понять есть ли на данной таблице ключ который позвоилит разделить данные интересующим нас способом. 
- К сожалению в самой таблице такого поля (дата время или текст по котором сразу ясно что данные архивные или какя-то такая метка) нет, а выбор декларативного способа секционирования позволяет использовать критерии проверки значений check только для полей самой таблицы, в результате нам прийдется отказаться от этого варианта


- Другой выриант: - Ускорение запросов может быть достигнуто, если размер секции будет достаточно мал, чтобы работа происходила в рамках одной секции, которая польностью умещалась бы в памяти выделенной текущему процессу обработчику
```
demo=# show work_mem ;
 work_mem
----------
 4MB
(1 row)
```
- Так как у нас тестовй стенд, предположим, что 4Мб это просто эквивалент реального значения которое уже подобрано для нашей БД и воспользуемся им в последствии
- значить нам нужно получить минимум: 455/4=113,75 секций, однако реальное количество секций лучше расчитывать после выбора ключа секционирования:

#### 2.2 Выбор ключа секционирования:
##### Результат:
- Посмотрим что представляет из себя выбранная нами таблица:
```
demo=# \d+ boarding_passes
                                                  Table "bookings.boarding_passes"
   Column    |         Type         | Collation | Nullable | Default | Storage  | Compression | Stats target |     Description
-------------+----------------------+-----------+----------+---------+----------+-------------+--------------+----------------------
 ticket_no   | character(13)        |           | not null |         | extended |             |              | Ticket number
 flight_id   | integer              |           | not null |         | plain    |             |              | Flight ID
 boarding_no | integer              |           | not null |         | plain    |             |              | Boarding pass number
 seat_no     | character varying(4) |           | not null |         | extended |             |              | Seat number
Indexes:
    "boarding_passes_pkey" PRIMARY KEY, btree (ticket_no, flight_id)
    "boarding_passes_flight_id_boarding_no_key" UNIQUE CONSTRAINT, btree (flight_id, boarding_no)
    "boarding_passes_flight_id_seat_no_key" UNIQUE CONSTRAINT, btree (flight_id, seat_no)
Foreign-key constraints:
    "boarding_passes_ticket_no_fkey" FOREIGN KEY (ticket_no, flight_id) REFERENCES ticket_flights(ticket_no, flight_id)
Access method: heap

```

- итак мы имеем 4 поля, 2 из которых используются при построении составного первичного ключа таблицы (ticket_no, flight_id) одно представляет собой номер места в салоне и одно номер посадочного талона.
- К сожалению у нас нет полей типа дата по которым мы смогли бы распретделить все записи на "старые" и актуальные. Прийдется рассмотреть то, что имеется:

- (seat_no) Номер места не подходит для построения равномерно заполненных секций, так как есть места на которых пассажиры сидят в любом самолете, а есть такие на которых почти никогда нет пассажира.
- (boarding_no) Номера посадочных талонов сами по себе не несут группирующей информации, более того следует помнить, что в случае "общей" на несколько арэопортов БД сквозная нумерация посадочних талонов в общем случае может помещать "рядом" номера талонов на разные рейсы 
- (ticket_no) Те же соображения по поводу номеров билетов
- (flight_id) Номера перелетов немного интереснее всех предыдущих полей, так как обладают "группирующим эффектом" для билетов и посадочных талонов

- Итак неким группирующим эффектом обладает только одно поле flight_id, но у него есть и недостаток салон самолета члишком мал, будет много секций, нужно придмать как не потеряв + поля (эффект группировки) избежать его недостатков:
- так как flight_id это целое число, мы можем получить стабильное распределение значений между чекциями, если воспользуемся остатком от деления как функцией распределения значений между секциями: при этом с одной стороны все посадочные места одноо перелета будут в одной секции, с другой секции будут плюч минус равномерно заполнены

#### 2.3 Дополнительный анализ:
##### Результат:
- Мы хотим разбить таблицу boarding_passes на секции декларативным способом, не потеряв данные и не прерывая работу БД, время на обслуживание должно быть минимальным.
- Что следует учесть:
- Нельзя унаследовать секционированную таблицу от несекционированной и наоборот,
- Нельзя "конвертировать" несекционированную таблицу в секционированную напрямую командами СУБД
- Можно: Переименовать таблицу, при этом все объекты типа view и trigger продолжат "смотреть" на неё, но уже под новым именем, НО в хранимых процедурах/функциях имя отношения не изменится.

- Проверим есть ли процедуры или функции, которые имеют прямое упоминание нашей таблицы:
```
demo=# \df+
                                                                                            List of functions
  Schema  | Name | Result data type | Argument data types | Type | Volatility | Parallel |  Owner   | Security | Access privileges | Language |                Source code                 | Description
----------+------+------------------+---------------------+------+------------+----------+----------+----------+-------------------+----------+--------------------------------------------+-------------
 bookings | lang | text             |                     | func | stable     | unsafe   | postgres | invoker  |                   | plpgsql  |                                           +|
          |      |                  |                     |      |            |          |          |          |                   |          | BEGIN                                     +|
          |      |                  |                     |      |            |          |          |          |                   |          |   RETURN current_setting('bookings.lang');+|
          |      |                  |                     |      |            |          |          |          |                   |          | EXCEPTION                                 +|
          |      |                  |                     |      |            |          |          |          |                   |          |   WHEN undefined_object THEN              +|
          |      |                  |                     |      |            |          |          |          |                   |          |     RETURN NULL;                          +|
          |      |                  |                     |      |            |          |          |          |                   |          | END;                                      +|
          |      |                  |                     |      |            |          |          |          |                   |          |                                            |
(1 row)

```
- нам повезло функций и процедур использующих таблицу boarding_passes нет

- Важно также чтобы не было view в которых наша таблица участвует в запросах, к счастью среди :
```
demo=# \dv+
                                     List of relations
  Schema  |      Name       | Type |  Owner   | Persistence |  Size   |    Description
----------+-----------------+------+----------+-------------+---------+--------------------
 bookings | aircrafts       | view | postgres | permanent   | 0 bytes | Aircrafts
 bookings | airports        | view | postgres | permanent   | 0 bytes | Airports
 bookings | flights_v       | view | postgres | permanent   | 0 bytes | Flights (extended)
 bookings | routes          | view | postgres | permanent   | 0 bytes | Routes
(4 rows)

```
- таковх нет

#### 3.1 Подготовка. Создание пустой секционированной таблицы:
##### Результат:

```
demo=# create table t1 (like boarding_passes including all) partition by hash(flight_id);
CREATE TABLE
demo=# \d+ t1
                                                  Partitioned table "bookings.t1"
   Column    |         Type         | Collation | Nullable | Default | Storage  | Compression | Stats target |     Description
-------------+----------------------+-----------+----------+---------+----------+-------------+--------------+----------------------
 ticket_no   | character(13)        |           | not null |         | extended |             |              | Ticket number
 flight_id   | integer              |           | not null |         | plain    |             |              | Flight ID
 boarding_no | integer              |           | not null |         | plain    |             |              | Boarding pass number
 seat_no     | character varying(4) |           | not null |         | extended |             |              | Seat number
Partition key: HASH (flight_id)
Indexes:
    "t1_pkey" PRIMARY KEY, btree (ticket_no, flight_id)
    "t1_flight_id_boarding_no_key" UNIQUE CONSTRAINT, btree (flight_id, boarding_no)
    "t1_flight_id_seat_no_key" UNIQUE CONSTRAINT, btree (flight_id, seat_no)
Number of partitions: 0

```
#### 3.2 Подготовка. Создание пустой секционированной таблицы:
##### Результат:
- Предположим что нам будет достаточно 120 равномерно заполненных секций:
- тогда создадим их следующим образом:
```
demo=# do $$
demo$# declare
demo$#     i int4;
demo$# begin
demo$#     for i in 0..119
demo$#     loop
demo$# execute format('create table t1_p%s partition of t1 FOR VALUES WITH (MODULUS 120, REMAINDER %s)',i,i);
demo$#     end loop;
demo$# end;
demo$# $$;
DO
demo=# \dt+
                                                        List of relations
  Schema  |      Name       |       Type        |  Owner   | Persistence | Access method |    Size    |        Description
----------+-----------------+-------------------+----------+-------------+---------------+------------+---------------------------
 bookings | aircrafts_data  | table             | postgres | permanent   | heap          | 16 kB      | Aircrafts (internal data)
 bookings | airports_data   | table             | postgres | permanent   | heap          | 56 kB      | Airports (internal data)
 bookings | boarding_passes | table             | postgres | permanent   | heap          | 455 MB     | Boarding passes
 bookings | bookings        | table             | postgres | permanent   | heap          | 105 MB     | Bookings
 bookings | flights         | table             | postgres | permanent   | heap          | 21 MB      | Flights
 bookings | seats           | table             | postgres | permanent   | heap          | 96 kB      | Seats
 bookings | t1              | partitioned table | postgres | permanent   |               | 0 bytes    |
 bookings | t1_p0           | table             | postgres | permanent   | heap          | 0 bytes    |
 bookings | t1_p1           | table             | postgres | permanent   | heap          | 0 bytes    |
 bookings | t1_p2           | table             | postgres | permanent   | heap          | 0 bytes    |
 bookings | t1_p3           | table             | postgres | permanent   | heap          | 0 bytes    |
 bookings | t1_p4           | table             | postgres | permanent   | heap          | 0 bytes    |
 bookings | t1_p5           | table             | postgres | permanent   | heap          | 0 bytes    |
			*	*	*	
```
- теперь у нас есть пустые таблица и её секции, перейдем к самому интересному:


#### 3.2 Подготовка. Создание хранимой триггерной функции для вьюхи:
##### Результат:
- для того чтобы мы могли безболезненно подменить нашу таблицу её секционированным аналогом, потребуется временно заменить её на view, но при этом следует организовать все так чтобы не потерять возможности обновлять/удалять/добавлять данные
- поэтому нам понадобится "спец" процедура для тириггера :
```
demo=# CREATE OR REPLACE FUNCTION update_boarding_passes_view() RETURNS TRIGGER AS $$
            UPDATE t1 SET (ticket_no,flight_id,boarding_no,seat_no) = (NEW.ticket_no,NEW.flight_id,NEW.boarding_no,new.seat_no) WHERE (ticket_no = OLD.ticket_no)and(flight_id = OLD.flight_id);
            IF NOT FOUND THEN nf1:=2; END IF;

            UPDATE t3_def SET (ticket_no,flight_id,boarding_no,seat_no) = (NEW.ticket_no,NEW.flight_id,NEW.boarding_no,new.seat_no) WHERE (ticket_no = OLD.ticket_no)and(flight_id = OLD.flight_id);
            IF NOT FOUND THEN nf2:=2; END IF;
demo$#     declare

            IF (nf1+nf2)>2 THEN RETURN NULL; END IF;
demo$#     nf1 integer:=0;
demo$#     nf2 integer:=0;
demo$#     BEGIN
demo$#         IF (TG_OP = 'DELETE') THEN
demo$#             DELETE FROM t1 WHERE (ticket_no = OLD.ticket_no)and(flight_id = OLD.flight_id);
demo$#             IF NOT FOUND THEN nf1:=1; END IF;
demo$#
demo$#             DELETE FROM t3_def WHERE (ticket_no = OLD.ticket_no)and(flight_id = OLD.flight_id);
demo$#             IF NOT FOUND THEN nf2:=1; END IF;
demo$#
demo$#             IF (nf1+nf2)>1 THEN RETURN NULL; END IF;
demo$#             RETURN OLD;
demo$#         ELSIF (TG_OP = 'UPDATE') THEN
demo$#             UPDATE t1 SET (ticket_no,flight_id,boarding_no,seat_no) = (NEW.ticket_no,NEW.flight_id,NEW.boarding_no,new.seat_no) WHERE (ticket_no = OLD.ticket_no)and(flight_id = OLD.flight_id);
demo$#             IF NOT FOUND THEN nf1:=2; END IF;
demo$#
demo$#             UPDATE t3_def SET (ticket_no,flight_id,boarding_no,seat_no) = (NEW.ticket_no,NEW.flight_id,NEW.boarding_no,new.seat_no) WHERE (ticket_no = OLD.ticket_no)and(flight_id = OLD.flight_id);
demo$#             IF NOT FOUND THEN nf2:=2; END IF;
demo$#
demo$#             IF (nf1+nf2)>2 THEN RETURN NULL; END IF;
demo$#
demo$#             RETURN NEW;
demo$#         ELSIF (TG_OP = 'INSERT') THEN
demo$#             INSERT INTO t1 VALUES(NEW.ticket_no,NEW.flight_id,NEW.boarding_no,new.seat_no);
demo$#             RETURN NEW;
demo$#         END IF;
demo$#     END;
demo$# $$ LANGUAGE plpgsql;
CREATE FUNCTION

```
- Данная процедура будет связана с триггером в представлении из п 3.3. 

#### 3.3 Подмена1. Преименование таблицы, создание представления и триггера (Все делается разом):
##### Результат:
- Выполним следующие команды:
```
demo=# alter table boarding_passes rename to t3_def;
ALTER TABLE
demo=# create view boarding_passes
demo-# as select * from t1
demo-# union all
demo-# select * from t3_def
demo-# ;
CREATE VIEW
demo=# CREATE TRIGGER boarding_passesvt
demo-# INSTEAD OF INSERT OR UPDATE OR DELETE ON boarding_passes
demo-#     FOR EACH ROW EXECUTE PROCEDURE update_boarding_passes_view();
CREATE TRIGGER

```

- Проверим:
```
demo=# \d+ boarding_passes
                               View "bookings.boarding_passes"
   Column    |         Type         | Collation | Nullable | Default | Storage  | Description
-------------+----------------------+-----------+----------+---------+----------+-------------
 ticket_no   | character(13)        |           |          |         | extended |
 flight_id   | integer              |           |          |         | plain    |
 boarding_no | integer              |           |          |         | plain    |
 seat_no     | character varying(4) |           |          |         | extended |
View definition:
 SELECT t1.ticket_no,
    t1.flight_id,
    t1.boarding_no,
    t1.seat_no
   FROM t1
UNION ALL
 SELECT t3_def.ticket_no,
    t3_def.flight_id,
    t3_def.boarding_no,
    t3_def.seat_no
   FROM t3_def;
Triggers:
    boarding_passesvt INSTEAD OF INSERT OR DELETE OR UPDATE ON boarding_passes FOR EACH ROW EXECUTE FUNCTION update_boarding_passes_view()

```

- а что с запросами?
```
demo=# select * from boarding_passes limit 5;
   ticket_no   | flight_id | boarding_no | seat_no
---------------+-----------+-------------+---------
 0005435189093 |    198393 |           1 | 27G
 0005435189119 |    198393 |           2 | 2D
 0005435189096 |    198393 |           3 | 18E
 0005435189117 |    198393 |           4 | 31B
 0005432208788 |    198393 |           5 | 28C
(5 rows)

```

- база тестовая, бояться нам нечего, проверим, что триггер работает:
```
demo=# delete from boarding_passes where flight_id=198393 and boarding_no=1;
DELETE 1
demo=# insert into boarding_passes (ticket_no,flight_id,boarding_no,seat_no)values('0005435189093',198393,1,'27G');
INSERT 0 1
demo=# select * from boarding_passes where flight_id=198393 and boarding_no=1;
   ticket_no   | flight_id | boarding_no | seat_no
---------------+-----------+-------------+---------
 0005435189093 |    198393 |           1 | 27G
(1 row)

demo=# select * from t3_def where flight_id=198393 and boarding_no=1;
 ticket_no | flight_id | boarding_no | seat_no
-----------+-----------+-------------+---------
(0 rows)

demo=# select * from t1 where flight_id=198393 and boarding_no=1;
   ticket_no   | flight_id | boarding_no | seat_no
---------------+-----------+-------------+---------
 0005435189093 |    198393 |           1 | 27G
(1 row)
```

- а где реально строка? (подглядел и заселектил)
```
demo=# \dt+
demo=# select * from t1_p102;
   ticket_no   | flight_id | boarding_no | seat_no
---------------+-----------+-------------+---------
 0005435189093 |    198393 |           1 | 27G
(1 row)

```

- В результате имеем вьюху которая эмулирует таблицу, и две таблицы: Новую и старую, теперь пока мы переносим данные пользователи могут работеть с вьюхой, причем все, что они изменят/удалят будет отображаться в наших таблицах, а вставка данных будет идти только в новую таблицу

#### 3.4 перенос данных:
##### Результат:
- напишем процедуру для порционнго преноса данных из t3_def в t1
```
CREATE OR REPLACE PROCEDURE move3(cntlim integer default 500) AS $$
    declare
    WCURS1 CURSOR (maxcount integer) FOR SELECT * FROM t3_def limit maxcount;
    BEGIN
	<<prcess1>>
	FOR CREC IN WCURS1(maxcount:=cntlim) LOOP
            DELETE FROM t3_def WHERE (ticket_no = CREC.ticket_no)and(flight_id = CREC.flight_id);
	    INSERT INTO t1 VALUES(CREC.ticket_no,CREC.flight_id,CREC.boarding_no,CREC.seat_no);
	END LOOP prcess1;
	commit;
    END;
$$ LANGUAGE plpgsql;

```

- теперь подберем значение cntlim, которое позволит нам не создавая большой нагрузки на БД перемещать попции данных
- эксперементальным путем установили что это 1000 строк в одном commit
- и запустим цикл вызовов процедуры перемещения:
```
do $$
declare
    i int4;
begin
    for i in 0..8000
    loop
	call move3(1000);
    end loop;
end;
$$;

```

- Проверим что данные переместились:
```
demo=# select count(*) from t3_def;
 count
-------
     0
(1 row)

demo=# select count(*) from t1;
  count
---------
 7925812
(1 row)
```

- Распределение прошло также успешно (см размеры таблиц-секций):
```
                                                        List of relations
  Schema  |      Name      |       Type        |  Owner   | Persistence | Access method |    Size    |        Description
----------+----------------+-------------------+----------+-------------+---------------+------------+---------------------------
 bookings | aircrafts_data | table             | postgres | permanent   | heap          | 16 kB      | Aircrafts (internal data)
 bookings | airports_data  | table             | postgres | permanent   | heap          | 56 kB      | Airports (internal data)
 bookings | audit          | table             | postgres | permanent   | heap          | 8192 bytes |
 bookings | bookings       | table             | postgres | permanent   | heap          | 105 MB     | Bookings
 bookings | flights        | table             | postgres | permanent   | heap          | 21 MB      | Flights
 bookings | seats          | table             | postgres | permanent   | heap          | 96 kB      | Seats
 bookings | t1             | partitioned table | postgres | permanent   |               | 0 bytes    |
 bookings | t1_p0          | table             | postgres | permanent   | heap          | 3784 kB    |
 bookings | t1_p1          | table             | postgres | permanent   | heap          | 4224 kB    |
 bookings | t1_p10         | table             | postgres | permanent   | heap          | 3648 kB    |
 bookings | t1_p100        | table             | postgres | permanent   | heap          | 3816 kB    |
 bookings | t1_p101        | table             | postgres | permanent   | heap          | 4048 kB    |
 bookings | t1_p102        | table             | postgres | permanent   | heap          | 3800 kB    |
 bookings | t1_p103        | table             | postgres | permanent   | heap          | 4072 kB    |
 bookings | t1_p104        | table             | postgres | permanent   | heap          | 3880 kB    |
 bookings | t1_p105        | table             | postgres | permanent   | heap          | 3904 kB    |
 bookings | t1_p106        | table             | postgres | permanent   | heap          | 3848 kB    |
 bookings | t1_p107        | table             | postgres | permanent   | heap          | 3952 kB    |
 bookings | t1_p108        | table             | postgres | permanent   | heap          | 3928 kB    |
 bookings | t1_p109        | table             | postgres | permanent   | heap          | 3584 kB    |
 bookings | t1_p11         | table             | postgres | permanent   | heap          | 4104 kB    |
 bookings | t1_p110        | table             | postgres | permanent   | heap          | 4056 kB    |
 bookings | t1_p111        | table             | postgres | permanent   | heap          | 4160 kB    |
 bookings | t1_p112        | table             | postgres | permanent   | heap          | 3936 kB    |
 bookings | t1_p113        | table             | postgres | permanent   | heap          | 3840 kB    |
 bookings | t1_p114        | table             | postgres | permanent   | heap          | 3784 kB    |
 bookings | t1_p115        | table             | postgres | permanent   | heap          | 3848 kB    |
 bookings | t1_p116        | table             | postgres | permanent   | heap          | 3960 kB    |
 bookings | t1_p117        | table             | postgres | permanent   | heap          | 4024 kB    |
 bookings | t1_p118        | table             | postgres | permanent   | heap          | 3960 kB    |
 bookings | t1_p119        | table             | postgres | permanent   | heap          | 4040 kB    |
 bookings | t1_p12         | table             | postgres | permanent   | heap          | 4016 kB    |
 bookings | t1_p13         | table             | postgres | permanent   | heap          | 3720 kB    |
 bookings | t1_p14         | table             | postgres | permanent   | heap          | 4064 kB    |
 bookings | t1_p15         | table             | postgres | permanent   | heap          | 3880 kB    |
 bookings | t1_p16         | table             | postgres | permanent   | heap          | 3928 kB    |
 bookings | t1_p17         | table             | postgres | permanent   | heap          | 3776 kB    |
 bookings | t1_p18         | table             | postgres | permanent   | heap          | 4112 kB    |
 bookings | t1_p19         | table             | postgres | permanent   | heap          | 3688 kB    |
 bookings | t1_p2          | table             | postgres | permanent   | heap          | 4040 kB    |
 bookings | t1_p20         | table             | postgres | permanent   | heap          | 4008 kB    |
 bookings | t1_p21         | table             | postgres | permanent   | heap          | 3968 kB    |
 bookings | t1_p22         | table             | postgres | permanent   | heap          | 3768 kB    |
 bookings | t1_p23         | table             | postgres | permanent   | heap          | 3992 kB    |
 bookings | t1_p24         | table             | postgres | permanent   | heap          | 4080 kB    |
 bookings | t1_p25         | table             | postgres | permanent   | heap          | 3880 kB    |
 bookings | t1_p26         | table             | postgres | permanent   | heap          | 3816 kB    |

```
- на все про все ушло около 30 минут.

#### 3.5 Подмена2: убираем view и переименовываем нашу таблицу t1 в boarding_passes
##### Результат:
- убираем view и переименовываем нашу таблицу t1 в boarding_passes
```
demo=# drop view boarding_passes ;
DROP VIEW
demo=# alter table t1 rename to boarding_passes;
ALTER TABLE
```

- Посмотрим что получилось в результате:
```
demo=# \dt+ boarding_passes;
                                                List of relations
  Schema  |      Name       |       Type        |  Owner   | Persistence | Access method |  Size   | Description
----------+-----------------+-------------------+----------+-------------+---------------+---------+-------------
 bookings | boarding_passes | partitioned table | postgres | permanent   |               | 0 bytes |
(1 row)

demo=# \d+ boarding_passes;
                                            Partitioned table "bookings.boarding_passes"
   Column    |         Type         | Collation | Nullable | Default | Storage  | Compression | Stats target |     Description
-------------+----------------------+-----------+----------+---------+----------+-------------+--------------+----------------------
 ticket_no   | character(13)        |           | not null |         | extended |             |              | Ticket number
 flight_id   | integer              |           | not null |         | plain    |             |              | Flight ID
 boarding_no | integer              |           | not null |         | plain    |             |              | Boarding pass number
 seat_no     | character varying(4) |           | not null |         | extended |             |              | Seat number
Partition key: HASH (flight_id)
Indexes:
    "t1_pkey" PRIMARY KEY, btree (ticket_no, flight_id)
    "t1_flight_id_boarding_no_key" UNIQUE CONSTRAINT, btree (flight_id, boarding_no)
    "t1_flight_id_seat_no_key" UNIQUE CONSTRAINT, btree (flight_id, seat_no)
Partitions: t1_p0 FOR VALUES WITH (modulus 120, remainder 0),
            t1_p1 FOR VALUES WITH (modulus 120, remainder 1),
            t1_p10 FOR VALUES WITH (modulus 120, remainder 10),
            t1_p100 FOR VALUES WITH (modulus 120, remainder 100),
            t1_p101 FOR VALUES WITH (modulus 120, remainder 101),
            t1_p102 FOR VALUES WITH (modulus 120, remainder 102),
            t1_p103 FOR VALUES WITH (modulus 120, remainder 103),
            t1_p104 FOR VALUES WITH (modulus 120, remainder 104),
            t1_p105 FOR VALUES WITH (modulus 120, remainder 105),
            t1_p106 FOR VALUES WITH (modulus 120, remainder 106),
            t1_p107 FOR VALUES WITH (modulus 120, remainder 107),
            t1_p108 FOR VALUES WITH (modulus 120, remainder 108),
            t1_p109 FOR VALUES WITH (modulus 120, remainder 109),
            t1_p11 FOR VALUES WITH (modulus 120, remainder 11),
            t1_p110 FOR VALUES WITH (modulus 120, remainder 110),
            t1_p111 FOR VALUES WITH (modulus 120, remainder 111),

```
#### 3.6 Удаление старой пустой таблицы
##### Результат:
- Старая таблица, хоть и является "пустой" продолжает заниметь на диске, "сюрприз" :
```
demo=# \dt+ t3_def
                                       List of relations
  Schema  |  Name  | Type  |  Owner   | Persistence | Access method |  Size  |   Description
----------+--------+-------+----------+-------------+---------------+--------+-----------------
 bookings | t3_def | table | postgres | permanent   | heap          | 455 MB | Boarding passes
(1 row)

```
- поэтому её удаляем
```
demo=# drop table t3_def;
DROP TABLE
```

- Если бы она была нужна то пришлось бы делать ей vacuum full но к счастью она уже не нужна

#### 4.1 Замечания. Автоинкремент:

- Следует отметить, что если бы в таблице были автоинкрементные поля, то пришлось бы дополнительно пори создании t1 переопределить последовательность иначе мы могли бы получить нарушение последовательности нумерации автоинкремента (вплоть до разрушения целостности данных)
- Сделать это можно по аналогии с:
```
lesson19=# \d tdt
                            Table "public.tdt"
 Column |  Type  | Collation | Nullable |             Default
--------+--------+-----------+----------+---------------------------------
 id     | bigint |           | not null | nextval('tdt_id_seq'::regclass)
 tx     | text   |           |          |

lesson19=# alter table tdt alter column id drop default;
ALTER TABLE
lesson19=# \d+ tdt
                                           Table "public.tdt"
 Column |  Type  | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
--------+--------+-----------+----------+---------+----------+-------------+--------------+-------------
 id     | bigint |           | not null |         | plain    |             |              |
 tx     | text   |           |          |         | extended |             |              |
Access method: heap


lesson19=# alter table tdt alter column id set default nextval('tdt_id_seq'::regclass);
ALTER TABLE

lesson19=# \d tdt
                            Table "public.tdt"
 Column |  Type  | Collation | Nullable |             Default
--------+--------+-----------+----------+---------------------------------
 id     | bigint |           | not null | nextval('tdt_id_seq'::regclass)
 tx     | text   |           |          |

```
- но изменив имя последовательности на реальное из подменяемого отношения.

#### 4.1 Замечания. А что если через наследование?:
- Так как здесь нет проблемы унаследовать партиции от самой таблицы, то их можно просто создать, и затем переместить данные в дочерние таблицы по аналогии с нашим случаем, но пришлось бы написать триггер для самой таблицы, а не для вьюхи.
- Задо здесь не получилось бы просто взять и дропнуть таблицу, пришлось бы её вакуумировать. (ну если бы мы, конечно, не создали вторую табличку)

#### 4.2 Замечания. А что если бы были хранимые процедуры?:
- Выгрузить текст в файл,
- Грепнуть на наличие прямых упоминаний нашей таблицы и если таковые найдены, то разобраться, что они делают, в лучшем случае заменить имя таблицы (прийдется делать 2 раза) в худшем анализировать и менять логику.

#### 4.2 Замечания. А что если бы были триггеры?:
- Выгрузить текст в файл,
- Разобраться что они делают, в лучшем случае ничего не надо будет менять, в худшем анализировать и менять логику, да, еще прийдется наверняка на нашу вьюху триггер переписывать раз там надо что-то еще делать с таблицей .

#### 4.2 Замечания. А что если бы были view?:
- Проанализировать запросы во вьюхах, понять можно ли через них обновить данные, выяснить далают ли это реально (если возможно) 
- и далее либо расслабиться либо писать/переписать триггеры на вьюхи/таблицу

#### 4.5 Замечания. Следим за foreign key и другими ограничениями 
- при создании секционированных таблиц можно не заметить такой ключик, и тогда прийдется думать, что же с ним делать или переделывать все или создавать после, а это пахнет блокировкой таблицы
