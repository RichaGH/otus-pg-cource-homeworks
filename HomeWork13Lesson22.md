# Домашнее задание
## Триггеры, поддержка заполнения витрин

### Цель:
Создать триггер для поддержки витрины в актуальном состоянии.

#### 1. Задание:
В БД создана структура, описывающая товары (таблица goods) и продажи (таблица sales). Есть запрос для генерации отчета – сумма продаж по каждому товару. БД была денормализована, создана таблица (витрина), структура которой повторяет структуру отчета.
Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину) Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE
#### Вопрос: Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)? 
##### Ответ:
- Витрина не прявязана напрямую к состоянию БД и таким, образом, может существовать отдельно от данных (можно сделать с неё срез "на память" и использовать его потом в каких либо целях) При этом срез витрины значительно легче среза БД, уже хранит агрегированные данные, его можно использовать для вравнения с другими "историческими данными"
#### Подсказка: В реальной жизни возможны изменения цен.
- Содержимое файла с заданием:

```
-- ДЗ тема: триггеры, поддержка заполнения витрин

DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;

SET search_path = pract_functions, publ

-- товары:
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);

-- Продажи
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);

-- отчет:
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу
CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);

-- Создать триггер (на таблице sales) для поддержки.
-- Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE

-- Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
-- Подсказка: В реальной жизни возможны изменения цен.


```

#### 2.0 Выполниние задания. Анализ задачи:
##### Результат:
- Итак к нам пришел "манагер" и говорит "хочу вот так". 
- Воспользуемся подсказкой "В реальной жизни возможны изменения цен." и немного доработаем это напильником:
- Понимаем, что "так" работать не будет так как нет исторических данных, ключем в таблице-витрине является строка, в общем все ужасно, поэтому поступим так:

#### 2.1 Выполниние задания. Немного изменим наши входные данные:
- Нужны исторические данные по изменениям цен. Без них не получится делать полноценные обновления витрины и тем более удаления. 
```
create table hist_goods_tseny (
id bigserial primary key,
set_date_time timestamp,
tsena numeric(12,2),
goods_id integer REFERENCES goods (goods_id)
);
CREATE TABLE

```

- дополним "витрину" еще одним полем, которое нам поможет еще не раз:
```
alter table good_sum_mart add column goods_id integer;
alter table good_sum_mart add constraint goods_fk foreign key (goods_id) references goods(goods_id);

```
- ну вот теперь можно работать:

#### 2.2 Выполниние задания. Создадим триггеры для sales:
- результаты выполнения команд по созданию функций и триггеров:
```
lesson22=# CREATE OR REPLACE FUNCTION delete_sales() RETURNS TRIGGER AS $$
lesson22$#     declare
lesson22$# l_sum_sale numeric(16,2):=0;
lesson22$#     BEGIN
lesson22$#    select sum_sale into l_sum_sale from good_sum_mart where good_sum_mart.goods_id=OLD.good_id limit 1;
lesson22$#     if ((l_sum_sale - ((OLD.sales_qty)*(select tsena from hist_goods_tseny where goods_id=OLD.good_id and set_date_time<OLD.sales_time order by set_date_time desc limit 1)))=0) then
lesson22$#

lesson22$# delete from good_sum_mart gsm where gsm.goods_id=OLD.good_id;
lesson22$#    else
           else
                insert into good_sum_mart (good_name, sum_sale, goods_id) values ((select g.good_name from goods g where g.goods_id=NEW.good_id),((NEW.sales_qty)*(select tsena from hist_goods_tseny hgt where hgt.goods_id=NEW.good_id and hgt.set_date_time<NEW.sales_time order by hgt.set_date_time desc limit 1)),NEW.good_id);
           end if;
        END IF;
        RETURN NEW;
    END;
$$ LANGUAGE plpgsql;
lesson22$#

lesson22$# update good_sum_mart set sum_sale=sum_sale - (OLD.sales_qty)*(select tsena from hist_goods_tseny where goods_id=OLD.good_id and set_date_time<OLD.sales_time order by set_date_time desc limit 1) where good_sum_mart.goods_id=OLD.good_id;
lesson22$#    end if;
lesson22$# RETURN NEW;
lesson22$#     END;
lesson22$# $$ LANGUAGE plpgsql;
CREATE FUNCTION
lesson22=#
lesson22=#
lesson22=# CREATE OR REPLACE FUNCTION insert_sales() RETURNS TRIGGER AS $$
lesson22$#     declare
lesson22$# isex integer:=0;
lesson22$#     BEGIN
lesson22$#    select 1 into isex from good_sum_mart where good_sum_mart.goods_id=NEW.good_id limit 1;
lesson22$#     if isex=1 then
lesson22$#

lesson22$# update good_sum_mart set sum_sale=sum_sale+(NEW.sales_qty)*(select tsena from hist_goods_tseny where goods_id=NEW.good_id and set_date_time<NEW.sales_time order by set_date_time desc limit 1) where good_sum_mart.goods_id=NEW.good_id;
lesson22$#    else
lesson22$#

lesson22$# insert into good_sum_mart (good_name, sum_sale, goods_id) values ((select g.good_name from goods g where g.goods_id=NEW.good_id),((NEW.sales_qty)*(select tsena from hist_goods_tseny hgt where hgt.goods_id=NEW.good_id and hgt.set_date_time<NEW.sales_time order by hgt.set_date_time desc limit 1)),NEW.good_id);
lesson22$#    end if;
lesson22$# RETURN NEW;
lesson22$#     END;
lesson22$# $$ LANGUAGE plpgsql;
CREATE FUNCTION
lesson22=#
lesson22=#
lesson22=# CREATE OR REPLACE FUNCTION update_sales() RETURNS TRIGGER AS $$
lesson22$#     declare
lesson22$# isex integer:=0;
lesson22$#     BEGIN
lesson22$# IF (NEW.good_id=OLD.good_id) THEN
lesson22$#    update good_sum_mart set sum_sale=sum_sale+(NEW.sales_qty)*(select tsena from hist_goods_tseny where goods_id=NEW.good_id and set_date_time<NEW.sales_time order by set_date_time desc limit 1) - (OLD.sales_qty)*(select tsena from hist_goods_tseny where goods_id=NEW.good_id and set_date_time<OLD.sales_time order by set_date_time desc limit 1) where good_sum_mart.goods_id=NEW.good_id;
lesson22$# ELSE
lesson22$#    update good_sum_mart set sum_sale=sum_sale - (OLD.sales_qty)*(select tsena from hist_goods_tseny where goods_id=OLD.good_id and set_date_time<OLD.sales_time order by set_date_time desc limit 1) where good_sum_mart.goods_id=OLD.good_id;
lesson22$#    select 1 into isex from good_sum_mart where good_sum_mart.goods_id=NEW.good_id limit 1;
lesson22$#     if isex=1 then
lesson22$#

lesson22$# update good_sum_mart set sum_sale=sum_sale+(NEW.sales_qty)*(select tsena from hist_goods_tseny where goods_id=NEW.good_id and set_date_time<NEW.sales_time order by set_date_time desc limit 1) where good_sum_mart.goods_id=NEW.good_id;
lesson22$#    else
lesson22$#

lesson22$# insert into good_sum_mart (good_name, sum_sale, goods_id) values ((select g.good_name from goods g where g.goods_id=NEW.good_id),((NEW.sales_qty)*(select tsena from hist_goods_tseny hgt where hgt.goods_id=NEW.good_id and hgt.set_date_time<NEW.sales_time order by hgt.set_date_time desc limit 1)),NEW.good_id);
lesson22$#    end if;
lesson22$# END IF;
lesson22$# RETURN NEW;
lesson22$#     END;
lesson22$# $$ LANGUAGE plpgsql;
CREATE FUNCTION

lesson22=# CREATE TRIGGER sales_af_ins
lesson22-# AFTER INSERT on sales
lesson22-# FOR EACH ROW EXECUTE PROCEDURE insert_sales();
CREATE TRIGGER
lesson22=#
lesson22=# CREATE TRIGGER sales_af_upd
lesson22-# AFTER UPDATE on sales
lesson22-# FOR EACH ROW EXECUTE PROCEDURE update_sales();
CREATE TRIGGER
lesson22=#
lesson22=# CREATE TRIGGER sales_af_del
lesson22-# AFTER DELETE on sales
lesson22-# FOR EACH ROW EXECUTE PROCEDURE delete_sales();
CREATE TRIGGER

```
- готово

#### 3 Проверка работы триггера. Подготовка тестовых данных:
##### Результаты:
- нам нужны такие исторические значения которые позволят легко подсчитать результаты и проверить их правильность:
```
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-365,0,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-365,0,2);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-300,100,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-250,200,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-200,300,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-150,400,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-100,500,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-50,600,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-5,700,1);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-350,100000000.01,2);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-300,1050000000.05,2);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-150,1550000000.05,2);
INSERT 0 1
lesson22=# insert into hist_goods_tseny (set_date_time,tsena,goods_id) values (date(now())-50,185000000.01,2);
INSERT 0 1

```

- Заметим, что я не создавал доп триггер на таблице goods для заполнения таблицы hist_goods_tseny, но в реяльности он нужен так как сейчас цена по прежнему лежит в goods (но только текущая...) и в общем случае пользователи не подозревающие о нашей новой таблице продолжили бы работать с goods, 
- позволю себе этого не делать так как для проверки работы по заполнению витрины это можно вынести за скобки

- итак, имеем следующие базовые данные:
```
lesson22=# select * from hist_goods_tseny order by set_date_time;
 id |    set_date_time    |     tsena     | goods_id
----+---------------------+---------------+----------
  1 | 2021-01-09 00:00:00 |          0.00 |        1
  2 | 2021-01-09 00:00:00 |          0.00 |        2
 10 | 2021-01-24 00:00:00 |  100000000.01 |        2
 11 | 2021-03-15 00:00:00 | 1050000000.05 |        2
  3 | 2021-03-15 00:00:00 |        100.00 |        1
  4 | 2021-05-04 00:00:00 |        200.00 |        1
  5 | 2021-06-23 00:00:00 |        300.00 |        1
  6 | 2021-08-12 00:00:00 |        400.00 |        1
 12 | 2021-08-12 00:00:00 | 1550000000.05 |        2
  7 | 2021-10-01 00:00:00 |        500.00 |        1
  8 | 2021-11-20 00:00:00 |        600.00 |        1
 13 | 2021-11-20 00:00:00 |  185000000.01 |        2
  9 | 2022-01-04 00:00:00 |        700.00 |        1
 14 | 2022-01-06 00:00:00 |       1000.00 |        1
(14 rows)

lesson22=# select * from goods;
 goods_id |        good_name         |  good_price
----------+--------------------------+--------------
        1 | Спички хозайственные     |      1000.00
        2 | Автомобиль Ferrari FXX K | 185000000.01
(2 rows)

```

- и пустые таблицы продаж и витрину

#### 3 Проверка работы триггера. Проверка:
##### Результаты:
- Добавление "продаж"

```
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-289, 10);
INSERT 0 1
lesson22=# select * from sales;
 sales_id | good_id |       sales_time       | sales_qty
----------+---------+------------------------+-----------
        3 |       1 | 2021-03-26 00:00:00+00 |        10
(1 row)

lesson22=# select * from good_sum_mart;
      good_name       | sum_sale | goods_id
----------------------+----------+----------
 Спички хозайственные |  1000.00 |        1
(1 row)

lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-249, 30);
INSERT 0 1
lesson22=# select * from good_sum_mart;
      good_name       | sum_sale | goods_id
----------------------+----------+----------
 Спички хозайственные |  7000.00 |        1
(1 row)

lesson22=# select * from sales;
 sales_id | good_id |       sales_time       | sales_qty
----------+---------+------------------------+-----------
        3 |       1 | 2021-03-26 00:00:00+00 |        10
        4 |       1 | 2021-05-05 00:00:00+00 |        30
(2 rows)

```
- товар один. строка тоже 
- сумма?
```
  3 | 2021-03-15 00:00:00 |        100.00 | * 10 =1000
  4 | 2021-05-04 00:00:00 |        200.00 | * 30 =6000
итого 7000
```
- порядок

- Обновление (даты):
```
lesson22=# update sales set(good_id,sales_time)=(1,date(now())-200) where sales_id=3;
UPDATE 1
lesson22=# select * from good_sum_mart;
      good_name       | sum_sale | goods_id
----------------------+----------+----------
 Спички хозайственные |  8000.00 |        1
(1 row)

lesson22=# select date(now())-200;
  ?column?
------------
 2021-06-23
(1 row)

lesson22=# select * from hist_goods_tseny where goods_id=1 order by set_date_time;
 id |    set_date_time    |  tsena  | goods_id
----+---------------------+---------+----------
  1 | 2021-01-09 00:00:00 |    0.00 |        1
  3 | 2021-03-15 00:00:00 |  100.00 |        1
  4 | 2021-05-04 00:00:00 |  200.00 |        1
  5 | 2021-06-23 00:00:00 |  300.00 |        1
  6 | 2021-08-12 00:00:00 |  400.00 |        1
  7 | 2021-10-01 00:00:00 |  500.00 |        1
  8 | 2021-11-20 00:00:00 |  600.00 |        1
  9 | 2022-01-04 00:00:00 |  700.00 |        1
 14 | 2022-01-06 00:00:00 | 1000.00 |        1
(9 rows)

lesson22=# select 40*200;
 ?column?
----------
     8000
(1 row)

```
- все в порядке

- проверим теперь удаление, для этого сначала добавим строки в таблицу, а затем удалим их:
```
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-149, 50);
INSERT 0 1
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-1, 50);
INSERT 0 1
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-7, 30);
INSERT 0 1
lesson22=# select * from good_sum_mart;
      good_name       | sum_sale | goods_id
----------------------+----------+----------
 Спички хозайственные | 97000.00 |        1
(1 row)

lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (2,date(now())-12, 2);
INSERT 0 1
lesson22=# select * from good_sum_mart;
        good_name         |   sum_sale   | goods_id
--------------------------+--------------+----------
 Спички хозайственные     |     97000.00 |        1
 Автомобиль Ferrari FXX K | 370000000.02 |        2
(2 rows)

lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (2,date(now())-120, 2);
INSERT 0 1
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (2,date(now())-220, 3);
INSERT 0 1
lesson22=# select * from good_sum_mart;
        good_name         |   sum_sale    | goods_id
--------------------------+---------------+----------
 Спички хозайственные     |      97000.00 |        1
 Автомобиль Ferrari FXX K | 6620000000.27 |        2
(2 rows)

lesson22=# select * from sales;
 sales_id | good_id |       sales_time       | sales_qty
----------+---------+------------------------+-----------
        4 |       1 | 2021-05-05 00:00:00+00 |        30
        3 |       1 | 2021-06-25 00:00:00+00 |        10
        5 |       1 | 2021-08-13 00:00:00+00 |        50
        6 |       1 | 2022-01-08 00:00:00+00 |        50
        7 |       1 | 2022-01-02 00:00:00+00 |        30
        8 |       2 | 2021-12-28 00:00:00+00 |         2
        9 |       2 | 2021-09-11 00:00:00+00 |         2
       10 |       2 | 2021-06-03 00:00:00+00 |         3
(8 rows)

lesson22=# delete from sales where sales_id=9;
DELETE 1
lesson22=# select * from good_sum_mart;
        good_name         |   sum_sale    | goods_id
--------------------------+---------------+----------
 Спички хозайственные     |      97000.00 |        1
 Автомобиль Ferrari FXX K | 3520000000.17 |        2
(2 rows)

lesson22=# delete from sales where sales_id=10;
DELETE 1
lesson22=# delete from sales where sales_id=8;
DELETE 1
lesson22=# select * from good_sum_mart;
      good_name       | sum_sale | goods_id
----------------------+----------+----------
 Спички хозайственные | 97000.00 |        1
(1 row)

```

- все как мы и хотели, если товар отсутствует в таблице продаж, то и из отчета строчки подтерлись. ОК

- а если удалить все строки в таблице продаж?
```
lesson22=# select * from sales;
 sales_id | good_id |       sales_time       | sales_qty
----------+---------+------------------------+-----------
        4 |       1 | 2021-05-05 00:00:00+00 |        30
        3 |       1 | 2021-06-25 00:00:00+00 |        10
        5 |       1 | 2021-08-13 00:00:00+00 |        50
        6 |       1 | 2022-01-08 00:00:00+00 |        50
        7 |       1 | 2022-01-02 00:00:00+00 |        30
(5 rows)

lesson22=# delete from sales;
DELETE 5
lesson22=# select * from sales;
 sales_id | good_id | sales_time | sales_qty
----------+---------+------------+-----------
(0 rows)

lesson22=# select * from good_sum_mart;
 good_name | sum_sale | goods_id
-----------+----------+----------
(0 rows)

```
- хм, и так работает.

- теперь более интересный вариант: что если кто-то решит обновить сразу 3 значения (и количество, и дату, и сам товар)?
```
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-149, 50);
INSERT 0 1
lesson22=# INSERT INTO sales (good_id, sales_time, sales_qty) VALUES (1,date(now())-1, 50);
INSERT 0 1
lesson22=# select * from good_sum_mart;
      good_name       | sum_sale | goods_id
----------------------+----------+----------
 Спички хозайственные | 70000.00 |        1
(1 row)

lesson22=# select * from sales;
 sales_id | good_id |       sales_time       | sales_qty
----------+---------+------------------------+-----------
       11 |       1 | 2021-08-13 00:00:00+00 |        50
       12 |       1 | 2022-01-08 00:00:00+00 |        50
(2 rows)

lesson22=# update sales set(good_id,sales_time,sales_qty)=(2,date(now())-299,2) where sales_id=12;
UPDATE 1
lesson22=# select * from sales;
 sales_id | good_id |       sales_time       | sales_qty
----------+---------+------------------------+-----------
       11 |       1 | 2021-08-13 00:00:00+00 |        50
       12 |       2 | 2021-03-16 00:00:00+00 |         2
(2 rows)

lesson22=# select * from good_sum_mart;
        good_name         |   sum_sale    | goods_id
--------------------------+---------------+----------
 Спички хозайственные     |      20000.00 |        1
 Автомобиль Ferrari FXX K | 2100000000.10 |        2
(2 rows)

lesson22=# select * from hist_goods_tseny where goods_id=2 order by set_date_time;
 id |    set_date_time    |     tsena     | goods_id
----+---------------------+---------------+----------
  2 | 2021-01-09 00:00:00 |          0.00 |        2
 10 | 2021-01-24 00:00:00 |  100000000.01 |        2
 11 | 2021-03-15 00:00:00 | 1050000000.05 |        2
 12 | 2021-08-12 00:00:00 | 1550000000.05 |        2
 13 | 2021-11-20 00:00:00 |  185000000.01 |        2
(5 rows)

lesson22=# select 1050000000.05*2;
   ?column?
---------------
 2100000000.10
(1 row)

lesson22=# select 50*400;
 ?column?
----------
    20000
(1 row)

```
- Кажется работает.

- Почему кажется? Потому-что мы просто "проверили" несколько ситуаций, а не привели здесь четких математических доказательств

#### Результат:
- Триггеры созданы.
- Первоначальное простое тестирование показало работоспособность решения.
- (Очевидно, что это не пром решение, но здесь и сами таблицы совсем не пром)
