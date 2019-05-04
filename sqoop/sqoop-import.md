# Sqoop user guide

## Sqoop Basic
//default port for mysql is 3306
```
sqoop list-databases \
  --connect jdbc:mysql://localhost:3306 \
  --username root \
  --password cloudera
```

```
sqoop list-tables \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera
```

```
sqoop eval \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --query "Select * from orders limit 10"
```

```
sqoop eval \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --query "Insert into orders values (100000, "2017-10-31 00:00:00.0", 100000, "DUMMY")"
```

## Sqoop import a table
// `target-dir` is the target dir of imported data, data ends in /user/jaszhou/sqoop_import/retail_db

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --target-dir /user/jaszhou/sqoop_import/retail_db
```

// `warehouse-dir` is the parent dir of the import data, data ends in `/user/jaszhou/sqoop_import/retail_db/order_items`
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db
```

Run the above command twice
`ERROR tool.ImportTool: Encountered IOException running import job: org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://quickstart.cloudera:8020/user/jaszhou/sqoop_import/retail_db/order_items already exists`

## Sqoop import --delete-target-dir or --append
//--delete-target-dir override existing data, --append appends data to exiting files
//Please use it, otherwise you get errors when dir exists
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --target-dir /user/jaszhou/sqoop_import/retail_db \
  --delete-target-dir
 ```

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --target-dir /user/jaszhou/sqoop_import/retail_db \
  --append
```

## Sqoop import with multiple mappers, columes should be indexed
//--num-of-mappers number of mappers you want to use
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --target-dir /user/jaszhou/sqoop_import/retail_db \
  --delete-target-dir \
  --num-mappers 1
```

// You can see 6 files in target dir when num of mappers is 6

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --target-dir /user/jaszhou/sqoop_import/retail_db \
  --delete-target-dir \
  --num-mappers 6
```

// When use more than 1 mappers, a primary key should be in place
// Create a table without primary key: `create table order_items_nopk as select * from order_items`;
// You can see 6 files in target dir when num of mappers is 6
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items_nopk \
  --target-dir /user/jaszhou/sqoop_import/retail_db \
  --delete-target-dir \
  --num-mappers 6
```

```
ERROR tool.ImportTool: Error during import: No primary key could be found for table order_items_nopk. Please specify one with --split-by or perform a sequential import with '-m 1'
```
