# Sqoop export

## Set up
Import all tables into hive
```
sqoop import-all-tables \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --warehouse-dir /user/root/sqoop_import/retail_db \
  --autoreset-to-one-mapper \
  --hive-import \
  --hive-database jaszhou_sqoop_import \
  --delete-target-dir
```

`Location:           	hdfs://quickstart.cloudera:8020/user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue`

hive> create table daily_revenue as
select order_date, sum(order_item_subtotal) daily_revenue
from orders join order_items on
order_id = order_item_order_id
where order_date like '2013-07%'
group by order_date

mysql>create table daily_revenue(order_date varchar(30), revenue float);


## Simple export
```
Input parsing arguments
--input-enclosed-by <char>	Sets a required field encloser
--input-escaped-by <char>	Sets the input escape character
--input-fields-terminated-by <char>	Sets the input field separator
--input-lines-terminated-by <char>	Sets the input end-of-line character
--input-optionally-enclosed-by <char>	Sets a field enclosing character

Output line formatted arguments
--enclosed-by <char>	Sets a required field enclosing character
--escaped-by <char>	Sets the escape character
--fields-terminated-by <char>	Sets the field separator character
--lines-terminated-by <char>	Sets the end-of-line character
--mysql-delimiters	Uses MySQL’s default delimiter set: fields: , lines: \n escaped-by: \ optionally-enclosed-by: '
--optionally-enclosed-by <char>	Sets a field enclosing character
```
```
sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue \
  --input-fields-terminated-by "\001"
```
## Column export
`mysql>create table daily_revenue_demo(order_date varchar(30), revenue float, description varchar(200));``
```
sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue_demo \
  --input-fields-terminated-by "\001"
```

  `ERROR tool.ExportTool: Error during export: Export job failed!`

```
sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue_demo \
  --columns order_date,revenue \
  --input-fields-terminated-by "\001" \
  --num-mappers 1
```

the missing field should be nullable, try with
`mysql>create table daily_revenue_demo(order_date varchar(30), revenue float, description varchar(200) not null);``

## Insert and Update
```
hive> create table daily_revenue as
select order_date, sum(order_item_subtotal) daily_revenue
from orders join order_items on
order_id = order_item_order_id
where order_date like '2013-07%'
group by order_date;
```

```
hive> insert into table daily_revenue  
select order_date, sum(order_item_subtotal) daily_revenue
from orders join order_items on
order_id = order_item_order_id
where order_date like '2013-08%'
group by order_date;
```

```
mysql> update daily_revenue set revenue = 0;

mysql> select * from daily_revenue;
+-----------------------+---------+
| order_date            | revenue |
+-----------------------+---------+
| 2013-07-25 00:00:00.0 |       0 |
| 2013-07-26 00:00:00.0 |       0 |
| 2013-07-27 00:00:00.0 |       0 |
| 2013-07-28 00:00:00.0 |       0 |
| 2013-07-29 00:00:00.0 |       0 |
| 2013-07-30 00:00:00.0 |       0 |
| 2013-07-31 00:00:00.0 |       0 |
+-----------------------+---------+
```

Note:  --update-key order_date

```
sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue \
  --update-key order_date \
  --input-fields-terminated-by "\001" \
  --num-mappers 1


mysql> select * from daily_revenue;
+-----------------------+---------+
| order_date            | revenue |
+-----------------------+---------+
| 2013-07-25 00:00:00.0 | 68153.8 |
| 2013-07-26 00:00:00.0 |  136520 |
| 2013-07-27 00:00:00.0 |  101074 |
| 2013-07-28 00:00:00.0 | 87123.1 |
| 2013-07-29 00:00:00.0 |  137287 |
| 2013-07-30 00:00:00.0 |  102746 |
| 2013-07-31 00:00:00.0 |  131878 |
+-----------------------+---------+
7 rows in set (0.00 sec)
```

Note:   --update-mode allowinsert 

```
sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue \
  --update-key order_date \
  --update-mode allowinsert \
  --input-fields-terminated-by "\001" \
  --num-mappers 1


mysql> select * from daily_revenue;
+-----------------------+---------+
| order_date            | revenue |
+-----------------------+---------+
| 2013-07-25 00:00:00.0 | 68153.8 |
| 2013-07-26 00:00:00.0 |  136520 |
| 2013-07-27 00:00:00.0 |  101074 |
| 2013-07-28 00:00:00.0 | 87123.1 |
| 2013-07-29 00:00:00.0 |  137287 |
| 2013-07-30 00:00:00.0 |  102746 |
| 2013-07-31 00:00:00.0 |  131878 |
| 2013-07-25 00:00:00.0 | 68153.8 |
| 2013-07-26 00:00:00.0 |  136520 |
| 2013-07-27 00:00:00.0 |  101074 |
| 2013-07-28 00:00:00.0 | 87123.1 |
| 2013-07-29 00:00:00.0 |  137287 |
| 2013-07-30 00:00:00.0 |  102746 |
| 2013-07-31 00:00:00.0 |  131878 |
| 2013-08-01 00:00:00.0 |  129002 |
| 2013-08-02 00:00:00.0 |  109347 |
| 2013-08-03 00:00:00.0 | 95266.9 |
| 2013-08-04 00:00:00.0 |   90931 |
| 2013-08-05 00:00:00.0 | 75882.3 |
| 2013-08-06 00:00:00.0 |  120574 |
| 2013-08-07 00:00:00.0 |  103351 |
| 2013-08-08 00:00:00.0 | 76501.7 |
| 2013-08-09 00:00:00.0 | 62316.5 |
| 2013-08-10 00:00:00.0 |  129575 |
| 2013-08-11 00:00:00.0 | 71149.6 |
| 2013-08-12 00:00:00.0 |  121883 |
| 2013-08-13 00:00:00.0 | 39874.5 |
| 2013-08-14 00:00:00.0 |  103939 |
| 2013-08-15 00:00:00.0 |  103031 |
| 2013-08-16 00:00:00.0 | 69264.5 |
| 2013-08-17 00:00:00.0 |  127362 |
| 2013-08-18 00:00:00.0 |  106789 |
| 2013-08-19 00:00:00.0 | 50348.3 |
| 2013-08-20 00:00:00.0 | 87983.1 |
| 2013-08-21 00:00:00.0 | 64860.8 |
| 2013-08-22 00:00:00.0 | 94574.7 |
| 2013-08-23 00:00:00.0 | 99616.2 |
| 2013-08-24 00:00:00.0 |  128884 |
| 2013-08-25 00:00:00.0 | 98521.6 |
| 2013-08-26 00:00:00.0 | 88114.2 |
| 2013-08-27 00:00:00.0 | 91634.9 |
| 2013-08-28 00:00:00.0 | 55189.2 |
| 2013-08-29 00:00:00.0 | 99960.6 |
| 2013-08-30 00:00:00.0 | 57008.5 |
| 2013-08-31 00:00:00.0 | 75923.7 |
+-----------------------+---------+
```

## Staging table

hive> insert into table daily_revenue  
select order_date, sum(order_item_subtotal) daily_revenue
from orders join order_items on
order_id = order_item_order_id
where order_date > '2013-07-01'
group by order_date;

mysql>create table daily_revenue_stage(order_date varchar(30), revenue float);

sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue \
  --input-fields-terminated-by "\001" \
  --num-mappers 1

When upserting process fails, you might end with inserting partial data into target table. If you use staging table, you can have all or nothing guarantee

sqoop export \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --export-dir /user/hive/warehouse/jaszhou_sqoop_import.db/daily_revenue \
  --table daily_revenue \
  --staging-table daily_revenue_stage \
  --input-fields-terminated-by "\001" \
  --num-mappers 1



  hive table -> mysql staging table -> mysql table
