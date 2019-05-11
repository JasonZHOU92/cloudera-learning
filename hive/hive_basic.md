# Hive Basic

## Create Database and tables
```
[root@quickstart ~]# hive
hive> create database jaszhou_retail_db_txt;
hive> use jaszhou_retail_db_txt;
hive> show tables;
hive> set hive.metastore.warehouse.dir;
hive.metastore.warehouse.dir=/user/hive/warehouse
hive> dfs -ls /user/hive/warehouse;
drwxrwxrwx   - root supergroup          0 2019-05-08 16:40 /user/hive/warehouse/jaszhou_retail_db_txt.db
drwxrwxrwx   - root supergroup          0 2019-05-05 08:18 /user/hive/warehouse/jaszhou_sqoop_import.db
```

create database jaszhou_retail_db_txt;

use jaszhou_retail_db_txt;

```
create table orders (
    order_id int,
    order_date string,
    order_customer_id int,
    order_status string
) row format delimited fields terminated by ',';
```
load data local inpath '/public/data/retail_db/orders' into table orders;


## Load data into table
```
create table order_items (
    order_item_id int,
    order_item_order_in int,
    order_item_product_id int,
    order_item_quantity int,
    order_item_subtotal float,
    order_item_product_price float
) row format delimited fields terminated by ','
stored as textfile;
```

load data local inpath '/public/data/retail_db/order_items' into table order_items;

select * from order_items limit 10;


## Create tables in orc
```
create table orders (
    order_id int,
    order_date string,
    order_customer_id int,
    order_status string
) stored as orc;
```

```
create table order_items (
    order_item_id int,
    order_item_order_in int,
    order_item_product_id int,
    order_item_quantity int,
    order_item_subtotal float,
    order_item_product_price float
) stored as orc;
```

desc formatted order_items;

insert into table orders select * from jaszhou_retail_db_txt.orders;
insert into table order_items select * from jaszhou_retail_db_txt.order_items;


spark-shell --master yarn --conf spark.ui.port=12345

## Run hive in spark-shell
```
scala > sc
scala > sqlContext


sqlContext.sql("use jaszhou_retail_db_txt")
sqlContext.sql("show tables").show
sqlContext.sql("select * from orders limit 10").show
```
