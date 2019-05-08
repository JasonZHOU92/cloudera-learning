#Delimiter and null string value


## --null-string and --null-non-string

The --null-string and --null-non-string arguments are optional.\ If not specified, then the string "null" will be used.

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/hr \
  --username root \
  --password cloudera \
  --warehouse-dir /user/jaszhou/sqoop_import/hr_db/ \
  --table employees \
  --append \
  --null-non-string -1
```

## Delimiter can be specified in column and line level
default Delimiter is comma
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/hr \
  --username root \
  --password cloudera \
  --warehouse-dir /user/jaszhou/sqoop_import/hr_db/ \
  --table employees
  --delete-target-dir
```

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/hr \
  --username root \
  --password cloudera \
  --warehouse-dir /user/jaszhou/sqoop_import/hr_db/ \
  --table employees \
  --append \
  --null-non-string -1 \
  --fields-terminated-by "\t" \
  --lines-terminated-by ":"
```

sqoop delimiter can use \000
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/hr \
  --username root \
  --password cloudera \
  --warehouse-dir /user/jaszhou/sqoop_import/hr_db/ \
  --table employees \
  --append \
  --null-non-string -1 \
  --fields-terminated-by "\000" \
  --lines-terminated-by ":"
```

## Incremental Load
you should use append mode

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders \
  --query "select * from orders where order_date_like '2013-%'" \
  --split-by order_id \
  --append
```


```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders \
  --query "select * from orders where \$CONDITIONS AND order_date like '2013-%'" \
  --split-by order_id \
  --append
```

`ERROR tool.ImportTool: Encountered IOException running import job: java.io.IOException: No columns to generate for ClassWriter`

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders \
  --query "select * from orders where \$CONDITIONS AND order_date like '2014-01%'" \
  --split-by order_id \
  --append
```

//incremental can be either append or lastmodified
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders \
  --table orders \
  --check-column order_date \
  --incremental append \
  --last-value '2014-02-28'
```
