# sqoop import data format



## Import data to specific format
+ --as-avrodatafile	Imports data to Avro Data Files
+ --as-sequencefile	Imports data to SequenceFiles
+ --as-textfile	Imports data as plain text (default)
+ --as-parquetfile	Imports data to Parquet Files

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --delete-target-dir \
  --num-mappers 2 \
  --as-sequencefile
```

## Compression algorithms
* Gzip
* Deflate
* Snappy
* Others

--compress -z enable compression
--compression-codec Use hadoop codec (default gzip)
```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --append \
  --num-mappers 2 \
  --as-textfile \
  --compress
```

```
sqoop import \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password cloudera \
  --table order_items \
  --warehouse-dir /user/jaszhou/sqoop_import/retail_db \
  --append \
  --num-mappers 2 \
  --as-textfile \
  --compress \
  --compression-codec org.apache.hadoop.io.compress.SnappyCodec
```

```
[root@quickstart order_items]# ls /etc/hadoop/conf
core-site.xml  hadoop-metrics.properties  log4j.properties  README
hadoop-env.sh  hdfs-site.xml              mapred-site.xml   yarn-site.xml
```

- org.apache.hadoop.io.compress.GZipCodec
- org.apache.hadoop.io.compress.DefaultCodec
- org.apache.hadoop.io.compress.SnappyCodec
