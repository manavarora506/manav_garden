---
title: Relational Databases
date: 2023-12-20
tags:
  - seed
enableToc: true
---
# What is a Relational Database
Sources:
- https://ibm.com/topics/relational-databases
- https://www.ibm.com/blog/sql-vs-nosql/
- https://www.youtube.com/watch?v=OqjJjpjDRLc&ab_channel=IBMTechnology
- https://www.youtube.com/watch?v=E9AgJnsEvG4&ab_channel=IBMTechnology

It organizes data into rows and columns, which collectively form a table. Data is typically structured across multiple tables, which can be joined together via a primary key or a foreign key. 

**ACID: 
- **Atomicity**: all changes to data are performed as if they are a single operation. That is, all the changes are performed, or none of them are.
- **Consistency**: Data remains in a consistent state from start to finish, reinforcing data integrity
- **Isolation**: the intermediate state of a transaction is not visible to other transactions, and as a result, transactions that run concurrently appear to be serialized
- **Durability**: after the successful completion of a transaction, changes to data persist and are not undone, even in the event of a system failure
# Relational vs non-relational databases
- **Key-value store**: schema-less model is organized into a dictionary of key-value pairs, where each item has a key and a value. Commonly used for caching and storing user session information, but it is not ideal when you need to pull multiple records at a time. 
- **Document store**: stores data as documents. Helpful with semi-structured data and it keeps the data together when it is used in applications, reducing the amount of translation needed to use the data. More flexible because data schemas do not need to match across documents. Can be problematic for complex transactions, leading to data corruption. 
- **Wide-column store**: this DB stores info in columns, enabling users to access only the specific columns they need without allocating additional memory on irrelevant data. It can be more complex to use so it is not recommended for use in newer teams and projects. 
- **Graph Store**: Houses data from a knowledge graph. Any object, place, or person can be a node and an edge defines the relationship between the nodes. 

NoSQL databases prioritize availability over consistency. Relational DBs ensure that information is always in-sync and consistent. 

## Benefits of relational databases
The primary benefit is the ability to create meaningful information by joining the tables. 
- Ease of use: there is more of a community around relational databases
- Reduced redundancy: normalization and access control
- Ease of backup and disaster recovery: transactional thus ensure system is consistent at any moment. Offer easy ways to export and import, making backup and restore trivial

### How SQL Works
- Scalability: db's can scale vertically, increasing the load on a server by migrating to a larger server that adds more CPU, RAM, or SSD capability. Not usually scaled horizontally
- Structure: schema organizes data in relational, tabular ways, using tables with columns or attributes and rows of records. Requires organizing and structuring data before starting with the db
- properties: ACID
- examples: MySQL,  PostgreSQL, YugabyteDB

### How NoSQL Woks
Less of a need to pre-plan and pre-organize data. Allows you to add new attributes and fields, as well as use varied syntax across databases.
- Scalability: scale better horizontally with additional servers or nodes to increase the load
- Structure: column oriented, key-value stores, document stores, graph databases 
- Properties: follows CAP theorem, only guarantees two of the three:
	- Consistency: every request receives either the most recent result or an error
	- Availability: every request has a non-error result
	- Partition tolerance: Any delays or losses between nodes do not interrupt the system operation
- examples: Redis, MongoDB, Cassandra, Elasticsearch, HBase

### When to use SQL vs NoSQL
**When to use SQL
- Use when working with related data
- efficient, flexible, and easily accessed by an app
- best for assessing data integrity
- A benefit of a relational database is that when one user updates a specific record, every instance of the database automatically refreshes, and that information is provided in real-time
**When to use NoSQL
- Good when availability is important
- Good when a company will need to scale because of changing requirements
- good when there are large amounts of (or ever-changing) data sets
- if there is a lot of unstructured data, then document dbs like MongoDB and CouchDB are good
- For quick access to a key value store with strong integrity, Redis is the best choice 
- When complex search is required then Elastic search is a good choice
- Need to horizontally scale

[[POSTGRESQL]]