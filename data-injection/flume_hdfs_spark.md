# Flume and Spark Streaming

flume -> sparkSink -> spark
flume -> hdfs

## Flume Design: exec -> memory -> sparkSink, hdfs


```
# Name the components on this agent
a1.sources = r1
a1.sinks = hd spark
a1.channels = hdmem sparkmem

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/gen_logs/logs/access.log

# Describe the sink
a1.sinks.hd.type = hdfs
a1.sinks.hd.hdfs.path = hdfs://localhost:8803/user/jaszhou/solution/flume_exec_demo

agent.sinks.spark.type = org.apache.spark.streaming.flume.sink.SparkSink
agent.sinks.spark.hostname = localhost
agent.sinks.spark.port = 8123

# Use a channel which buffers events in memory
a1.channels.hdmem.type = memory
a1.channels.hdmem.capacity = 1000
a1.channels.hdmem.transactionCapacity = 100

a1.channels.sparkmem.type = memory
a1.channels.sparkmem.capacity = 1000
a1.channels.sparkmem.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = hdmem sparkmen
a1.sinks.hd.channel = hdmem
a1.sinks.spark.channel = sparkmem

```

note:
1. One source can connect to multiple channels
2. Each sink only connect to one channel


## Spark Design: spark streaming

// FlumeStreamingDepartmentCount.scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{StreamingContext, Seconds}
import org.apache.spark.streaming.flume._

object FlumeStreamingDepartmentCount {
  def main(args: Array[String]) {
    val conf = new SparkConf()
      .setAppName("Flume streaming and count")
      .setMaster(args(0))

    val ssc = new StreamingContext(conf, Seconds(30))

    val stream = FlumeUtils.createPollingStream(ssc, args(1), args(2).toInt)

    val messages = stream.map{s =>
      s => new String(msg.event.getBody.array())
    }
    val departmentMessage = messages
      .filter{ msg =>
        val endpint = msg.split(" ")(6)
        endpoint.split("/")(1) == "department"
      }
    val departments = departmentMessage.map { msg =>
      (msg.split(" ")(6).split("/")(2), 1)
    }

    val departmentTraffic = departments.conutByKey()

    departmentTraffic.saveAsTextFile("/user/jaszhou/solutions/departmentTraffic/cnt")

    ssc.start()
    ssc.awaitTermination()
  }
}
// build.sbt
```
name := "retail"
version := "1.0"
scalaVersion := "2.10.5"

libraryDependencies += "org.apache.spark" % "spark-core_2.10" % "1.6.3"
libraryDependencies += "org.apache.spark" % "spark-streaming_2.10" % "1.6.3"
libraryDependencies += "org.apache.spark" % "spark-streaming-flume_2.10" % "1.6.3"
libraryDependencies += "org.apache.spark" % "spark-streaming-flume-sink_2.10" % "1.6.3"
libraryDependencies += "org.apache.spark" % "scala-library" % "2.10.5"
libraryDependencies += "org.apache.commons" % "commons-lang3" % "3.3.2"
```

`sbt package`
