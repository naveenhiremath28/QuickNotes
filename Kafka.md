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
