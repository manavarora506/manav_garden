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