---
title: Sharding
date: 2025-04-14
tags:
  - seed
enableToc: true
---

# Database Sharding Summary

Sharding is a database architecture strategy that horizontally partitions data across multiple servers or "shards." Each shard contains a subset of the total data and operates as an independent database, allowing the system to distribute load and scale horizontally.

## Key Concepts

- **Horizontal Partitioning**: Data is split based on specific values within a column (shard key)
- **Shard Key Selection**: Critical for even data distribution and query efficiency
- **Distributed Queries**: Queries may need to access multiple shards for complete results
- **Data Rebalancing**: Process of redistributing data when adding or removing shards

## Benefits

- Improves scalability by distributing load across multiple machines
- Enhances performance by reducing index size and contention
- Increases fault tolerance when implemented with proper redundancy
- Allows for geographic distribution of data

## Real-Life Examples

1. **Instagram**: Uses sharding based on user IDs to manage billions of photos and videos across thousands of servers.

2. **MongoDB**: Implements auto-sharding capabilities to distribute data across multiple machines, with automatic load balancing.

3. **Google Bigtable**: Shards data by row keys, enabling Google to handle petabytes of data across thousands of commodity servers.

4. **Uber**: Shards trip data geographically to optimize for local queries and manage their enormous real-time data processing needs.

5. **Pinterest**: Utilizes sharding with MySQL to handle over 100 million active users and billions of pins.

6. **Shopify**: Implements a multi-tenant architecture with sharded databases to support millions of online stores.

7. **GitHub**: Uses multiple MySQL shards to distribute repository data and handle high-volume developer activity.

Each of these implementations tailors sharding strategies to their specific workload patterns, query requirements, and scaling needs.


How to Shard (split) your data?

1) What to shard by?
	1) Choosing a shard key
		1) Good
			1) High Cardinality
			2) Even distribution
			3) aligns with queries
			4) ex. userId, orderId
		2) Bad
			1) Low cardinality
			2) unevenly distributed
			3) queries require scatter-gather
			4) ex. bool flag like isPremium, createdAt eg. social media posts 
	2) Example
2) How to distribute data
	1) Range based -> good if data grows steadily 
	2) Hash based -> hash(key) % numShards
		1) Issues with rebalancing 
			1) Consistent hashing
				1) Places key and shards on virtual ring which effectively eliminates reshuffling 
	3) Directory Based Sharding 
		1) for each user, find the shard they belong to
		2) lookup table 
		3) downside is that every request has extra latency
		4) creates single point of failure
		5) Almost never right answer in sys design
3) Challenges
	1) Hot spots
		1) Celebrity problem
			1) Compound shard key ex. hash(userId) -> hash(userId + createdAt)
			2) Dedicated celebrity shard 
	2) Cross-shard operations 
		1) ex. get popular posts aggreate all posts acorss shards
		2) cache result of expensive queries 
		3) Denormalize data such that queries are quick
			1) copy data across shards so u only need to hit one db but then on writes u need to write to all the shards
	3) Maintaining consistency
		1) Ex. Alice and boba pay each other but live on diff shards
		2) 2 Phase commit
			1) In practice very slow/fragile
		3) Try not to do cross shard transactions
		4) Saga pattern
4) Wrap Up
	1) Storage
	2) Write throughput
	3) Read throughput
	4) Propose a shard key based on your access patterns
	5) Choose your distribution strategy
	6) Call out the trade-offs
	7) Address how you'll handle growth

