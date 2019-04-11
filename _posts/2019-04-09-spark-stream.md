---
published: true
title: "Spark - Processing Data with the Streaming API"
excerpt: "Processing Data with the Streaming API"
toc: true
toc_sticky: true
toc_label: "Processing Data with the Streaming API"
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

![dstream-1](../assets/images/streaming-1.JPG "Streaming example")

The DStream bastraction is itself built on top of RDDs, where data bucket is really just an RDD. In fact, the operations you specify for your DStreams end up translating into operations on each underlying RDD. 

![dstream-2](../assets/images/streaming-2.JPG "Streaming example")

By default, they work **sequentially**, following the order you specified in your application: every increase in the number of ouput jobs translates into an increase in overall batch time. So, if you do have multiple output and are looking to increase your throughput, then you can increase the parallelism via spark. 

If you have more multiple outputs and are looking to increase your throughout, then you can increase the parallelize **spark.streaming.concurrentJobs**

By default, this parameter is set to 1, because it makes batch debugging easier reduces the possibility of resource conflicts: you must try to merge output operations as much as possible before turning up this configuration value. 

The streaming context acts as the central coordinator, and that each input source uses up an entire executor, so that it can continuously receive incoming data. So, how does this all intertwine into the batch model? At the end of each batch interval, the streaming context launches the necessary jobs using its stored Spark context. Tasks are passed to the available executors. 

The input is continuing to be gathered by the receiver in parallel. So at any point in time, Spark can be simultaneously processing the previous intervals batch while the receiver continues to collect new data towards the current interval. Here, in this example, it's the 3 to 4 second data which is being gathered, all while the 2 to 3 second batch is still being analyzed, stored, and then printed to the console. 

![dstream-3](../assets/images/streaming-3.JPG "Streaming example")