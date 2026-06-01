kafka is like distributed log file

broker = server in kafka contains n partitions (where actual events/message store)

partition = physical distribution of data, its an ordered log

order is maintained only inside individual partition, even though topic has multiple partitions, oder is not maintained between the partitions

topic = logical partitioning/grouping and topic will contained related events

topic can contain multiple partitions but partition can be part of single topic 

kafka has multiple brokers, to avoid single point of failure. and also each partition has replicas, for fault tolerance

producers - who produce message to kafka

consumers - one who consume messages from kafka

consumer groups - group of consumers, where each consumer can consume from 1 or multiple partitions but rest other consumer wont consume message from the partitions which are already taken care by other consumer
eg. consider cricbuzz. all the employees are not intended to capture events from same partition/a cricket match. where each one should capture different match

consumers cannot exist without consumer group, even though if we create one consumer by default kafka creates one consumer group for it 
eg. if a person is employee means he should be working in one company

kafka can be served message queue as well as event stream

message queue is like a task, once the message consumed by consumer it will be removed. similar to task in the list, once task finished. will remove it

events are like log file. data will be stored until the retention period no matter whether it is consumed or not

offset = monotonically increasing number that uniquely identifies a record within a partition

offsets stored in  `_consumer_offset`
offset is maintained for per consumer per partition
How consumers consume from events from kafka -> Polling

kafka is not a log file, its a distributed log file

what if wrong data goes into kafka? -> 
1. we add a new event which corrects it since we cannot update
2. validate well before pushing to kafka like schema validation, required fields or business logics
3. **Dead Letter Queue (DLQ)** if consumer unable to process event we will put into dlq topic

partition replications (replication-factor):  copy of partitions
Leader Broker: read and write happens from this
Follower Broker: only maintains copy from leader  (take over if leader fails)

consider, partition 3 replication factor 2
L -> Leader F -> Follower
P0:
	L -> B1
	F -> B2

P1:
	L -> B2
	F -> B3

P2:
	L -> B3
	F -> B1

how does kafka knows which is leader and which is follower -> its from **kafka metadata** which is maintained by **kafka controller**

older kafka -> kafka controller is zookeeper
new kafka -> kafka controller is kraft, internal consensus

## Complete end-to-end Kafka flow
```


STEP 1: Producer startup
Producer does not blindly send data.

Producer connects to any broker
   Producer → Broker X

Asks:
   "Give me cluster metadata"

------------------------------------------------

STEP 2: Producer receives metadata

Producer now knows:
   • all brokers
   • all topics
   • partitions
   • leader broker for each partition

Producer caches this info.

------------------------------------------------

STEP 3: Producer wants to send an event

Producer now has to answer:
   Which partition should this message go to?

This is producer's responsibility.

STEP 4: How producer chooses partition

Case 1: Key is present

   Plain text
   partition = hash(key) % number_of_partitions

Example:
   Plain text
   key = orderId = 123
   hash(123) % 3 = partition 1

Guarantee:
   • same key → same partition
   • ordering preserved

------------------------------------------------

STEP 5: Producer finds leader broker

Using metadata:
   Plain text
   partition 1 → leader broker 2

Producer sends message directly to broker 2.

No load balancer.
No proxy.

------------------------------------------------

STEP 6: Leader broker writes data

Leader:
   • appends record to partition log
   • assigns offset
   • starts replication to followers

------------------------------------------------

STEP 7: Followers replicate

Followers:
   • pull data from leader
   • append to their own logs
   • send ack

If acks=all:
   • leader waits for ISR
   • then acknowledges producer

------------------------------------------------

STEP 8: Consumer side (brief)

Consumers:
   • get metadata
   • know partition → leader
   • pull data
   • track offsets

Same metadata-driven routing.
```

Commit Statergy:
1. Auto Commit 
	1. kafka auto commits when message is read
	2. bit risky, consumer fail to process as per business logic but kafka already committed it
2. Manual Commit
	1. consumer will manually commit after message processed successfully

Delivery Guarentees:
1. At-most-once
	1. commit before processing
2. At-least-once (Default)
	1. commit after pocessing
3. Exactly-once
	1. Kafka transaction - rollbacks everything if it fails

How consumer know event arrived: It polls periodically
kafka doent support push based approach instead pull based approach for scalability - > means consumers will poll periodically

Note:
Kafka prefers data duplication instead data loss
eg. when we go with default Guarentee strategy (at-least-once), if consumer crashed to process at some point while processing offset 100-110, once it restarted it again starts from 100 it may cause duplication, so we need to handle it(which is better than data loss)


Can offset go backward -> NO, but manually seek `consumer seek(partition, offset)`

Who assigns offset for partition -> broker (leader of partition)
Producers do NOT control offsets.
Offsets are purely a broker responsibility.

What exactly is stored in __consumer_offsets?

Each record in _consumer_offsets contains:
(groupId, topic, partition) → committedoffset

eg:
(payment-service, orders, PO) -> 125
(payment-service, orders, P1) -> 78

Backpressure is:

A mechanism that prevents a fast producer (or broker) from overwhelming a slow consumer.
In Kafka context:
- ﻿﻿Producer produces fast
- ﻿﻿Consumer processes slow
- ﻿﻿System must slow down intake instead of crashing

Without backpressure:
- ﻿﻿Memory fills up  
- Threads block
- ﻿﻿JVM crashes  
- Data loss or cascading

What happens internally when topic is created?
- Kafka controller:
- ﻿﻿Creates metadata
- ﻿﻿Assig partitions to brokers
- ﻿﻿Assigns leader & followers

Metadata stored in Kafka's metadata store (ZooKeeper earlier, KRaft now)

How to add consumer group to existing topic?
```
group.id = "payment-service"
consumer subscribe("orders")
```


Why is Kafka so good at handling massive writes?

1 Append-only log
- ﻿﻿No random writes
- ﻿﻿Always append at end
- ﻿﻿Sequential disk writes (FAST)

2 OS Page Cache
- ﻿﻿Kafka relies on OS cache
- ﻿﻿Writes hit memory first
- ﻿﻿Disk flush happens asynchronously
Zero-copy optimizations

3 Batching
Producers send batches, not single messages:
- ﻿﻿Fewer syscalls
- ﻿﻿Better compression
- ﻿﻿Higher throughput

4 Partitioning
Writes are spread across:
- ﻿﻿Partitions
- ﻿﻿Brokers
- ﻿﻿Disks
Parallelism everywhere

5 Minimal broker logic
Kafka broker does NOT:
- ﻿﻿Deserialize messages
- ﻿﻿Apply business logic
- ﻿﻿Filter per consumer


Is Kafka message queue or event stream? -> both