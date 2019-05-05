#Tranform and filtering


## Sqoop import use boundary query
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --append \
  --num-mappers 6
  --boundary-query 'select min(order_item_id), max(order_item_id) from order_items where order_item_id > 99999'
```

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --append \
  --num-mappers 6
  --boundary-query 'select 100000, 172190'
```

## Column/Query Import
table and/or columns is mutually exclusive with query

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --columns order_item_order_id, order_item_id, order_item_subtotal \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --append \
  --num-mappers 2
```

~ERROR tool.BaseSqoopTool: Error parsing arguments for import~
No space between columns


```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --columns order_item_order_id,order_item_id,order_item_subtotal \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --append \
  --num-mappers 2
```

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db\
  --num-mappers 2 \
  --query "select o.*, sum(oi.order_item_subtotal) order_revenue from orders o join order_items oi on o.order_id = oi.order_item_order_id group by o.order_id, o.order_date, o.order_customer_id, o.order_status" \
  --split-by order_id
```

~Must specify destination with --target-dir~
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db/orders_with_revenue \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders_with_revenue/ \
  --num-mappers 2 \
  --query "select o.*, sum(oi.order_item_subtotal) order_revenue from orders o join order_items oi on o.order_id = oi.order_item_order_id group by o.order_id, o.order_date, o.order_customer_id, o.order_status" \
  --split-by order_id
```

ERROR tool.ImportTool: Encountered IOException running import job: java.io.IOException: Query [select o.*, sum(oi.order_item_subtotal) order_revenue from orders o join order_items oi on o.order_id = oi.order_item_order_id group by o.order_id, o.order_date, o.order_customer_id, o.order_status] must contain '$CONDITIONS' in WHERE clause.


```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders_with_revenue/ \
  --num-mappers 2 \
  --query "select o.*, sum(oi.order_item_subtotal) order_revenue from orders o join order_items oi on o.order_id = oi.order_item_order_id AND $CONDITIONS group by o.order_id, o.order_date, o.order_customer_id, o.order_status" \
  --split-by order_id
```

$CONDITIONS should be escaped

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --target-dir /user/jaszhou/sqoop_import/retail_db/orders_with_revenue/ \
  --num-mappers 2 \
  --query "select o.*, sum(oi.order_item_subtotal) order_revenue from orders o join order_items oi on o.order_id = oi.order_item_order_id AND \$CONDITIONS group by o.order_id, o.order_date, o.order_customer_id, o.order_status" \
  --split-by order_id
```

Notes:
1. table/column is mutually exclusive to Query
2. query should specify split-by
3. query should have placeholder \$CONDITIONS
  tables
