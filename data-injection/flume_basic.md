# Flume Basic
- agent
- channel
- source
- sink

![data flow model](http://archive.cloudera.com/cdh5/cdh/5/flume-ng-1.6.0-cdh5.10.1/_images/UserGuide_image00.png)


## Example: netcat -> memory -> logger

```
# netcat_memory_logger.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

start a flume job:
`flume-ng agent --name a1 --conf-file ./flume_example/netcat_memory_logger.conf`

`telnet localhost 44444`


## Example exec -> memory -> hdfs
```
# exec_memory_hdfs.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/gen_logs/logs/access.log

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://localhost:8020/user/jaszhou/solution/flume_exec_demo

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

notes:
1. use logger sink to debug, then switch to hdfs sink
2. the name node of hdfs is store in `/etc/hadoop/conf/core-site.xml`, look for `fs.defaultFS`


## Example spoolDir-> memory -> hdfs
Use case: We have s bunch of access generated in /var/log/apache/flumeSpool/YYYY_MM_DD_HH_access.log, a new file will be created every hour

```
# spoolDir_memory_hdfs.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = spoolDir
a1.sources.r1.spoolDir = /var/log/apache/flumeSpool

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://localhost:8020/user/jaszhou/solution/flume_spoolDir_demo

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## Example tailDir -> memory -> hdfs
Use case: We have a access log per country and generated in /var/log/apache/flumeTail/(England.log, Denmark.log ... )


```
# tailDir_memory_hdfs.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = tailDir
a1.sources.r1.positionFile = /var/log/flume/taildir_position.json
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /var/log/test1/example_system.log
#(headerKey = logType, value = systemLog)
a1.sources.r1.headers.f1.logType = systemLog

a1.sources.r1.filegroups.f2 = /var/log/apache/flumeTail/*.access.log*
#(headerKey = logType, value = accessLog)
a1.sources.r1.headers.f2.logType = accessLog

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://localhost:8020/user/jaszhou/solution/flume_tailDir_demo

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
