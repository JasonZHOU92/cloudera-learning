# Data formats & Compression

## Save RDD back to HDFS ( text, sequence)

### saveAsTextFile
val orders = sc.textFile("/user/root/sqoop_import/retail_db/orders")
val orderCountbyStatus = orders.
  map( order => (order.split(",")(3), 1)).
  reduceByKey( _ + _ )

orderCountbyStatus.saveAsTextFile("/user/root/solutions/order_count_by_status")

orderCountbyStatus.saveAsSequenceFile("/user/root/solutions/order_count_by_status")


### Delimiter
orderCountbyStatus.map(rec => rec.replace(",", "\t"))


### Compression
org.apache.hadoop.io.compress.GzipCodec
org.apache.hadoop.io.compress.DefaultCodec
org.apache.hadoop.io.compress.SnappyCodec

orderCountbyStatus.saveAsSequenceFile("/user/root/solutions/order_count_by_status_gzip",classOf[org.apache.hadoop.io.compress.GzipCodec])


## Save DF into HDFS ( orc, json, parquet, avro)
steps to save into different file transformations
- Make sure data is represented as DataFrame
- Use write or save API to save DataFrame into different file formats
- Use Compression algorithm if required

### Read from json/parquet/orc and save as another format

//val ordersDF = sqlContext.read.json("/user/root/retail_db_json/orders")
val ordersDF = sqlContext.load("/user/root/retail_db_json/orders", "json")
ordersDF.save("/user/root/solutions/orders_parquet", "parquet")
sqlContext.load("/user/root/solutions/orders_parquet", "parquet")

ordersDF.write.orc("/user/root/solutions/orders_orc")
sqlContext.read.orc("/user/root/solutions/orders_orc")
