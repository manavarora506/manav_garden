What is Apache Kafka?

Linkedin 2011

- Extremely scalable
- Message bus
- For example, brad pit making instagram posts
- Good for distributed event streaming 
- Good for event based systems
- Producers -> apps that send messages to various partitions for a particular topic
- Topics -> split into partitions
- Why not just let producers send msgs immediately to consumers? B/c not scalable
- Guarantee is whenever producer sends msg to partition it will be ordered within partition.
- Partition and topics lie within a broker which is basically a kafka server
- Consumers can pull on-demand, at their own pace
- Producers produce messages using clients
- You can have multiple producers, servers, consumers which requires a lot of bandwidth
- Keep replicas of each partition 


1) Motivating Example
	- World Cup
		- Producer will put all events on the queue like sub Brazil, goal by Neymar
		- Consumers read from the queue
		- Problem #1 Too many events on the queue and the server hosting queue running out of space
		- Solution #1 Scale horizontally (more servers), issue is events then are not read in order
		- Problem #2 Now events are not processed in the correct order
		- Solution #2 Partition by game now all games are separated
		- We need to distribution/partition by user
		- Problem #3 Consumer can't keep up with the rate of new events
		- Solution #3 Consumer groups -> each event processed by 1 consumer in the group 
		- Introduce topics where each event associated with topic and each consumer subscribed to a topic 
2) Overview
	- Broker: The servers (physical or virtual) that hold the queue
	- Partition: The 'queue'. An ordered immutable sequence of messages that we append to like a log file. Each broker can multiple partitions. Exists physically. Way of scaling data
	- Topic: A logical grouping of partitions. You publish and consume from topics in kafka. Exists in code.
	- Producer: write messages/records to topics
	- Consumer: read messages/records off of topics
	- Look under the hood
		- Producer create and publishes a message/record structure
		- Message will heave headers, key, value, timestamp
		- You can add a record via CLI
		- Kafka then assigns message to correct topic, broker, partition
		- Message then appended to partition via append only log file
		- Consumers read next message based on offset
		- Each partition has a leader and replicated

3) When to use it
	1) Anytime you need a message queue
		- Processing can be done async i.e. Youtube transcoding 
		- in-order message processing i.e. Ticketmaster waiting queue
		- Decouple producer and consumer so they can scale independently i.e. Leetcode or Online Judge 
	2) Anytime you need a stream
		- Need to process a lot of data in real time ie. Ad click aggregator 
		- Stream of messages need to be processed by multiple consumers simultaneously i.e. Messenger or FB live comments
4) What you should know for deep dives
	1) Scalability
		- Constraints:
			- Aim for < 1MB per message
			- One broker up to 1 TB data & 10k messages per second
		- How to scale?
			- More brokers
			- Choose a good partition key
		- How to handle a hot partition?
			- Remove the key (would work if order does not matter)
			- Compound the key - AdId:1-10 or AdId:UserId
			- Backpressure - slow down the producer 
	2) Fault tolerance & Durability
		- Relevant settings:
				- acks(acks=all)
				- replication factor (3 is default)
		- What happens when a consumer goes down (kafka will not go down)?
			- Just read the latest offset
			- If a consumer group, rebalance
	3) Errors & retries
	4) Performance Optimizations
		1) Batch messages in producer
		2) Compress messages in producer
		3) Partition key is where you should start
	5) Retention Policies
		1) Two settings:
			1) Retention.ms (default 7 days) - how long to keep message
			2) Retention.bytes (default 1GB) when to start purging based on size