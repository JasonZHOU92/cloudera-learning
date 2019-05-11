
# Sqoop hive Import

## create a hive database
You need to create a hive database firstly
[root@quickstart ~]# hive
hive> create database jaszhou_sqoop_import;


## Basic Hive import

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --target-dir /user/jaszhou/sqoop_import/retail_db
```

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --hive-import \
  --hive-database jaszhou_sqoop_import \
  --hive-table order_items \
  --num-mappers 2
```

data will be firstly import into hdfs /user/${username}/, then copied into /user/hive/warehouse/jaszhou_sqoop_import.db


To figure out details and storage information of the table do
`hive> describe formatted order_items;`


## --hive-override override existing table
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --hive-import \
  --hive-database jaszhou_sqoop_import \
  --hive-table order_items \
  --hive-override \
  --num-mappers 2
```

don't use `--create-hive-table`, it is highly confusing

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --hive-import \
  --hive-database jaszhou_sqoop_import \
  --hive-table order_items \
  --create-hive-table \
  --num-mappers 2
```

FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. AlreadyExistsException(message:Table order_items already exists)

However you can find the imported data at /user/${username}/order_items

## Import all-tables
Limitations
- --warehouse-dir is mandatory
- Better to use autoreset-to-one-mapper
- cannot specify many arguments such as --query, --cols, --where which does filtering or transformations on the data
- incremental import not possible

```
sqoop import-all-tables \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --warehouse-dir /user/root/sqoop_import/retail_db \
  --autoreset-to-one-mapper  
```

You cannot also use sqoop import-all-table to import data to hive. However you need to
1. remove existing tables in hive, because you can not use --hive-override
2. remove existing files in --warehouse-dir, because you cannot use --append or --delete-target-dir
3. the hive import mechanism still holds, date will first be imported `warehouse-dir`, if hive import succeed, then copy to `hive-home`; otherwise stays at `warehouse-dir`


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
