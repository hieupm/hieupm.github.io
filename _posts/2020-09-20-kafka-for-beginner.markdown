---
layout: post
title: Apache Kafka Series - Learn Apache Kafka for Beginners
date: 2020-09-20 2:32:20 +0200
description:  # Add post description (optional)
fig-caption: # Add figcaption (optional)
tags: [kafka, mooc, sum-up]
---

This is a very quick summary about Kafka mainly based on Stéphane Maarek'course ([Apache Kafka Series - Learn Apache Kafka for Beginners](https://www.udemy.com/course/apache-kafka/))


## Why Apache Kafka?
- **Multiple Producers**: Kafka is able to seamlessly handle multiple producers, whether those clients are using many topics or the same topic. It allows to aggregate data from many frontend systems and make it consistent.  
- **Multiple Consumers**: Kafka is designed for multiple consumers to read any single stream of messages without interfering with each other. Multiple Kafka consumers can choose to operate as part of a group and share a stream, assuring that the entire group processes a given message only once. 
- **Disk-Based Retention**: Messages are committed to disk, and will be stored with configurable retention rules. 
- **Scalable**: Kafka's flexible scalability makes it easy to handle any amount of data. A cluster of multiple brokers can handle the failure of an individual broker. Replication factors option help Kafka to handle more simultaneous failures. 
- High Performance

## Topics, Partitions and Offsets
- **Topics**: a particular stream of data. A topic is identified by its name. 
- Topics are split into **partitions**
    - Need to specify "how many" partitions while creating a topic, this number can be changed later. 
    - Each message within a partition gets an incremetal id, called **offset**. It's infinite and unbounded.  
    - Messages are ordered within the partition but not across other partitions
  ![Topics, Partitions and Offsets]({{site.baseurl}}/assets/img/2020-09-20-kafka-for-beginner/20200920-1.png "Topics, Partitions and Offsets"){: .align-center}
- Data is kept only for a limited time (default is one week)
- Once the data is written to a partition, it can't be changed  (**immutability**)
- Data is sent randomly over partitiions if key is not provided

## Brocker and Topics
- A Kafka cluster is composed of multiple brokers (servers);
- Each broker is identified with its ID (integer) and contains certain (replicated) topic partitions. Partitions are distributed across all the brokers;
- After connecting to any broker (called a bootstrap broker), you will be connected to the entire cluster. For recommandation, a good number to get started is 3 brokers;

## Topic Replication
- **Topic replication factor** must be defined at the creation of topic and should more than 1 (usually between 2 and 3). If a broker is down, another broker can serve the data;
- At any time, only ONE broker can be a **leader** for a given partition. Only the leader can receive data for a partition. The other brokers will synchronize the data. Each partition has one leader and multiple **ISR (in-sync replica)** participant;
- **Zookeeper** decides leaders and ISR in cluster:
    - the **re-election** will be executed when the leader comes down; 
    - the leader is **re-elected** <span style="color:red">**(?)**</span> when it comes back;

## Producers and Message Keys
- Producers write data to topics (which is made of partitions);
- The keys used by kafka allows producers to know to which broker and partition to write to, so that they can put all the messages with the **same key on one partition**. Thanks to a mathematical process Hash(key)%partitionNb, we can guarantee that one key will always be in the same partition (as long as you don’t change the partition number); 
  - if the number of partition has been changed during the life of the topic, a key will be put in a different partition than before the changing of partition number.
- In case of broker-failures, producers will automatically recover by choosing to receive or not **acknowledgment of data writing operation**;
  - acks = 0 (no acks)
  - ack = 1 (leader acks)
  - acks = all 
    - used in conjunction with min.insync.replicas
    - Exception in case of not enough replicas: NOT_ENOUGH_REPLICAS
- The load is balanced to many brokers hosting the number of partitions. If data is sent without a key, it will be sent round robin across the brokers;

## Producer retries
- In case of transient failure, developpers are expected to handle exceptions, otherwise the data will be lost. Example: NotEnoughReplicasException;
- In case of retries, by default there is a chance that messages will be sent out of order (if a batch has failed to be sent). If you rely on key-based ordering, that can be an issue;
- You can set the setting while controls how many requests re-produced can be made in parallel in any partition: **max.in.flight.requests.per.connection**, default is 5;

## Consummers & Consumer Groups
- Consumers read data from a topic (identified by name);
- Consumers know which broker to read from <span style="color:red">**(?)**</span> and it can also read from multiple partitions;
- In case of broker-failures, consumers know how to recover <span style="color:red">**(?)**</span>;
- Data is read in order **within each partition**. In case of reading from multiple partitions, there is no gurantee about the order betwen partitions. The consumer read data in parallel. 
- Consumers read data in **consumer groups**. Each consumer within a group reads from exclusive partitions. If you have more consumers than partitions, some consumers will be inactive. The inactive consumer **become active** in case of failure/crash of other consumer in the same group. 
  - a **GroupCoordiantor** and a **ConsumerCoordiantor** are automatically used to assign a consumer to a partition. It's a mechanism already implemented in Kafka. 

## Consumer Offsets & Delivery Semantics
- Kafka stores the offsets at which a consumer group has been reading (like checkpointing or bookmarking)
- The offsets committed live in a Kafka topic named (**__consumer_offsets**)
- When a consumer in a group has processed data received from Kafka, it should be committing the offsets. It's an action (done automatically for you) of writing through the topic named __consumer_offsets. 
- If a consumer (group) dies, it will be able to read back from where it left off thanks to the committed consumer offsets. 
- **Delivery semantics** for consumers: consumers choose when to commit offsets. There are 3 delivery semantics: 
  - **At most once**: offsets are committed as soon as the message is received. If the processing goes wrong, the message will be **lost** (it won't be read again). 
  - **At least once**: Offsets are committed after the message is processed. If the processing goes wrong, the message will be read again. This can result in duplicate processing of messages. Make sure your processing is *idempotent* (i.e. processing again the messages won't impact your systems)
  - **Exactly once**: Can be achived for Kafka to Kafka workflows using Kafka Streams API. 
    - For Kafka to External System Workflows, use an idempotent consumer to make sure there's no duplicates in the final database. 

## Kafka Broker Discovery
- Every Kafka broker is also called a "bootstrap server". That means that **you only need to connect to one broker**, and you will be connected to the entire cluster. 
- Each broker knows about all brokers, topics and partitions (metadata). The mechanism is already implemented in Kafka. 

	![Kafka Broker Discovery]({{site.baseurl}}/assets/img/2020-09-20-kafka-for-beginner/20200920-2.png "Kafka Broker Discovery"){: .align-center}

## Zookeeper
- Zookeeper manages brokers (keeps a list of them)
- Zookeeper helps in performing leader election for partitions. 
- Zookeeper sends notifications to Kafka in case of changes (new topic, broker dies, broker comes up, delete topics,...)
- Kafka can't work without Zookeper
- Zookeeper by design operates with an odd number of servers (brokers). 
- Zookeeper has a leader (handle writes) the rest of the servers are followers (handle reads) **(?)**
- Zookeeper is completely isolated from the consumers

## Kafka Guarantee
- Messages are appended to a topic-partition in the order they are sent
- Consumers read messages in the order stored in a topic-partition
- With a replication factor of N, producers and consumers can tolerate up to N-1 brokers being down. 
- Replication factor of 3 is a good idea:
  - Allows for one broker to be taken down for maintenance
  - Allows for another broker to be taken down unexpectedly
- As long as the number of partitions remains constant for a topic (no new partitions), the same key will always go to the same partition. 

## Get started with Kafka
- Check if java version is java 8
  - In case not, you have to upgrade your java version to 8
  ```shell
  # Install brew if needed:
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  // install java 8 using brew
  brew tap caskroom/versions
  brew cask install java8
  ```
- Download Kafka lastest version from https://kafka.apache.org/downloads 
  ```shell
  # Move tar file to install folder 
  mv /Users/[user name]/Download/kafka_2.12-2.2.0.tar /Users/[user name]/Documents/install
  # Unzip file
  tar -xvf kafka_2.12-2.2.0.tar
  ```
- Add Kafka directory to bash_profile file and "source" the file
  ```shell
  export PATH="$PATH:/Users/[user name]/Documents/install/kafka_2.12-2.2.0/bin"
  ```
- Make Zookeeper data directory
  ```
  mkdir data
  mkdir data/zookeeper

  # Edit config/zookeeper.properties
  # change line to 
  # dataDir=/your/path/to/data/zookeeper
  dataDir=/Users/[user name]/Documents/install/kafka_2.12-2.2.0/data/zookeeper
  ```
- Start Zookeeper
  ```
  zookeeper-server-start.sh [kafka installed directory]/config/zookeeper.properties
  # Zookeeper will be started at *0.0.0.0:2181*
  ```
- Open the second terminal window and make the Kafka data directory
  ```
  # create Kafka data directory
  mkdir data/kafka
  # Edit config/server.properties
  # change line to 
  # log.dirs=/your/path/to/data/kafka
  log.dirs=/Users/minh-hieu.pham/Documents/install/kafka_2.12-2.2.0/data/kafka
  ```
- Start Kafka
  ```
  kafka-server-start.sh [kafka installed directory]/config/server.properties
  ```

## Kafka - CLI (Command Line Interface)

- Create a topic
 
```
kafka-topics.sh --zookeeper 127.0.0.1:2181 \
  --create \
  --topic first_topic \
  --partitions 3 \
  --replication-factor 1
```
- In Kafka, you cannot create a topic with replication factor greater than the number of brokers you have.
- Topic operations

```bash
kafka-topics.sh --zookeeper 127.0.0.1:2181 --list 
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic first_topic --describe
#Topic:first_topic       PartitionCount:3        ReplicationFactor:1     Configs:
#        Topic: first_topic      Partition: 0    Leader: 0       Replicas: 0     Isr: 0
#        Topic: first_topic      Partition: 1    Leader: 0       Replicas: 0     Isr: 0
#        Topic: first_topic      Partition: 2    Leader: 0       Replicas: 0     Isr: 0
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic first_topic --delete
```

## Kafka Consoles Producer CLI
- Producing

```bash
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic first_topic
#>Hello
#>Bonjour 
#>Je m'appelle Minh-Hieu PHAM
#>juste un autre message 
#>
```
- Producing with properties

```bash
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic first_topic --producer-property acks=all
```
- If you send message to a topic not exist yet, you get the warning:

```bash
[2019-05-08 15:54:53,508] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 3 : {new_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
```
- The topic gets created with default properties (partition 1, replication factor 1,...) but there was no leader election that happened yet, so you get this exception LEADER_NOT_AVAILABLE. 
- The producer just tried and waited until the leader was available, and it produced another message. 
- You can change the default properties by editing the *config/server.properties*
> num.partitions=3


## Kafka Consoles Producer CLI
- Consuming from a topic: 

```bash
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic
```
- By default, a consumer only reads from that point when you launch it and will only intercept the new message. 
- Consuming from beginning of the topic:

```bash
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --from-beginning
```
- The order of messages in this consumer is not "total", the order is per partition. The "first_topic" was created with 3 partitions. 

## Kafka Consumers in Group
- The consumer groups rebalanced and shared the load between consumers. 

```bash
# start one consumer
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --group my-first-application --from-beginning

# start one producer and start producing
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic first_topic

# start another consumer part of the same group. See messages being spread
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group my-first-application
```

- When a consumer in group went down, the new partitions have been reassigned to another consumer available in group. 
- When the group is specified, the offsets will be committed in Kafka.
- Some Kafka consumer groups operations
  
```bash
# list consumer groups
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
 
# describe one specific group
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-second-application

# describe another group
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
#Consumer group 'my-first-application' has no active members.

#TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
#first_topic     0          2               2               0               -               -               -
#first_topic     1          1               1               0               -               -               -
#first_topic     2          1               1               0               -               -               -

# start a consumer in another terminal
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --group my-first-application

# describe the group now
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
#TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
#first_topic     0          2               2               0               consumer-1-2868a5d6-9dd7-4c37-b7d9-67818eafd298 /192.168.0.18   consumer-1
#first_topic     1          1               1               0               consumer-1-2868a5d6-9dd7-4c37-b7d9-67818eafd298 /192.168.0.18   consumer-1
#first_topic     2          1               1               0               consumer-1-2868a5d6-9dd7-4c37-b7d9-67818eafd298 /192.168.0.18   consumer-1
``` 
- Reset the offset of a consumer group
    - Assignments can only be reset if the group 'my-first-application' is inactive. You need to desactivate all consumers in group befor assignment. 
  
    ```bash
    kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --to-earliest --execute --topic first_topic
    #TOPIC                          PARTITION  NEW-OFFSET     
    #first_topic                    0          0              
    #first_topic                    2          0              
    #first_topic                    1          0  
    # describe the group again
    kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application
    #TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
    #first_topic     0          0               2               2               -               -               -
    #first_topic     1          0               1               1               -               -               -
    #first_topic     2          0               1               1               -               -               -
    ```
    - Shift forward and backward 
  
    ```bash
    # shift offsets by 2 (forward) as another strategy
    kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --shift-by 1 --execute --topic first_topic

    # shift offsets by 2 (backward) as another strategy
    kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --shift-by -1 --execute --topic first_topic
    ```

## Client Bi-Directional Compability
- As of Kafka 0.10.2, your clients & Kafka Brokers have a capability called bi-directional compability (because API calls are now versioned)
    - an OLDER client can talk to a NEWER broker
    - a NEWER client can talk to an OLDER broker
- *NOTE*: always use the latest client library version if you can
([Ref](https://www.confluent.io/blog/upgrading-apache-kafka-clients-just-got-easier/))

## Use cases
### Activity tracking
A website's users interact with frontend applications, which generte messages regarding actions the user is taking. The messages are published to one or more topics, which are then consumed by applications on the backend. These applications maybe generating reports, feeding machine learning systems, updating results, or performing other operations that are necessary to provide a rich user experience.

### Messaging
Applications can produce messages without needing to be concerned about formatting or how the messages will actually be sent. A single application can then read all the message to be sent and handle them consistently: 
- Formatting the messages using a common look and feel. 
- Collecting multiple messages into a single notification to be sent. 
- Applying a user's preferences for how they want to receive messages. 
Using a single application avoids the need to duplicate functionality in multiple applications, as well as allows operations like aggregation which would not otherwise be possible. 

### Metrics and logging. 
Kafka is also ideal for collecting application and system metrics and logs which applications producing the same type of message shines. Applications publish metrics on a regular basis to a Kafka topic, and those metrics an be consumed for monitoring and alerting. 
- Log messages can be published in the same way and can be routed to dedicated log search systems like Elasticsearch or security analysis applications. 
- There is no need to alter the frontend applications or the means of aggregation when the destination system needs to change. 

### Commit log
Database changes can be published to Kafka and applications can easily monitor this stream to receive live updates as they happen:
- Replicating database updates to a remote system
- Consolidating changes from multiple applications into a single database view. 
- Durable retention is useful here for providing a buffer for the changelog, it can be replayed in the event of a failure of the consuming applications. Log-compacted topics can be used to provide longer retention by only retaining a single change per key. 

### Stream processing
Streaming processing operates on data in real time, as quickly as messages are produced. Stream framworks allow users to write small applications to operate on Kafka messages, performing tasks such as counting metrics, partionning messages for efficient processing by other applications, or transforming messages using data from multiple sources. 