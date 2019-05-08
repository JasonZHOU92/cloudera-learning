# Spark Basic

## start spark shell
spark-shell --master yarn \
  --conf spark.ui.port=12654

spark-shell --help

hadoop fs -ls /user/root/retail_db
hadoop fs -du -s -h /user/root/retail_db

spark-shell --master yarn \
  --conf spark.ui.port=12654 \
  --num-executors 1 \
  --executor-memory 512M

sc.getConf.getAll.foreach(println)

// Initialize programatically
import org.apache.spark.{SparkContext, SparkConf}

val conf = new SparkConf().setAppName("Daily Revenue").setMaster("yarn-client")
val sc = new SparkContext(conf)

```
scala> sc.getConf.getAll.foreach(println)
(spark.org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter.param.PROXY_URI_BASES,http://quickstart.cloudera:8088/proxy/application_1556983252327_0055)
(spark.executor.memory,512M)
(spark.master,yarn-client)
(spark.org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter.param.PROXY_HOSTS,quickstart.cloudera)
(spark.executor.id,driver)
(spark.externalBlockStore.folderName,spark-f575896e-8f71-447f-b399-1251c7b26e50)
(spark.repl.class.uri,spark://172.17.0.2:39985/classes)
(spark.driver.port,39985)
(spark.repl.class.outputDir,/tmp/spark-689d0430-ed8a-4d10-99d2-c29e7db5d3d3/repl-0b9229f0-371f-4549-a2c0-768e4df2f87f)
(spark.driver.host,172.17.0.2)
(spark.driver.appUIAddress,http://172.17.0.2:12654)
(spark.ui.port,12654)
(spark.app.name,Spark shell)
(spark.executor.instances,1)
(spark.jars,)
(spark.submit.deployMode,client)
(spark.ui.filters,org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter)
(spark.app.id,application_1556983252327_0055)
```

## Import Data as RDD

### From hdfs
al orderItemsRDD = sc.textFile("/user/root/sqoop_import/retail_db/orders")
orderItemsRDD.take(1)

### from local
val productsRaw = scala.io.Source.fromFile("/data/retail_db/products/part-00000").getLines.toList
val productsRDD = sc.parallelize(productsRaw).first


## Read data from Different file formats (use sqlContext)
val sqlContext.load(path,source): DataFrame
val sqlContext.read.json(jsonRDD/path): DataFrame

val orderDF = sqlContext.read.json("/user/root/retail_db.json/orders"

val orderDF2 = sqlContext.read.json("hdfs:///user/root/retail_db_json/orders")
val orderDF = sqlContext.read.json("file:///data/retail_db_json/orders")

```
scala> orderDF.show
+-----------------+--------------------+--------+---------------+
|order_customer_id|          order_date|order_id|   order_status|
+-----------------+--------------------+--------+---------------+
|            11599|2013-07-25 00:00:...|       1|         CLOSED|
|              256|2013-07-25 00:00:...|       2|PENDING_PAYMENT|
|            12111|2013-07-25 00:00:...|       3|       COMPLETE|
|             8827|2013-07-25 00:00:...|       4|         CLOSED|
|            11318|2013-07-25 00:00:...|       5|       COMPLETE|
|             7130|2013-07-25 00:00:...|       6|       COMPLETE|
|             4530|2013-07-25 00:00:...|       7|       COMPLETE|
|             2911|2013-07-25 00:00:...|       8|     PROCESSING|
|             5657|2013-07-25 00:00:...|       9|PENDING_PAYMENT|
|             5648|2013-07-25 00:00:...|      10|PENDING_PAYMENT|
|              918|2013-07-25 00:00:...|      11| PAYMENT_REVIEW|
|             1837|2013-07-25 00:00:...|      12|         CLOSED|
|             9149|2013-07-25 00:00:...|      13|PENDING_PAYMENT|
|             9842|2013-07-25 00:00:...|      14|     PROCESSING|
|             2568|2013-07-25 00:00:...|      15|       COMPLETE|
|             7276|2013-07-25 00:00:...|      16|PENDING_PAYMENT|
|             2667|2013-07-25 00:00:...|      17|       COMPLETE|
|             1205|2013-07-25 00:00:...|      18|         CLOSED|
|             9488|2013-07-25 00:00:...|      19|PENDING_PAYMENT|
|             9198|2013-07-25 00:00:...|      20|     PROCESSING|
+-----------------+--------------------+--------+---------------+
only showing top 20 rows
```

```
scala> orderDF.select("order_id", "order_date").show
+--------+--------------------+
|order_id|          order_date|
+--------+--------------------+
|       1|2013-07-25 00:00:...|
|       2|2013-07-25 00:00:...|
|       3|2013-07-25 00:00:...|
|       4|2013-07-25 00:00:...|
|       5|2013-07-25 00:00:...|
|       6|2013-07-25 00:00:...|
|       7|2013-07-25 00:00:...|
|       8|2013-07-25 00:00:...|
|       9|2013-07-25 00:00:...|
|      10|2013-07-25 00:00:...|
|      11|2013-07-25 00:00:...|
|      12|2013-07-25 00:00:...|
|      13|2013-07-25 00:00:...|
|      14|2013-07-25 00:00:...|
|      15|2013-07-25 00:00:...|
|      16|2013-07-25 00:00:...|
|      17|2013-07-25 00:00:...|
|      18|2013-07-25 00:00:...|
|      19|2013-07-25 00:00:...|
|      20|2013-07-25 00:00:...|
+--------+--------------------+
only showing top 20 rows
```
