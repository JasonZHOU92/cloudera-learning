# Problem statement

## User Retail_db dataset
## Problem statement
- get daily revenue by product considering complete and closed orders
- Data need to be sorted by ascending order by date and then descending order by revenue caomputed for each product for each day
- Data should be delimited by "," in this order - order_date, daily_revenue_per_product, product_name

## Data for orders and order_items is available in HDFS /public/retail_db/orders and /public/retail_db/order_items

## Data for products is available locally under /data/retail_db/products

## Final output need to be stored under
- HDFS location - avro format
  /user/YOUR_USERNAME/daily_revenue_avro_scala
- HDFS location - text format
  /user/YOUR_USERNAME/daily_revenue_txt_scala
- Solution need to be stored under
  /home/YOUR_USERNAME/daily_revenue_scala.txt




# Solution

// Load data
val orders  = sc.textFile("/public/retail_db/orders")
val orderItems  = sc.textFile("/public/retail_db/order_items")
val products = scala.io.Source.fromFile("/data/retail_db/products")

val orderByOrderId = orders.
  filter(order => (order.split(",")(3) == "COMPLETE" || orderItem.split(",")(3) == "CLOSED")).
  map( order => (order.split(",")(0), order))
val orderItemByOrderId = orderItems.
  map( orderItem => (orderItem.split(",")(1), orderItem))

val orderItemPerDatePerProductId =
  orderByOrderId.join(orderItemByOrderId).
  map{ rec =>
    val order = rec._2._1
    val orderItem = rec._2._2
    val oi = orderItem.split(",")
    val o = order.split(",")
    val productId = oi(2).toInt
    val date = o(1).substring(0,10)
    val subtotal = oi(4).toFloat
    (productId, (date, subtotal))
  }

val productById = products.map( p => (p.split(",")(0).toInt, p))

val orderItemPerDatePerProductName = orderItemPerDatePerProductId.join(productById).
  map{ rec =>
    val productName = rec._2._2.split(",")(2)
    val date = rec._2._1._1
    val subtotal = rec._2._1._2
    ((date, productName), subtotal)
  }.reduceByKey( _ + _ )

val resultRDD = orderItemPerDatePerProductName.map{ rec =>
  val date = rec._1._1
  val productName = rec._1._2
  val revenue = rec._2
  (date, (revenue, productName))
}.sortByKey.map{ rec =>
  rec._2.toList.sortBy(false)
}.map {rec =>
  (rec._1, rec._2._1, rec._2._2)
}

orderItemPerDatePerProductName.saveAsTextFile("hdfs:///user/jaszhou/daily_revenue_txt_scala")

val resultDF = resultRDD.toDF("order_date", "daily_revenue_per_product", "product_name")
resultDF.write.avro("/user/jaszhou/daily_revenue_avro_scala")

orderItemPerDatePerProductName.saveAsTextFile("file:///user/jaszhou/daily_revenue_scala.txt")
