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