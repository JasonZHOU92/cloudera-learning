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

//(orderId, order)
val orderByOrderId = orders.
  filter(order => (order.split(",")(3) == "COMPLETE" || orderItem.split(",")(3) == "CLOSED")).
  map( order => (order.split(",")(0), order))

//(orderId, orderItem)
val orderItemByOrderId = orderItems.
  map( orderItem => (orderItem.split(",")(1), orderItem))

//(productId, (order, orderItem))
val orderItemPerDatePerProductId =
  orderByOrderId.join(orderItemByOrderId).
  //(productId, (date, subtotal))
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

//(productId, product)
val productById = products.map( p => (p.split(",")(0).toInt, p))

//(productId, ((date, subtotal), product))
val orderItemPerDatePerProductName = orderItemPerDatePerProductId.join(productById).
//((date, productName), subtotal)
  map{ rec =>
    val productName = rec._2._2.split(",")(2)
    val date = rec._2._1._1
    val subtotal = rec._2._1._2
    ((date, productName), subtotal)
//((date, productName), revenue)
  }.reduceByKey( _ + _ )

//((date, subtotal), (date, subtotal, productName))
val resultRDD = orderItemPerDatePerProductName.map(
    rec => ((rec._1._1, - rec._2), (rec._1._1, rec._1._2, rec._2 )
  ).sortByKey
  .(rec => rec._2)

val finalRDD =  resultRDD.map(rec => rec._1 + "," + rec._2 + "," + rec._3)


finalRDD.saveAsTextFile("hdfs:///user/jaszhou/daily_revenue_txt_scala")

import spark.sqlContext.implicits._
import spark.implicits._
val resultDF = resultRDD.toDF("order_date", "daily_revenue_per_product", "product_name")
resultDF.write.avro("/user/jaszhou/daily_revenue_avro_scala")

orderItemPerDatePerProductName.saveAsTextFile("file:///user/jaszhou/daily_revenue_scala.txt")
