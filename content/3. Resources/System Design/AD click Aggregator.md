An Ad Click aggregator is a system that collects and aggregates data on ad clicks. It is used by advertisers to track the performance of their ads and optimize their campaigns. For our purposes, we will assume these are ads displayed on a website or app like Facebook.

Requirements:
- Functional Requirements:
	- Users click on an ad and get redirected to the advertisers website
	- Advertisers can query click metrics over time with 1 minute min. granularity
- Out-of-scope
	- Ad targeting and serving
	- Cross device tracking
	- Integration with offline channels
- Non-Functional Requirements:
	- 10M ads at any given time
	- 10k ad clicks per second at peak
	- Scalable to support peak 10k cps
	- Low latency analytic queries < 1s
	- Fault Tolerance
	- Data integrity
	- As realtime as possible
	- Idempotency of ad clicks
- Out of scope:
	- Spam detection
	- Demographic profiling
	- Conversion tracking 

System Interface and Data Flow:
- Input:
	- Click Data
	- Advertiser queries
- Outputs:
	- Redirection
	- Aggregated click metrics
- Data flow:
	- Click Data comes into the system
	- User is redirected
	- Click data validated 
	- Click data logged
	- Click data aggregated 
	- Aggregated data queried 

High-level Design:
- Browser -> click processor Service ->getRedirectUrl(adId) return 302 direct
- Browser -> Ad placement service -> Ad DB (AdId, RedirectURI, metadata)
- Ad blockers could prevent click metrics
- Ad service only returns ad id so that we hit processor service which will log metrics and then get redirectURI
- Need to store click metrics (persist)
- Click processor service -> click DB (eventId, adId, userId, timestamp)
- Advertiser Browser -> Query Service -> click DB
- Click DB needs to be good for a lot of writes (cassandra not good for range queries or aggregation) Will not satisfy non-functional too slow
- Need to aggregate data so it is read optimized
	- Use spark
	- Have map-reduce job to aggregate data at minute intervals and then write to OLAP data
	- Advertiser Browser -> Query Service -> OLAP DB (pulling from spark)
	- Chron Scheduler that kicks of spark every 5 min to aggregate data and then write to OLAP (AdId, Minute, Total Clicks)
	- 10k tps can be handled on most modern dbs
	- But we should separate read and write heavy workloads to reduce contention and resource competition
	- Also ensures fault isolation 

Deep Dives:
- How can we address latency issues
	- Spark will not cut it anymore because it will take x min to run aggregation. Even if we run job more frequently there will be a lot of overhead
	- Use Kafka/Kineses (event stream)
	- Click processor service -> event stream -> Flink (stream aggregator) AdId, minute, count
	- In flink you can specify aggregation window and once window complete you can write data to OLAP so now we can read data per minute
	- Aggregation window: 1 minute, Flush intervals (crucial for latency sensitive systems)
- How can we support 10k clicks/second
	- Scale processor service
	- Add a load balancer/API gateway in front of services
	- Need stream to be able to scale. Kinesis has limit of 100 records/second
		- Shard by adId
	- Shard OLAP
- What to do with a hot key in an event stream?
	- Partition by adding to shard key some number 1-n
	- Hot shards: AddId:0-N
- What to do for fault tolerance and data integrity
	- Stream is always available so no issues there
	- What to do if flink jobs fail
	- You can add retention policies so if flink goes down, once it is alive again it can pick up where it left off
	- You can add checkpoints in flink but if aggregation windows are so small it doesn't make much sense
	- Enable kinesis to dump events to s3 and then have spark (map reduce) with chron job and reconciliation worker that will help with fixing potentially incorrect records
- Idempotency (prevent mass clicks by 1 user)
	- generate an ad impression id token in the ad placement service
	- when user clicks on ad send ad impression id as well
	- Have redis cache to store ad impression id, userId and if user clicks on it again then drop it
	- Issue then again user could keep sending different adImpressionId that might be illegitimate. So to fix issue, we can send ad id, user id, and signed ad impression id with timestamp. Then first thing we can do is verify signature and then check in redis cache