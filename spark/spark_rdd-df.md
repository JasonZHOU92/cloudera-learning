# Conversion between RDD and DataFrame in Spark


## RDD -> DataFrame


### use toDF
import spark.implicits._
import sqlContext.implicits._

```
val df = rdd.map({
  case Row(id: String, name: String, age: Int) => (id, name, age)
}).toDF("id", "name", "age")
```

or

```
case class Person(id: String, name: String, age: Int)
val df = rdd.map({
  case Row(id: String, name: String, age: Int) => Person(id, name, age)
}).toDF("id", "name", "age")
```

### use createDataFrame
import spark.implicits._
import sqlContext.implicits._


```
val rdd = oldDF.rdd
val newDF = oldDF.sqlContext.createDataFrame(rdd, oldDF.schema)
```

or
```
val schema = new StructType()
  .add(StructField("id", StringType, true))
  .add(StructField("name", StringType, true))
  .add(StructField("age", IntegerType, true))

val df = spark.createDataFrame(rdd, schema)
```

### use toDF
```
import spark.implicits._
val someDF = rdd.toDF("id", "name", "age")
```

### DataFrame -> RDD
val userRDD = userDF.rdd
