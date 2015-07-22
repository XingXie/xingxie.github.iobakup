
---
layout: post
title:  "Spark Study Notes"
date:   2015-06-24
categories: Study-Notes
excerpt: spark
comments: true
---

## Adv

[! alt text](https://cloud.githubusercontent.com/assets/5607138/8816519/d82dc1f4-2fdc-11e5-9c74-6e2904f3bcd3.png)
[! alt text](https://cloud.githubusercontent.com/assets/5607138/8816520/d83dac5e-2fdc-11e5-83eb-4c35c8d79ab5.png)


## Basics 

Only one SparkContext may be active per JVM. You must stop() the active SparkContext before creating a new one.

The appName parameter is a name for your application to show on the cluster UI. master is a Spark, Mesos or YARN cluster URL, or a special “local” string to run in local mode. In practice, when running on a cluster, you will not want to hardcode master in the program, but rather launch the application with spark-submit and receive it there

~~~ scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
~~~

Once created, the distributed dataset (distData) can be operated on in parallel. 
For example, we might call distData.reduce((a, b) => a + b) to add up the elements of the array. 
We describe operations on distributed datasets later on.Normally, 
Spark tries to set the number of partitions automatically based on your cluster. 
However, you can also set it manually by passing it as a second parameter to parallelize 
(e.g. sc.parallelize(data, 10)).
Note: some places in the code use the term slices (a synonym for partitions) to maintain backward compatibility.

By default, each transformed RDD may be recomputed each time you run an action on it.
However, you may also persist an RDD in memory using the persist (or cache) method, in which case Spark will keep the elements around on the cluster for much faster access the next time you query it. There is also support for persisting RDDs on disk, or replicated across multiple nodes.

In addition, each persisted RDD can be stored using a different storage level, allowing you, for example, to persist the dataset on disk, persist it in memory but as serialized Java objects (to save space), replicate it across nodes, or store it off-heap in Tachyon. These levels are set by passing aStorageLevel object (Scala, Java, Python) to persist(). The cache() method is a shorthand for using the default storage level, which isStorageLevel.MEMORY_ONLY (store deserialized objects in memory).

### Removing Data

Spark automatically monitors cache usage on each node and drops out old data partitions in a least-recently-used (LRU) fashion. If you would like to manually remove an RDD instead of waiting for it to fall out of the cache, use the RDD.unpersist() method.

Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks. They can be used, for example, to give every node a copy of a large input dataset in an efficient manner. Spark also attempts to distribute broadcast variables using efficient broadcast algorithms to reduce communication cost.

### spark streaming

Note that, if you want to receive multiple streams of data in parallel in your streaming application, you can create multiple input DStreams (discussed further in the Performance Tuning section). This will create multiple receivers which will simultaneously receive multiple data streams. But note that Spark worker/executor as a long-running task, hence it occupies one of the cores allocated to the Spark Streaming application. Hence, it is important to remember that Spark Streaming application needs to be allocated enough cores (or threads, if running locally) to process the received data, as well as, to run the receiver(s).

Spark Streaming will monitor the directory dataDirectory and process any files created in that directory (files written in nested directories not supported). Note that

*The files must have the same data format.
*The files must be created in the dataDirectory by atomically moving or renaming them into the data directory.
*Once moved, the files must not be changed. So if the files are being continuously appended, the new data will not be read.
To summarize, metadata checkpointing is primarily needed for recovery from driver failures, whereas data or RDD checkpointing is necessary even for basic functioning if stateful transformations are used.

Typically, a checkpoint interval of 5 - 10 times of sliding interval of a DStream is good setting to try.

ackage the application JAR - You have to compile your streaming application into a JAR. If you are using spark-submit to start the application, then you will not need to provide Spark and Spark Streaming in the JAR. However, if your application uses advanced sources(e.g. Kafka, Flume, Twitter), then you will have to package the extra artifact they link to, along with their dependencies, in the JAR that is used to deploy the application. For example, an application using TwitterUtils will have to include spark-streaming-twitter_2.10 and all its transitive dependencies in the application JAR.

### Upgrading Application Code

If a running Spark Streaming application needs to be upgraded with new application code, then there are two possible mechanism.

* The upgraded Spark Streaming application is started and run in parallel to the existing application. Once the new one (receiving the same data as the old one) has been warmed up and ready for prime time, the old one be can be brought down. Note that this can be done for data sources that support sending the data to two destinations (i.e., the earlier and upgraded applications).
* The existing application is shutdown gracefully (see StreamingContext.stop(...) or JavaStreamingContext.stop(...) for graceful shutdown options) which ensure data that have been received is completely processed before shutdown. Then the upgraded application can be started, which will start processing from the same point where the earlier application left off. Note that this can be done only with input sources that support source-side buffering (like Kafka, and Flume) as data needs to be buffered while the previous application was down and the upgraded application is not yet up. And restarting from earlier checkpoint information of pre-upgrade code cannot be done. The checkpoint information essentially contains serialized Scala/Java/Python objects and trying to deserialize objects with new, modified classes may lead to errors. In this case, either start the upgraded app with a different checkpoint directory, or delete the previous checkpoint directory.

The following two metrics in web UI are particularly important:

* Processing Time - The time to process each batch of data.
* Scheduling Delay - the time a batch waits in a queue for the processing of previous batches to finish.

If the batch processing time is consistently more than the batch interval and/or the queueing delay keeps increasing, then it indicates the system is not able to process the batches as fast they are being generated and falling behind. In that case, consider reducing the batch processing time.
Receiving data over the network (like Kafka, Flume, socket, etc.) requires the data to deserialized and stored in Spark. If the data receiving becomes a bottleneck in the system, then consider parallelizing the data receiving. Note that each input DStream creates a single receiver (running on a worker machine) that receives a single stream of data. Receiving multiple data streams can therefore be achieved by creating multiple input DStreams and configuring them to receive different partitions of the data stream from the source(s). For example, a single Kafka input DStream receiving two topics of data can be split into two Kafka input streams, each receiving only one topic. This would run two receivers on two workers, thus allowing data to be received in parallel, and increasing overall throughput. These multiple DStream can be unioned together to create a single DStream. Then the transformations that was being applied on the single input DStream can applied on the unified stream. This is done as follows.

~~~ scala
val numStreams = 5
val kafkaStreams = (1 to numStreams).map { i => KafkaUtils.createStream(...) }
val unifiedStream = streamingContext.union(kafkaStreams)
unifiedStream.print()
~~~

A good approach to figure out the right batch size for your application is to test it with a conservative batch interval (say, 5-10 seconds) and a low data rate. To verify whether the system is able to keep up with data rate, you can check the value of the end-to-end delay experienced by each processed batch (either look for “Total delay” in Spark driver log4j logs, or use the StreamingListener interface). If the delay is maintained to be comparable to the batch size, then system is stable. Otherwise, if the delay is continuously increasing, it means that the system is unable to keep up and it therefore unstable. Once you have an idea of a stable configuration, you can try increasing the data rate and/or reducing the batch size. Note that momentary increase in the delay due to temporary data rate increases maybe fine as long as the delay reduces back to a low value (i.e., less than batch size).

1. An RDD is an immutable, deterministically re-computable, distributed dataset. Each RDD remembers the lineage of deterministic operations that were used on a fault-tolerant input dataset to create it.
2. If any partition of an RDD is lost due to a worker node failure, then that partition can be re-computed from the original fault-tolerant dataset using the lineage of operations.
3. Assuming that all of the RDD transformations are deterministic, the data in the final transformed RDD will always be the same irrespective of failures in the Spark cluster.

Furthermore, there are two kinds of failures that we should be concerned about:

1. Failure of a Worker Node - Any of the worker nodes running executors can fail, and all in-memory data on those nodes will be lost. If any receivers were running on failed nodes, then their buffered data will be lost.
2. Failure of the Driver Node - If the driver node running the Spark Streaming application fails, then obviously the SparkContext is lost, and all executors with their in-memory data are lost.

To avoid this loss of past received data, Spark 1.2 introduces an experimental feature of write ahead logs which saves the received data to fault-tolerant storage. With the write ahead logs enabled and reliable receivers, there is zero data loss and exactly-once semantics.

you have two control knobs in Spark that determine read parallelism for Kafka:

1. The number of input DStreams. Because Spark will run one receiver (= task) per input DStream, this means using multiple input DStreams will parallelize the read operations across multiple cores and thus, hopefully, across multiple machines and thereby NICs.
2. The number of consumer threads per input DStream. Here, the same receiver (= task) will run multiple threads. That is, read operations will happen in parallel but on the same core/machine/NIC.

For practical purposes option 1 is the preferred.

The example below is taken from the Spark Streaming Programming Guide.

~~~ scala
val ssc: StreamingContext = ??? // ignore for now
val kafkaParams: Map[String, String] = Map("group.id" -> "terran", /* ignore rest */)

val numInputDStreams = 5
val kafkaDStreams = (1 to numInputDStreams).map { _ => KafkaUtils.createStream(...) }
~~~

In this example we create five input DStreams, thus spreading the burden of reading from Kafka across five cores and, hopefully, five machines/NICs. (I say “hopefully” because I am not certain whether Spark Streaming task placement policy will try to place receivers on different machines.) All input DStreams are part of the “terran” consumer group, and the Kafka API will ensure that these five input DStreams a) will see all available data for the topic because it assigns each partition of the topic to an input DStream and b) will not see overlapping data because each partition is assigned to only one input DStream at a time. In other words, this setup of “collaborating” input DStreams works because of the consumer group behavior provided by the Kafka API, which is used behind the scenes by KafkaInputDStream.

That is, streams are not able to detect if they have lost connection to the upstream data source and thus cannot react to this event, e.g. by reconnecting or by stopping the execution. Similarly, if you lose a receiver that reads from the data source, then your streaming application will generate empty RDDs.

### Combining options 1 and 2

Here is a more complete example that combines the previous two techniques:

~~~ scala
val ssc: StreamingContext = ???
val kafkaParams: Map[String, String] = Map("group.id" -> "terran", ...)

val numDStreams = 5
val topics = Map("zerg.hydra" -> 1)
val kafkaDStreams = (1 to numDStreams).map { _ =>
    KafkaUtils.createStream(ssc, kafkaParams, topics, ...)
  }
~~~

We are creating five input DStreams, each of which will run a single consumer thread. If the input topic “zerg.hydra” has five partitions (or less), then this is normally the best way to parallelize read operations if you care primarily about maximizing throughput.

Hence repartition is our primary means to decouple read parallelism from processing parallelism. It allows us to set the number of processing tasks and thus the number of cores that will be used for the processing. Indirectly, we also influence the number of machines/NICs that will be involved.
A related DStream transformation is union. (This method also exists for StreamingContext, where it returns the unified DStream from multiple DStreams of the same type and same slide duration. Most likely you would use the StreamingContext variant.) A union will return a UnionDStream backed by a UnionRDD. A UnionRDD is comprised of all the partitions of the RDDs being unified, i.e. if you unite 3 RDDs with 10 partitions each, then your union RDD instance will contain 30 partitions. In other words, union will squash multiple DStreams into a single DStream/RDD, but it will not change the level of parallelism. Whether you need to use union or not depends on whether your use case requires information from all Kafka partitions “in one place”, so it’s primarily because of semantic requirements. One such example is when you need to perform a (global) count of distinct elements.
RDDs are not ordered. So when you union RDDs, then the resulting RDD itself will not have a well-defined ordering either. If you need ordering sort the RDD.

* Spark’s usage of the Kafka consumer parameter auto.offset.reset is different from Kafka’s semantics. In Kafka, the behavior of setting auto.offset.reset to “smallest” is that the consumer will automatically reset the offset to the smallest offset when a) there is no existing offset stored in ZooKeeper or b) there is an existing offset but it is out of range. Spark however will always remove existing offsets and then start all the way from zero again. This means whenever you restart your application with auto.offset.reset = "smallest", your application will completely re-process all available Kafka data. Doh! See this discussionand that discussion.

[source](http://www.michael-noll.com/blog/2014/10/01/kafka-spark-streaming-integration-example-tutorial/)

### modes

There are 4 was to run spark:

* local
* Standalone
* YARN
* Mesos

When Spark is run in local mode (as you're doing on your laptop) a separate Spark Master and separate Spark Workers + Exectuors are not launched.

Instead a single Java Virtual Machine (JVM) is launched with one Executor, whose ID is <driver>. This special Executor runs the Driver (which is the "Spark shell" application in this instance) and this special Executor also runs our Scala code. By default, this single Executor will be started with X threads, where X is equal to the # of cores on your machine.

Local mode is used when you want to run Spark locally and not in a distributed cluster. Spark local mode is different than Standalone mode (which is still designed for a cluster setup).

To summarize, in local mode, the Spark shell application (aka the Driver) and the Spark Executor is run within the same JVM. More precisely, the single Executor that is launched is named <driver> and this Executor runs both the driver code and the executes our Spark Scala transformations and actions.

So, although there is no Master UI in local mode, if you are curious, here is what a Master UI looks like here is a screenshot:
