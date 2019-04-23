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

# Stateful Streaming: windowing and checkpointing

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

# Utilizing State for Speedy Fraud Detection

In this example, we want to boost our code so we can flag down possible fraudulent transactions. We'll do this by calculating overall transaction averages, as well as averages over windows of time, and compare that data against our stored historic averages. Fisrt, we have some case classes: 

```scala
case class Account(number: String, firstName: String, lastName: String)
case class Transaction(id: Long, account: Account, date: java.sql.Date, amount: Double, 
                       description: String)
case class TransactionForAverage(accountNumber: String, amount: Double, description: String, 
                                 date: java.sql.Date)
case class SimpleTransaction(id: Long, account_number: String, amount: Double, 
                             date: java.sql.Date, description: String)
case class UnparsableTransaction(id: Option[Long], originalMessage: String, exception: Throwable)
case class AggregateData(totalSpending: Double, numTx: Int, windowSpendingAvg: Double) {
  val averageTx = if(numTx > 0) totalSpending / numTx else 0
}
case class EvaluatedSimpleTransaction(tx: SimpleTransaction, isPossibleFraud: Boolean)
```

The real guts of our code is down here where the streaming logic awaits. 

```scala
val checkpointDir = "file:///checkpoint"	
    val streamingContext = StreamingContext.getOrCreate(checkpointDir, () => {
	    val ssc = new StreamingContext(spark.sparkContext, Seconds(5))
		  ssc.checkpoint(checkpointDir)
		  val kafkaStream = KafkaUtils.createStream(ssc, "localhost:2181", "transactions-group", Map("transactions"->1))
	    kafkaStream.map(keyVal => tryConversionToSimpleTransaction(keyVal._2))
	               .flatMap(_.right.toOption)
				         .map(simpleTx => (simpleTx.account_number, simpleTx))
				         .window(Seconds(10), Seconds(10))
				         .updateStateByKey[AggregateData]((newTxs: Seq[SimpleTransaction], 
				                                          oldAggDataOption: Option[AggregateData]) => 
		  {
			  val calculatedAggTuple = newTxs.foldLeft((0.0,0))((currAgg, tx) => (currAgg._1 + tx.amount, currAgg._2 + 1))
			  val calculatedAgg = AggregateData(calculatedAggTuple._1, calculatedAggTuple._2, 
			                             if(calculatedAggTuple._2 > 0) calculatedAggTuple._1/calculatedAggTuple._2 else 0)
			  Option(oldAggDataOption match { 
				  case Some(aggData) => AggregateData(aggData.totalSpending + calculatedAgg.totalSpending, 
				                                      aggData.numTx + calculatedAgg.numTx, calculatedAgg.windowSpendingAvg)
					case None => calculatedAgg
				})
		  })
                 .transform(dStreamRDD => 
      {
			  val sc = dStreamRDD.sparkContext
			  val historicAggsOption = sc.getPersistentRDDs
					                         .values.filter(_.name == "storedHistoric").headOption
			  val historicAggs = historicAggsOption match {
				  case Some(persistedHistoricAggs) => 
				    persistedHistoricAggs.asInstanceOf[org.apache.spark.rdd.RDD[(String, Double)]]
					case None => {
					  import com.datastax.spark.connector._  
					  val retrievedHistoricAggs = sc.cassandraTable("finances", "account_aggregates")
					                                .select("account_number", "average_transaction")
						                              .map(cassandraRow => (cassandraRow.get[String]("account_number"), 
						                                                    cassandraRow.get[Double]("average_transaction")))
						retrievedHistoricAggs.setName("storedHistoric")
						retrievedHistoricAggs.cache
					}
				}
				dStreamRDD.join(historicAggs)
			})
				         .filter{ case (acctNum, (aggData, historicAvg)) => 
				           aggData.averageTx - historicAvg > 2000 || aggData.windowSpendingAvg - historicAvg > 2000 
				         }
				         .print
		  ssc
	  })
       
	  streamingContext.start
	  streamingContext.awaitTermination
  }
```
- stateful code requires a checkpoint directory to be set, 
- for a fully recovery-enabled application,your logic need to be built up inside of the getOrCreate method available on the static Spark Streaming object. 
- Spark can handle worker failure natively, but what happens when the driver itself dies? GetOrCreate handles this by putting all of the stream setup logic inside this function, to be used for the initial run of the stream. Basically when getOrCreate is called, it'll first look for the the checking point (**checkpointDir**) data using the provided checkpoint directory, only invoking the provided logic as a fallback. 
- Although this only makes the code able to be restarted, so you'll still need to setup some kind of supervisor to handle relaunching the driver upon failure. 
  - if you're using Spark standalone as your cluster manager, then you can submit your application, run in cluster mode, with the supervise flag, and it'll handle it for you. 
    ```java
		spark-submit --deploy-mode cluster --supervise
		```
  - YARN cluster mode provides a restart mechanism enabled via a configuration in your YARN-site.xml (**yarn.resourcemanager.am.max-attempts**)
  - Mesos handles restarts through its marathon orchestration platform. 
- there of course also has to be checkpoint data to be read; otherwise, it'll just keep invoking this initial setup logic, which is why we set the checkpoint directory on the streaming context object that'll be returned out of this createOrCreate
- the context batch is 5 seconds, so as to make reviewing our realtime stream easier. 
- Before digging into the checkpoint method itself, let's wrap up the discussion on our creation method and some similar alternatives. There are two optional parameters.
  
```java
public static StreamingContext getOrCreate(String checkpointPath, scala.Function0<StreamingContext> creatingFunc, org.apache.hadoop.conf.Configuration hadoopConf, boolean createOnError)
```

```
Either recreate a StreamingContext from checkpoint data or create a new StreamingContext. If checkpoint data exists in the provided checkpointPath, then StreamingContext will be recreated from the checkpoint data. If the data does not exist, then the StreamingContext will be created by called the provided creatingFunc.
Parameters:
- checkpointPath - Checkpoint directory used in an earlier StreamingContext program
- creatingFunc - Function to create a new StreamingContext
- hadoopConf - Optional Hadoop configuration if necessary for reading from the file system
- createOnError - Optional, whether to create a new StreamingContext if there is an error in reading checkpoint data. By default, an exception will be thrown on error.
```

There are also three similar, albeit experimental methods available for extracting a streaming context object. 
- getActive
  - As there can only be one active started streaming context, getActive is the method to statically access it. 
- getActiveOrCreate(createFunc) and getActiveOrCreate(checkpointDir, createFunc)
  - The getActiveOrCreate methods have two flavors, one only expects the creation function, with the primary context coming from the active running one before falling back to the provided logic, no checkpoint search is performed. Whereas the other overload is a replica of getOrCreate except the order of preference is to: 
    - search for an inactive context, then 
    - build one from the provided checkpoint directory 
    - and only then falling back to the logic

Now to wrap this recovery story up, let's return to discuss our setting of the checkpoint directory. The checkpoint must be explicitly set, or else this setup is mostly moot, as it'll continuously fail to find the checkpoint files and always go to the creation logic. Of course in case of stateful logic, you're somewhat covered, as it'll throw an exception as soon as you call start, but keep in mind that if you set up getOrCreate without statefull logic and do not set the checkpoint, then it won't complain, it'll just act as though it wasn't there. With that warning out of the way, let's dig into checkpoint itself. There's one more method centered around checkpointing, this one comes from the DStream **dStream.checkpoint(frequencyInterval)**, the frequencyInterval is must by a multiple of the slide duration. It's used if you're looking to trigger execution of the checkpoint process more or less frequent than the default where the default is only set if it's stateful and is either the slide duration or 10 seconds, whichever is larger. This is something that you'll typically only change if you're in need of either higher stability or higher performance, as that's the tradeoff here. If you set checkpointing to occur too often, then it'll result in a slowdown of total operation throughput. But if you set it to be too infrequent, then the recovery time suffered due to larger lineages and task sizes. If you are facing a stability or performance issue, then changing the checkpoint interval is a good starting point. And per the Spark docs, a good starting point is to set the interval to be 5 to 10 times your sliding interval, and tweak it up or down from there. 

Moving on, the map and flatMap remain the same as before, with the addition of another map, keying our stream off of the account number, which will give us access to the pair DStream functions, where the majority of stateful methods reside. 

```scala
kafkaStream.map(keyVal => tryConversionToSimpleTransaction(keyVal._2))
	        .flatMap(_.right.toOption)
				  .map(simpleTx => (simpleTx.account_number, simpleTx))
```

However, before that, we'll chunk (**cắt lát**) our data into 10-second slides via window method marking the window and slides duration on the stream where in fact slide is optional, defaulting to be the batch interval. 

```
window(windowLength, slideInterval)	
Return a new DStream which is computed based on windowed batches of the source DStream.
```

There are a number of other methods with window built in, all of which could be built with windows separately; however, it's always nice to simplify common cases with helpers, and some are even built with an extra hint of efficiency already in mind, so you should try one of those first. In our case, all we're looking for is the chunking, so that we can get extra insight into our data, comparing the long-running average, as well as the average of the most recent time window to historic average. And we do that by turning the transaction chunks into average data, which we carry across different iterations via the **updateStateByKey** method

```scala
.updateStateByKey[AggregateData]((newTxs: Seq[SimpleTransaction], 
				                          oldAggDataOption: Option[AggregateData]) => 
		  {
			  val calculatedAggTuple = newTxs.foldLeft((0.0,0))((currAgg, tx) => (currAgg._1 + tx.amount, currAgg._2 + 1))
			  val calculatedAgg = AggregateData(calculatedAggTuple._1, calculatedAggTuple._2, 
			                             if(calculatedAggTuple._2 > 0) calculatedAggTuple._1/calculatedAggTuple._2 else 0)
			  Option(oldAggDataOption match { 
				  case Some(aggData) => AggregateData(aggData.totalSpending + calculatedAgg.totalSpending, 
				                                      aggData.numTx + calculatedAgg.numTx, calculatedAgg.windowSpendingAvg)
					case None => calculatedAgg
				})
		  })
```
This method groups the key value input stream, providing the newest values and the last state tied to that key, if one already exists. 

# An improved stateful stream via mapWithState

In the last section, we have seen UpdateStateByKey to keep state throughout our stream; however, we also learned that each iteration processes the entire state map, no matter if there's input or not. In this section, we'll learn about a more performant alternative at your disposal, but have a little bit different format, so we first have to modify our code. 

```scala
  val kafkaStream = KafkaUtils.createStream(ssc, "localhost:2181", "transactions-group", Map("transactions"->1))
	    kafkaStream.map(keyVal => tryConversionToSimpleTransaction(keyVal._2))
	               .flatMap(_.right.toOption)
				         .map(simpleTx => (simpleTx.account_number, simpleTx))
                 .mapValues(tx => List(tx))
				         .reduceByKeyAndWindow((txs,otherTxs) => txs ++ otherTxs, (txs, oldTxs) => txs diff oldTxs, 
				                               Seconds(10), Seconds(10))
				         .mapWithState(
				          StateSpec.function((acctNum:String, newTxsOpt: Option[List[SimpleTransaction]], 
				                         aggData: State[AggregateData]) => 
		  {
			  val newTxs = newTxsOpt.getOrElse(List.empty[SimpleTransaction])
				val calculatedAggTuple = newTxs.foldLeft((0.0,0))((currAgg, tx) => (currAgg._1 + tx.amount, currAgg._2 + 1))
				val calculatedAgg = AggregateData(calculatedAggTuple._1, calculatedAggTuple._2, 
					                         if(calculatedAggTuple._2 > 0) calculatedAggTuple._1/calculatedAggTuple._2 else 0)
				val oldAggDataOption = aggData.getOption
				aggData.update(
				  oldAggDataOption match { 
					  case Some(aggData) => 
					    AggregateData(aggData.totalSpending + calculatedAgg.totalSpending, 
					                  aggData.numTx + calculatedAgg.numTx, calculatedAgg.windowSpendingAvg)
					  case None => calculatedAgg
					}
			  )
        newTxs.map(tx => EvaluatedSimpleTransaction(tx, tx.amount > 4000))
			}))
				         .stateSnapshots
```

We're changing our value type to be a list of transactions, which we run through the reduceByKeyAndWindow method. 
```scala
	.mapValues(tx => List(tx))
	.reduceByKeyAndWindow((txs,otherTxs) => txs ++ otherTxs, (txs, oldTxs) => txs diff oldTxs, Seconds(10), Seconds(10))
```
This replaces our simple call to window, as this reduction method is one of the helper window methods referenced in the last section. In fact, it's one that allows for an increase in performance. That's because it has the normal reduction method, where we're jsut merging lists into one blob but it also has an overload that takes inverse reduction function. This parameter provides you the new values and the old, so you can essentially substract the two and hint to the system which values have dropped from one slide to the next. This data allows the method to work only against the deltas, and boost the overall performance of the sliding window. In this case, it's as simple as using diff to act as a substraction then of course we provide the window and slide duration to complete the method. And don't worry, if there's no inverse for your reduction, then there's an overload without it, it just might not be quite as efficient. Additionally, should you not have a keyed data stream, then there are also reduced by window variants matching the keyed version. 

Now we're readky for our new and improved state tracking function mapWithState. As already eluded, this method was added in Spark 1.6 and boasts a possible performance improvement up to 10 times overs updateStateByKey, and this is in large part due to the fact that it only runs through the data coming from the stream, not the joining of the stream and the existing state. This means that the performance is no longer based on the size of the state, but instead the size of the batch. **mapWithState** takes only one input, a StateSpec object, which is built by calling StateSpec.function, and passing in a function with either 3 or 4 inputs. 

```scala
.mapWithState(StateSpec.function((acctNum:String, newTxsOpt: Option[List[SimpleTransaction]], aggData: State[AggregateData]) => 
		  {
			  val newTxs = newTxsOpt.getOrElse(List.empty[SimpleTransaction])
				val calculatedAggTuple = newTxs.foldLeft((0.0,0))((currAgg, tx) => (currAgg._1 + tx.amount, currAgg._2 + 1))
				val calculatedAgg = AggregateData(calculatedAggTuple._1, calculatedAggTuple._2, 
					                         if(calculatedAggTuple._2 > 0) calculatedAggTuple._1/calculatedAggTuple._2 else 0)
				val oldAggDataOption = aggData.getOption
				aggData.update(
				  oldAggDataOption match { 
					  case Some(aggData) => 
					    AggregateData(aggData.totalSpending + calculatedAgg.totalSpending, 
					                  aggData.numTx + calculatedAgg.numTx, calculatedAgg.windowSpendingAvg)
					  case None => calculatedAgg
					}
			  )
        newTxs.map(tx => EvaluatedSimpleTransaction(tx, tx.amount > 4000))
			}))
```

In Java, this is accomplished more explicitly via the function 3 and function 4 objects. Here we're using the 3 variants, where the first parameter represents the key, the second is an optional value because it has the potential to be empty, and the third is a state object representing the state for the key. The fourth parameter function is there as it provides a time parameter to the front of the parameter list, representing the time of each processing. 

Now you might see why we had to reduce our window of data into one list. Remember, updateStateByKey would provide us with each key's values for that window as one chunk, and that allowed us to easily figure out that window's average. However, since the new method is a map, it passes each iteam separately, making it harder to figure out the overall average for a given time frame. So the easiest solution here is to reduce the values and pass them in as the UpdateStateByKey did. Of course the condensed data must be able to reside in memory on a single worker, but that isn't a concern in thi scenario. With that setup, the core logic doesn't change too much. 

mapWithState's output DStream is special, in that it can also return the current state stream, just as updateStateByKey did via the stateSnapShots method. And now our code is back to what it was before the change to mapWithState, albeit more performant with more possibilities of working with either the state, or the mapped values, or both. 

Now there are some additional areas of interest that were implementd with this new method. Since it takes a spec object, that object has special methods to make your life simple: 

```scala
stateSpecObj.numPartitions(200)
					.partitioner(customPartitioner)
					.initialState(someKeyValueRDD)
					.timeout(Seconds(30))
```
- numPartitions and partitioner for setting those values directly, as well as an initial state method that allows you to provide a RDD that will act as, you guessed it, the initial starting state of the state map. And last, but possibly most interesting is the timeout method, which accepts an ideal duration value. This is useful for session-based state, where you want state to drop out if thre hasn't been any input after this specified period of time. This takes us back to the reason why the input is an option. In the case where state is about to be dropped due to a timeout, it will first go through the map method one more time with an empty value. The state object even has an isTimingOut property to notify you of this. In fact, you'll see that modifying any state that's timing out will yied an exception, as this is more informational in case you want to perform some trigger based on that information. Although it should be noted that both removal and timeouts are related to the frequency of checkpointing and how much space is available. You can think of it closer to garbage collection, where calling remove acts as more of marker for the next clean up, so trusting to the exact timing here should be tested and more often considered fuzzy logic. 

groupByKeyAndWindow 

Sources: 
- Comparison of Apache Streaming Processing Frameworks: Fetr Zapletal
- ...

# Increasing Stream Resiliency 

In th last module, we built up our streaming code to be able to track state across batches while preparing for failure via data and metadata checkpointing. 


