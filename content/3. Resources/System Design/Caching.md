
A cache is just a temp storage that stores recently used data so you can fetch
Accessing data from disk takes 1ms
Accessing data from cache takes 100 ns

Where to cache?
- External Caching - dedicated service like memcache or redis
	- All app servers might share cache
- In-Process Caching 
	- Skips complexity of redis 
	- each app server has its own in memory cache 
	- ultra low latency matters like config data
- CDN
	- geographic distribution of servers 
	- can do edge logic, media, html, videos
- client-side-caching
	- http cache stored in memory on device
	- downside is you have less control
	- not often in interviews
	- forte 
Cache Architecture
1. Cache Aside
	1. Check Cache
	2. Read from Db as fallback
	3. keeps cache lean
	4. downside is that cache miss has latency
	5. most important
2. Write Through
	1. cache writes to cache, then syncs write to db
	2. Write To db
	3. slower writes, pollute cache 
	4. dual write problem 
	5. less common
3. Write behind
	1. Write to cache, then async flush to db
	2. if cache were to crash, big problems
	3. useful for analytics or where data loss is acceptable
4. Read through
	1. Check cache
	2. if we miss cache requests data from db
	3. cache acts as proxy

Cache Eviction Policies
- LRU - evict items that havent been used recently. Most common and balanced
- LFU - evict items that are used least often
- FIFO - evict oldest item first, simple but rarely right choice
- TTL - each time expires after a set time 

Common Issues/ Deep Dives
1) Cache Stampede (Thundering Herd)
	1) When mutliple request try to build cache key, only one should work
	2) cache warming. instead of waiting for popular keys to expire, proactively refresh them
2) Cache Consistency (most common)
	1) no perfect fix
	2) invalidate on write (when value is updated, then update it in cache)
	3) use short ttls
	4) just accept eventual consistency 
3) Hot keys
	1) Taylor Swift key could be receiving lots of reads 
	2) Add instances of caches and shard data across them
	3) then app server can load balance between the caches
	4) Add fallback cache 

Wrap Up
- When Should we bring it up
	- read heavy workload
	- expensive queries
	- high db cpu
	- latency requirements
- How to introduce caching
	- identify bottleneck
	- decide what to cache
	- choose your cache architecture
	- set an eviction policy
	- address the downsides