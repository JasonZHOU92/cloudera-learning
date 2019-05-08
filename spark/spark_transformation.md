# Spark transformations

## map, flatMap


val orderItemsRDD = sc.textFile("/user/root/sqoop_import/retail_db/orders")
orderItemsRDD.take(1)
val productsRaw = scala.io.Source.fromFile("/data/retail_db/products/part-00000").getLines.toList
val productsRDD = sc.parallelize(productsRaw).first


val orders = sc.textFile("/user/root/sqoop_import/retail_db/orders")
val order = orders.first
order.split(",")(1).substring(0,10).replace("-", "").toInt

val orderDates = orders.map{ order =>
  order.split(",")(1).substring(0,10).replace("-","").toInt
}

val ordersPairedRDD = orders.map{ order =>
  val o = order.split(",")
  (o(0).toInt, o(1).substring(0,10).replace("-","").toInt)  
}

val orderItems = sc.textFile("/user/root/sqoop_import/retail_db/order_items")

val orderItemsPairedRDD = orderItems.map{ orderItem =>
  (orderItem.split(",")(1).toInt, orderItem)
}

val sentences = List("Hello", "How are you doing", "Let us perform word count", "As part of the word count program", "we will see how many times each word repeat")

val sentencesRDD = sc.parallelize(sentences)

val words =  sentencesRDD.flatMap( s => s.split(" "))
val wordCount = words.map(w => (w, 1)).countByKey
val wordCount_ = words.map(w => (w, 1)).reduceByKey( _ + _ ).toMap

orders.map(order => order.split(",")(3)).distinct.collect.foreach(println)

## filter
```
scala> orders.map(order => order.split(",")(3)).distinct.collect.foreach(println)
CLOSED
DUMMY
CANCELED
PAYMENT_REVIEW
COMPLETE
PENDING_PAYMENT
PENDING
ON_HOLD
PROCESSING
SUSPECTED_FRAUD

```
orders.filter{ order =>
  val o = order.split(",")
  (o(3) == "COMPLETE" || o(3) == "CLOSED")
}


## Join (K, V) (K, W)
// Join orders and orderItems
val orders = sc.textFile("/user/root/sqoop_import/retail_db/orders")
val orderItems = sc.textFile("/user/root/sqoop_import/retail_db/order_items")

scala> orders.first
res69: String = 1,2013-07-25 00:00:00.0,11599,CLOSED
scala> orderItems.first
res71: String = 1,1,957,1,299.98,299.98

val ordersMap = orders.map { order =>
  (order.split(",")(0).toInt, order.split(",")(1).substring(0,10))
}

val orderItemsMap = orderItems.map{ orderItem =>
  val oi = orderItem.split(",")
  ((oi(1).toInt, oi(4).toFloat))
}

### Inner Join: (K, (V, W))
val ordersJoin = ordersMap.join(orderItemsMap)

### Left Outer Join: (K, (V, Option(W)))
// Get all the orders which do not have corresponding entries in orderItems
val orderLeftOuterJoin = ordersMap.leftOuterJoin(orderItemsMap)

scala> val orderLeftOuterJoin = ordersMap.leftOuterJoin(orderItemsMap)
orderLeftOuterJoin: org.apache.spark.rdd.RDD[(Int, (String, Option[Float]))] = MapPartitionsRDD[100] at leftOuterJoin at <console>:37
```
val ordersWithoutOrderItem = orderLeftOuterJoin.filter{ o =>
  o._2._2 == None
}
```

### Right Outer Join (K, (Option[V],W)
// Get all the orderItems which do not have corresponding entries in orders

val orderRightOuterJoin = ordersMap.rightOuterJoin(orderItemsMap)
scala> val orderRightOuterJoin = ordersMap.rightOuterJoin(orderItemsMap)
orderRightOuterJoin: org.apache.spark.rdd.RDD[(Int, (Option[String], Float))] = MapPartitionsRDD[105] at rightOuterJoin at <console>:37

```
val orderItemsWithoutOrders = orderRightOuterJoin.filter{ o =>
  o._2._1 == None
}
```

### Full Join (K, (Option[V], Option[W]))


## reduceByKey(reduce), groupByKey, aggregateByKey
countByKey: (K, V) => (K, Int)
reduceByKey: (K, V) => (K, V)
groupByKey: (K, V) => (K, Iterator[V])
aggregateByKey: (K, V) => (K, U)

// Aggregation - How many orders per stauts: countByKey

val orders = sc.textFile("/user/root/sqoop_import/retail_db/orders")
orders.map(order => (order.split(",")(3), "")).countByKey.foreach(println)
scala> orders.map(order => (order.split(",")(3), "")).countByKey.foreach(println)
(DUMMY,1)
(PAYMENT_REVIEW,729)
(CLOSED,7556)
(SUSPECTED_FRAUD,1558)
(PROCESSING,8275)
(COMPLETE,22899)
(PENDING,7610)
(PENDING_PAYMENT,15030)
(ON_HOLD,3798)
(CANCELED,1428)

// Aggregation - total revenue: reduce
val orderItems = sc.textFile("/user/root/sqoop_import/retail_db/order_items")
val orderItemsRevenue = orderItems.map(oi => oi.split(",")(4).toFloat)
orderItemsRevenue.reduce( _ + _ )

// Aggregation -  revenue per order: reduceByKey
val orderItemsRevenuePerOrder = orderItems.map(oi => (oi.split(",")(1).toInt, oi.split(",")(4).toFloat)).reduceByKey( _ + _ )

// Aggregation - total revenue, max revenue
```
val aggregatedResult = orderItems.map{ oi =>
  (oi.split(",")(1).toInt, oi.split(",")(4).toFloat)
}.aggregateByKey(0.0f, 0.0f){
  (inter, value) => {
    val total = inter._1 + value._2
    val max = if(inter._2 <= value._2) value._2 else inter._2
    (total, max)
  }
  (p1, p2) => {
    val total = p1._1 + p2._1
    val max = if(p1._2 <= p2._2) p2._2 else p1._2
    (total, max)
  }
}
```

```
// Get data in descending order by order_item_subtotal for each order_id
val orderItemsMap = orderItems.map(oi => (oi.split(",")(1).toInt, oi.split(",")(4).toFloat))
val orderItemsGBK = orderItemsMap.groupByKey
val ordersSortedbyRevenue = orderItemsGBK.flatMap( rec => rec._2.toList.sortBy(o => -o).map(k => (rec._1, k)))
```
// Get min and total of each orderId

val totalRevenuePerOrderId = orderItemsMap.reduceByKey( _ + _ )
val minRevenuePerOrderId = orderItemsMap.reduceByKey{
  (min, revenue) => if (min <= revenue) min else revenue
}

```
// Use aggregateByKey
val revenueAndMaxPerOrderId = orderItemsMap.aggregateByKey((0.0, 0.0)) (
    (inter, subtotal)  => (inter._1 + subtotal, if(inter._2 > subtotal) inter._2 else subtotal),
    (p1, p2) => (p1._1 + p2._1, if(p1._2 > p2._2) p1._2 else p2._2)
)
```

## Sorting & Ranking

### Sorting sortByKey(isAsc)
val products = sc.textFile("/user/root/sqoop_import/retail_db/products")
products.take(10).foreach(println)

val productsMap = products.map{product =>
    val p = product.split(",")
    (p(1).toInt, product)
}

val productsSortedByCategoryId = productsMap.sortByKey(false)
productsSortedByCategoryId.take(10).foreach(println)

// filter out empty string
val productsMap = products.filter( product => product.split(",")(4) != "").map{product =>
    val p = product.split(",")
    (p(1).toInt, p(4).toFloat)
}

val productsSortedByCategoryId = productsMap.sortByKey(false)

### Global Ranking: Global (details of top 5 products) takeOrdered
val products = sc.textFile("/user/root/sqoop_import/retail_db/products")
val productsMap = products.filter( product => product.split(",")(4) != "").map{product =>
    val p = product.split(",")
    (p(4).toFloat, product)
}

productsMap.sortByKey(false).take(5).foreach(println)

val products = sc.textFile("/user/root/sqoop_import/retail_db/products")
products.
  filter( product => product.split(",")(4) != "").
  takeOrdered(10)(Ordering[Float].reverse.on(product => product.split(",")(4).toFloat))

products.
  filter( product => product.split(",")(4) != "").
  takeOrdered(10)(Ordering[Float].on(product => product.split(",")(4).toFloat))

### Key Ranking: Get top N priced products in each category
val productsMap = products.
  filter( product => product.split(",")(4) != "").
  map(product => (product.split(",")(1), product))

val productsGroupByCategory = productsMap.groupByKey()


val productPrices = productsIterable.map( p => p.split(".")(4).toFloat).toSet
val topNPrices = productPrices.toList.sortBy( p => -p).take(topN)

def getTopNPrices(productsIterable: Iterable[String], topN: Int): Iterable[Float] = {
  val productPrices = productsIterable.map( p => p.split(".")(4).toFloat).toSet
  val topNPrices = productPrices.toList.sortBy( p => -p).take(topN)
  topNPrices
}

```
val top5PricePerCategory = productsGroupByCategory.map { productsByCategoryId =>
  getTopNProducts(productsByCategoryId._2, 5)
}
```

### Get all items in descending order

def getTopNProducts(productsIterable: Iterable[String], topN: Int): Iterable[String] = {
  productsIterable.toList.sortBy{ product=>
    product.split(",")(4).toFloat
  }.reverse.take(topN)
}
```
productsGroupByCategory.map { productsByCategoryId =>
  getTopNProducts(productsByCategoryId._2, 5)
}
```

### Set operation: union, intersection, distinct, minus

val orders = sc.textFile("/user/root/sqoop_import/retail_db/orders")
val customers_201308 = orders.
  filter(order => order.split(",")(1).contains("2013-08")).
  map( order => order.split(",")(2).toInt)

val customers_201309 = orders.
  filter(order => order.split(",")(1).contains("2013-09")).
  map( order => order.split(",")(2).toInt)

// Get all the customers who placed orders in 2013 August and 2013 September
val customers_201308_and_201309 = customers_201308.intersection(customers_201309).distinct

// Get all the customers who placed orders in 2013 August or 2013 September
val customers_201308_or_201309 = customers_201308.union(customers_201309).distinct

// Get all the customers who placed orders in 2013 August but not in 2013 September
```
val customers_201308_minus_201309 = customers_201308.map( c => (c, 1)).
  leftOuterJoin(customers_201309.map( c => (c, 1))).
  filter(c => c._2._2 == None).
  distinct
```
// Get all the customers who placed orders not in 2013 August but in 2013 September
```
val customers_201308_minus_201309 = customers_201308.map( c => (c, 1)).
  rightOuterJoin(customers_201309.map( c => (c, 1))).
  filter(c => c._2._1 == None).
  map(c => c._1).
  distinct

```
