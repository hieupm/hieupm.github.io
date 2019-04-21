---
published: true
title: "Spark - Processing Data with the Streaming API"
excerpt: "Processing Data with the Streaming API"
toc: true
toc_sticky: true
toc_label: "Content"
toc_icon: "terminal"
author_profile: false
comments: true
header:
  overlay_image: "assets/images/covers/cover-cloud.jpg"
  overlay_filter: 0.2
  teaser: "assets/images/covers/cover-cloud.jpg"
categories: [spark, scala, streaming]
---

# Spark Streaming's Mechanics

The core abstraction is known as a DStream, representing a discretized of RDDs built up over time. As time passes and data arrives, it's grouped together into time buckets, where the bucketing interval is based on your specified batch size, set during creation of the streaming context. 

For example, our batch size will be 1: 

![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-1.JPG "Streaming example"){: .align-center}

The DStream bastraction is itself built on top of RDDs, where data bucket is really just an RDD. In fact, the operations you specify for your DStreams end up translating into operations on each underlying RDD. 

![dstream-2]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-2.JPG){: .align-center}

By default, they work **sequentially**, following the order you specified in your application: every increase in the number of ouput jobs translates into an increase in overall batch time. So, if you do have multiple output and are looking to increase your throughput, then you can increase the parallelism via spark. 

If you have more multiple outputs and are looking to increase your throughout, then you can increase the parallelize **spark.streaming.concurrentJobs**

By default, this parameter is set to 1, because it makes batch debugging easier reduces the possibility of resource conflicts: you must try to merge output operations as much as possible before turning up this configuration value. 

The streaming context acts as the central coordinator, and that each input source uses up an entire executor, so that it can continuously receive incoming data. So, how does this all intertwine into the batch model? At the end of each batch interval, the streaming context launches the necessary jobs using its stored Spark context. Tasks are passed to the available executors. 

The input is continuing to be gathered by the receiver in parallel. So at any point in time, Spark can be simultaneously processing the previous intervals batch while the receiver continues to collect new data towards the current interval. Here, in this example, it's the 3 to 4 second data which is being gathered, all while the 2 to 3 second batch is still being analyzed, stored, and then printed to the console. 

![dstream-3]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-3.JPG "Streaming example"){: .align-center}

# Streaming in action with Kafka

In this section, we will browser Spark Streaming API throughout an example of transaction analysis with helping of Kafka as messaging broker management. 

Firstly, let's take a quick look a custom Kafka streaming utility used to produce our transaction stream. This is a console application that either produces a random flow of transactions or 100 non-random transactions. It takes as arguments: 
- SERVERS is a comma delimited list of servers in the format of [ServerAddress]:[Port] ~ Default = localhost:9092
- TOPICS is the Kafka topic you want to produce messages to ~ Default = transactions
- RANDOMIZE is a yes/no flag to turn on/off randomization of transaction output ~ Default = no

The necessary import

```java
import java.util.Properties
import org.apache.kafka.clients.producer.{KafkaProducer, ProducerConfig, Producer, ProducerRecord}
import org.apache.kafka.clients.producer

object KafkaTransactionProducer{
  def main(args: Array[String]) {
    val props = new Properties()
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, args.lift(0).getOrElse("localhost:9092"))
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer")
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer")
	
	  val accountNumbers = List("123-ABC-789", "456-DEF-456", "333-XYZ-999", "987-CBA-321")
	  val descriptions = List("Drug Store", "Grocery Store", "Electronics", "Park", "Gas", "Books", "Movies", "Misc")
	  val transactionAmounts = List(10.34, 94.65, 2.49, 306.21, 1073.12, 20.00, 7.92, 4322.33)
 
    val producer = new KafkaProducer[Nothing, String](props)
	
	System.out.print(">>>Press [ENTER] to shut the producer down")
	val topic = args.lift(1).getOrElse("transactions")
	val randomize = args.lift(2).map(_.toLowerCase).getOrElse("no") == "yes"
	var currentStep = 0
    while(System.in.available == 0 || (!randomize && currentStep <= 100)){
	  val delayUntilNextSend = if(randomize) scala.util.Random.nextInt(5000) else ((currentStep + 1) * 50) //Up to 5 seconds
	  Thread.sleep(delayUntilNextSend)
	  val accountNumber = if(randomize) accountNumbers(scala.util.Random.nextInt(accountNumbers.size)) else accountNumbers(currentStep % accountNumbers.size)
	  val description = if(randomize) descriptions(scala.util.Random.nextInt(descriptions.size)) else descriptions(currentStep % descriptions.size) 
	  val currentDate = (new java.text.SimpleDateFormat("MM/dd/yyyy")).format(new java.util.Date())
	  val txAmount = if(randomize) math.floor((scala.util.Random.nextInt(5000) + scala.util.Random.nextDouble) * 100) / 100 else transactionAmounts(currentStep % transactionAmounts.size) 
	  val transactionLogLine = s"$currentStep,$currentDate,$accountNumber,$txAmount,$description"
	  producer.send(new ProducerRecord(topic, transactionLogLine))
	  println("Sent -> " + transactionLogLine)
	  currentStep = currentStep + 1
	}
    
	producer.close()
  }
}
```

Now, we start to build streaming consumer by firstly import the necessary package. 

```java
import org.apache.spark.sql.{Dataset, Encoders, SaveMode, SparkSession}
import org.apache.spark.sql.functions._
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.expressions.scalalang.typed
import org.apache.spark.sql.cassandra._
import org.apache.spark.streaming._
import org.apache.spark.streaming.kafka._
import com.datastax.spark.connector._
```

There a some new domain classes which will be used to store our parsed or unparsed transactions. 

```java
case class SimpleTransaction(id: Long, account_number: String, amount: Double, 
                             date: java.sql.Date, description: String)
case class UnparsableTransaction(id: Option[Long], originalMessage: String, exception: Throwable)
```

Next, we head down to our actual streaming logic. We've left the stream context initialization from the current sparkContext and the batchSizeDuration of 1 second which it's the time interval at which streaming data will be divided into batches

```java
val streamingContext = new StreamingContext(spark.sparkContext, Seconds(1))
val kafkaStream = KafkaUtils.createStream(streamingContext, 
              "localhost:2181", "transactions-group", Map("transactions"->1))
```

It should be noted that there are number of possible overloads the SparkContext initialization methods (Ref: [Documentation](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext))

```java
val kafkaStream = KafkaUtils.createStream(streamingContext, 
              "localhost:2181", "transactions-group", Map("transactions"->1))
```

Let's take a little bit deep drive into "consumer group id" and "per topic number" conception.  

```java
val kafkaStream = KafkaUtils.createStream(streamingContext, [ZK quorum], [consumer group id], [per-topic number of Kafka partitions to consume])
```

```
Consumers label themselves with a consumer group name, and each record published to a topic is delivered to one consumer instance within each subscribing consumer group. Consumer instances can be in separate processes or on separate machines.

If all the consumer instances have the same consumer group, then the records will effectively be load balanced over the consumer instances.

If all the consumer instances have different consumer groups, then each record will be broadcast to all the consumer processes.
```

# More of the Streaming API

Now let's quickly exhibit some basic methods. Firstly, we need to import some neccessary library. 

```scala
import org.apache.spark.streaming._
import org.apache.spark.rdd._
val ssc = new StreamingContext(sc, Seconds(5))
val rddStream = (1 to 10).map(x => sc.makeRDD(List(x%4)))
val testStream = ssc.queueStream(scala.collection.mutable.Queue(rddStream: _*))
testStream.print
ssc.remember(Seconds(60))
val currMillis = System.currentTimeMillis
val startTime = Time(currMillis - (currMillis % 5000))
ssc.start
```

The setup here is that I've created a context with a 5-second batch interval, built a list of RDDs based on couting to 10, resetting to 0 on every fourth number, and then feeding that into the QStream method. This method will push out 1 RDD every interval. The next optional argument signifies that only one RDD should be consumed from the queue in every interval, and is set to true by default. 

```scala
val testStream = ssc.queueStream(scala.collection.mutable.Queue(rddStream: _*), [oneAtATime: true])
```

There's one last optional parameter, which is for providing a default RDD to handle the case where the queue runs out or is empty. This value is set as null by default if no RDD should be returned when empty.   

```scala
queueStream[T](queue: Queue[RDD[T]], oneAtATime: Boolean, defaultRDD: RDD[T])(implicit arg0: ClassTag[T]): InputDStream[T]
```

It's interessting that we can tell the streaming context to remember the last 60 seconds of data, as streaming will otherwise discard sources RDDs soon after their computation. Unless your stream is stateful,the data from the previous inputs  are kept until their pertinence to the state is gone or you need to explicitly keep the data if you're working on interactive queries. Explicit, as in you need to tell the streaming context how much trailing input should be kept via this remember method. Without this, you'll encounter exception at best, or odd, somewhat unpredictable behavior at wort. 

After running the code, we can requested a slice of the RDDs behind the batches, ranging from the start of stream until 10 seconds later. 

```scala
val slicedRDDs = testStream.slice(startTime, startTime+Seconds(10))
```

As long as the RDD is still in memory, then this allows you to run queries against the underlying data outside of the stream logic itself. 

Two more basic methods are count and countByValue over the stream. 
- First is **count**, which once started, will store the count of the current batch, note that it doesn't cross batch boundaries to create a running count. That is a bit more possible once we introduce tracking state across batches. 
- The second count function with "by value" suffix **countByValue** makes in our account, which as you might have expected, it's still batch-centric. The different is that **countByValue** creates a map of th values associated with their specific count (value -> count). In case of our example, when the "oneAtATime" parameter is true at the QStream method, the result is pretty much the same as count function described above. But if we add false as "oneAtATime", signifying to pump out the entire queue available in the first batch processing, the you'll see the aggregate counting effect. 

# DStream "RDD" API

This section is going to provide a quick overview of the available DStream methods that mimic those of the RDD API. First are the truly basic ones:
- map(Partitions) & flatMap
- filter
- reduce
- glom
  - Ref: http://blog.madhukaraphatak.com/glom-in-spark/

	![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/glom.PNG "glom example"){: .align-center}

- context / repartition
  - these  work exactly the same as RDD, except  that this context is the streaming context. 
- cache / persist
  - The different betwen streaming API and normal batch API is that, the default persistence level in streaming context is serialized in memory (storageLevel.MEMORY_ONLY_SER). This is to reduce the garbage collection overhead of handling tons of fast-moving, separate objects by storing them as larger blobs of data, although any of the libraries network-based streaming sources will change this to add a replication factor, so as to aid in a speedier failure recovery. These methods work by taking the specified persistence level, or default if not specified, and applying it to each underlying RDD, then utilizing the **spark.cleaner.ttl** (ttl=time to live) configuraiton value to handle removing the persisted RDD from memory. At the 2.x version, this config was removed as the internal context cleaner was improved to the point of no longer needing it. Of course, if you want more fine-grain control, then you could skip caching at this level, and perform it on the RDD yourself using the foreachRDD method, and calling unpersist via another foreachRDD when you're done. The rule of thumb when deciding whether to cache or not is just about the same as with RDDs. If you're going to use the same underlying RDD more than once, then you should use caching. In fact, some of stateful methods call persist automatically, since it's known that in general, each underlying RDD will be used across multiple batches to build up the state.
- (ssc.)union
  - the unions work the same as the RDD ones, where the DStream version takes the first DStream and combines it with one other, and the context version can take an arbitrary number of DStreams for combination. This is useful for when you need to increase throughput by spinning up multiple stream receivers, yet still treat them as though they're one single source DStream.
- ssc.tranform/transformWith
  - these methods are not RDD based, provide a way to write an union and tranform in one method call, where you join multiple streams, apply a transformation, and output it all back as one single DStream.
- PairDStream: 
  - mapValues & flatMapValues 
  - groupByKey
  - reduceByKey
  - combineByKey
  - cogroup
  - join
    - fullOuterJoin
    - leftOuterJoin
    - rightOuterJoin
- The persistence methods are the biggest difference where their method names are pluralized, since the action is executed more than once, which also means that the output parameter isn't one specific file, but instead a prefix and a suffix used to create a unique name comprised of this base filename prefix, followed by timestamp and the extension suffix. And although these methods are made available for generic cases, **if you can find a more native extension, like Cassandra code, then avoid HadoopFiles is probably best, as it tends to increase overall latency**. 
  - ObjectFiles
  - TextFiles
  - (NewAPI) HadoopFiles

# Stateful Streaming

```
The key difference between stateful and stateless applications is that stateless applications don’t “store” data whereas stateful applications require backing storage. Stateful applications like the Cassandra, MongoDB and mySQL databases all require some type of persistent storage that will survive service restarts.

Keeping state is critical to running a stateful application whereas any data that flows via a stateless service is typically transitory and the state is stored only in a separate back-end service like a database. 
```
Ref: https://www.bizety.com/2018/08/21/stateful-vs-stateless-architecture-overview/

We just saw how the Spark Streaming API can be used to create similar code to that of your batching logic, except now it can handle live streaming data so it's the main goal of Spark to unify the processing landscape, and cut down on the struggles of context switching between paradigms. Although there are some concepts that are fairly unique to streaming, such as working with state across a streaming period of time. So let's see some concepts to move easily create safe stateful streaming logic. 
- **windowing**: working withe internally-batched RDDs grouped as though they're one via sliding windows of data. In this example below, the data stream is batched into RDD buckets every one second. 
	![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-4.PNG "glom example"){: .align-center}
- **slide interval**, specified here as two 2s. This is the interval at which the window slides between each computation. In a nutshell, it's the amount of time for each executed computation. 
  ![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-5.PNG "glom example"){: .align-center}
- **window duration**, which has been set to 3s for this example. So if slide is the time between executions, then the window is how much data is passed into that computation. So for each subsequent slide, the amount of data worked on is equivalent to the window duration. 
  ![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-6.PNG "glom example"){: .align-center}

Notice that since our window is larger than our slide, that it actually trails back into the data from previous slide execution. So that you have to be aware of the duplication of data in each slide computation. And this will continue for every subsequent slide, with our window of data overlapping by the delta between slide and window. 

![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-7.PNG "glom example"){: .align-center}

This overlap is also why window-based operations implicitly call cache, since the same RDD is typically going to be used more than once, so why not optimize for it? Also, as mentioned above, technically window duration can be set to less than the slide, but it isn't good practice, as this just result in the slide computations working against a lagging window of constantly-outdated information. 

To recap, the slide is the period of time between each execution of your window logic, and the window duration deals with the amount of data involved in those executions, with the first data window being the same as the slide, if Window is equal or greater than the slide, there won't be time to build up enough data for a full window. **The window and slide intervals must be multiples of the batch timing**, as they need to make sure to work off of a completed batching bucket; otherwise you'd be working against still-incoming incomplete RDDs. 

![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-8.PNG "glom example"){: .align-center}

Another important, even required topic of stateful streaming is checkpointing. It just like caching, but persists to a resilient store like HDFS or S3, instead of a more transient one. And, as already stated, it's required for any stateful operations, meaning that as soon as you start a streaming context containing a stream using a stateful method, one that results in a must checkpoint DStream, and if you have not et a checkpoint, then it'll immediately throw an exception. Why is this required component of stateful streaming? Because managing state often relies on utilizing a portion of the previous data stream instead of just the currnt one, so extra care must be made so as to avoid any potential data loss and to improve fault tolerance. That's because streaming application typically run continuously resulting in their lineages building up over time, which leads to lengthy recomputation upon recovery. Checkpointing help us to avoid having to rebuild the entire lineage from scratch. 

![dstream-1]({{ site.url }}{{ site.baseurl }}/assets/images/streaming-9.PNG "glom example"){: .align-center}

It instead uses the checkpoint and rebuilds directly to the last known good state, and continues on its merry streaming way. Checkpointing in Spark is not only stores your stream state, it'll also save useful metadata information to help in the case of the driver failing. 