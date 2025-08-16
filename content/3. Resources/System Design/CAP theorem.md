---
title: CAPTheorem
date: 2025-04-14
tags:
  - sapling
enableToc: true
---

# CAP Theorem Summary

The CAP theorem is a fundamental principle in distributed computing that states it's impossible for a distributed data store to simultaneously provide more than two of the following three guarantees:

**Consistency**: Every read receives the most recent write or an error. All nodes see the same data at the same time.

**Availability**: Every request receives a non-error response, without guaranteeing it contains the most recent write. The system remains operational even when nodes fail.

**Partition Tolerance**: The system continues to operate despite network partitions (communication breakdowns between nodes).

In practical terms, since network partitions are unavoidable in distributed systems, designers must choose between:

- **CP systems** (consistent and partition-tolerant): These may become unavailable during partitions but maintain consistency. Examples include traditional relational databases with distributed transactions.

- **AP systems** (available and partition-tolerant): These remain available during partitions but may return stale data. Examples include NoSQL databases like Cassandra or DynamoDB.

Many modern distributed systems employ nuanced approaches that make specific trade-offs based on business requirements, implementing different consistency models (strong, eventual, causal) depending on the use case.

What is CAP Theorem?

- You can only have 2/3
- Consistency - all nodes/users see the same data at the same time
- Availability - every request gets a response (successful or not)
- Partition tolerance - system works despite network failures between nodes

Why does this matter?

- Important to define the system characteristics early during non-functional requirements
- They influence your design decisions during deep dives

Network fails between them
1) Stop serving data (consistency)
2) Or risk wrong data (availability)

Examples where we need strong consistency:
1) Ticket booking platform (airline, event, etc)
	- If we sell a ticket, everyone needs to see it as unavailable, without delay
2) Inventory System (Amazon)
	- Can't sell the same last item to multiple customers
3) Financial Systems
	 - Stock trades must be executed in strict order 

If you don't need strong consistency, then choose availability!
1) Social media app
2) Yelp like business review service
3) Streaming service like Netflix

How does it influence my decision?
- Strong consistency
- Implement distributed transactions
- limit to single node
- discuss consensus protocols 
- accept higher latency
- Example tools:
	- PostgresSQL
	- Trad RDMS
	- Spanner
	- NoSQL with strong consistency mode (DynamoDB)

For availability
- use multiple replicas 
- CDC and eventual consistency is ok
- Example tools:
	- DynamoDB (in mult-AZ mode)
	- Cassandra

Advanced!
Different parts of system can have different requirements:

Ticketmaster
- availability for CRUD events
- consistency for booking tickets

Tinder
- availability for viewing profile data
- consistency for matching 

Different types of consistency:
1) Strong consistency: all reads reflect most recent writes
2) Casual consistency: related events appear in order
3) Read-your-writes consistency: user sees their own updates
4) Eventual consistency: updates will propagate eventually