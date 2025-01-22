###  Пример загрузки данных из CSV файла

- в данном случае таблица Чикагского Такси ( данные можно взять в интернете )<br>
- за 11 лет с 2013 по 2023 год там накопилось более 200 млн записей<br>
- Можно использовать для тестов больших данных, шардирования и пр.

- Для загрузки в СУБД PostgreSQL

- на всякий случай установим формат таты как в табицах csv
```
set datestyle to sql,mdy;
```

- создадим базу 
```
CREATE DATABASE taxi;
  -- подключимся к базе
\c taxi
  -- создадим таблицу
CREATE TABLE trips (
	trip_id                varchar(50) NOT NULL,
	taxi_id                varchar(128) NOT NULL,
	trip_start_timestamp   timestamp NOT NULL,
	trip_end_timestamp     timestamp NULL,
	trip_seconds           int4 NULL,
	trip_miles             float4 NULL,
	pickup_census_tract    int8 NULL,
	dropoff_census_tract   int8 NULL,
	pickup_community_area  int4 NULL,
	dropoff_community_area int4 NULL,
	fare                   float4 NULL,
	tips                   float4 NULL,
	tolls                  float4 NULL,
	extras                 float4 NULL,
	trip_total             float4 NULL,
	payment_type           varchar(50) NULL,
	company                varchar(50) NULL,
	pickup_centroid_latitude    float4 NULL,
	pickup_centroid_longitude   float4 NULL,
	pickup_centroid_location    varchar(50) NULL,
	dropoff_centroid_latitude   float4 NULL,
	dropoff_centroid_longitude  float4 NULL,
	dropoff_centroid_location   varchar(50) NULL
);
  -- пример загрузки в psql
copy trips from '/tmp/test.csv' delimiter ',' csv header;
```

-- при загрузке с помощью pgAdmin или DBeaver надо делать соответствие полей

-- если делать загрузку например в DBeaver с созданием полей,<br>
-- то поля будут с именами из заголовка файла CSV<br>
-- и типы полей сложно подобрать, и потом надо будет исправлять

-- поля *_location в CSV представлены "POINT ( xx.xxxxx yy.yyyyy )"<br>
-- не удалось преобразовать в тип point<br>
-- оставил строковые значения, отдельно есть latitude и longitude с теми-же значениями

-- возможно сразу можно индексы создать у таблицы<br>
-- trip_id - уникальный, первичным ключом может быть


-- количество записей по годам<br>
-- 2023 --  6495570<br>  
--      в базе данных занимает 3.2 GB<br>
--      загружается 3 min командой copy<br>
--      ВМ ( vCPU-2 vRAM-2Gb ssd )<br>
--<br>
-- 2022 --  6382425<br>
-- 2021 --  3948045<br>
-- 2020 --  3889032<br>
-- 2019 -- 16477365<br>
-- 2018 -- 20732088<br>
-- 2017 -- 24988003<br>
-- 2016 -- 31759339<br>
-- 2015 -- 32385875<br>
-- 2014 -- 37395436<br>
-- 2013 -- 27217716



