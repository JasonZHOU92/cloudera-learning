# Hiva in spark-shell

## spark sql
spark-shell --master yarn --conf spark.ui.port=12345

scala > sc
scala > sqlContext

sqlContext.sql("create database jaszhou_retail_db_txt")
sqlContext.sql("use jaszhou_retail_db_txt")
sqlContext.sql("show tables").show
sqlContext.sql("select * from orders limit 10").show


val orders = sc.textFile("/public/datea/retail_db/orders")
val ordersDF = orders.map { order =>
  val o = orders.split(",")
  (o(0).toInt, o(1), o(2).toInt, o(3))
}.toDF("order_id", "order_date", "order_customer_id", "order_status")

ordersDF.registerTempTable("orders")
sqlContext.sql("select * from orders limit 10").show

val productRaw = scala.io.SOurce.fromFile("/data/retail_db/products/part-m-00000").getLines.toList
val products = sc.parallelize(productRaw)
val productDF = products.map{ product =>
  val p = product.split(",")
  (p(0).toInt, p(2))
}.toDF("product_id", "product_name")

productDF.registerTempTable("products")
sqlContext.sql("select * from products limit 10").show


val dailyRevenuePerProduct = sqlContext.sql(
"""
select o.order_date, p.product_name, sum(oi.order_item_subtotal) daily_revenue_per_product
from orders o join order_items oi on o.order_id = oi.order_item_order_id
join product p on p.product_id = oi.order_item_product_id
where o.order_status in ("COMPLETE", "CLOSED")
group by o.order_date, p.product_name
order by o.order_date ASC, daily_revenue_per_product DESC
"""
)

sqlContext.sql("create datbase jaszhou_daily_revenue")
sqlContext.sql("create table jaszhou_daily_revenue.daily_revenue " +
  "(order_date string, product_name string, daily_revenue float) " +
  "stored as orc"
)

dailyRevenuePerProduct.insertInto("jaszhou_daily_revenue.daily_revenue")
sqlContext.sql("select * from jaszhou_daily_revenue.daily_revenue limit 2")

dailyRevenuePerProduct.registerTempTable()
dailyRevenuePerProduct.saveAsTable("jaszhou_daily_revenue.daily_revenue","orc")
dailyRevenuePerProduct.write,orc("/public/data/solutions/daily_revenue_orc")

## DataFrame Operations
dailyRevenuePerProduct.write.orc("/public/data/solutions/daily_revenue_orc")
dailyRevenuePerProduct.save("/public/data/solutions/daily_revenue_json", "json")
dailyRevenuePerProduct.saveAsTable("jaszhou_daily_revenue.daily_revenue","orc")
dailyRevenuePerProduct.insertInto("jaszhou_daily_revenue.daily_revenue")

dailyRevenuePerProduct.rdd.saveAsTextFile("/public/data/solutions/daily_revenue_txt")
dailyRevenuePerProduct.rdd.saveAsTextFile("hdfs:///public/data/solutions/daily_revenue_txt")
dailyRevenuePerProduct.rdd.saveAsTextFile("file:///data/solutions/daily_revenue_txt")
dailyRevenuePerProduct.saveAsSequenceFile("/public/data/solutions/daily_revenue_sequence_gzip",classOf[org.apache.hadoop.io.compress.GzipCodec])


dailyRevenuePerProduct.select("order_date", "daily_revenue_per_product").filter(dailyRevenuePerProduct["order_date"] == "2019-05-10 00:00:00.0").count
