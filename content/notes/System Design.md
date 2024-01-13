---
title: System Design
date: 2023-11-22
tags:
  - sapling
enableToc: true
---
### This page will include all my notes for System Design Prep

[System Design Interview: Design Tiktok](https://www.youtube.com/watch?v=Z-0g_aJL5Fw&ab_channel=Exponent)

**Notes:
- take notes on the requirements
- ask clarifying questions
- Functional requirements
	- upload videos ~30-60s (include text)
	- view feed(videos of followers + recommendations)
	- favorite videos + follow users
- Non-Functional requirements
	- availability (needs to be highly available because of scale)
	- latency(can cache a lot of the content since it's mobile? why?)
	- scale (you can ask how many users per day?) 1 million active users
	- estimates:
		- videos - 5MB per video* 2 = 10MB/day/user
		- User meta data -  1k day/user
- Start with API endpoints
	- uploadVideo
		- POST(video, user) -> returns 200 for success
	- viewFeed
		- GET()
		- We want to preload ahead of time so user isn't waiting for videos to load, but not so much to use up user bandwidth
		- Reddis cache - preload a list of top 10 videos we will load for user before we get to viewFeed
			- precache_service - could run on schedule or demand (connected to read only RDB)
			- how will this reduce latency?
		- system seems very read heavy
			- read worker (read-only DB) to reduce load
	- user_activity
		- deals with followers, liking
		- table schema
			- user_id -> uuid
			- following -> fk to another table (for any one user, we will be following multiple other accounts)
			- likes -> fk to a videos table 
			- meta -> string
- Talk about DataBase schema
	- user_id -> uuid
	- video_link -> url (blob storage?)
	- meta -> string
	- video_id -> uuid
- Relational DB (postgress)
	- why use this? - use it more so with user data objects, linking tables together, more strict, more space and speed efficient for specific queries
	- noSQL - better for unstructured data (more freeform), searching for key values
	- *comment to note: I don't feel as though there was enough thought put into the SQL vs NoSQL discussion. In my opinion based on the non functional requirements presented to us for a highly available system and 1 million active users per day, I think the data scalability should be a huge consideration in that decisions but wasn't even mentioned. Horizontally scaling a SQL database is possible with sharding etc. but can be complicated to do. NoSQL can pretty trivially and cost effectively scale horizontally especially in cloud environments. And the data we are storing related to the videos is just small chunks, we aren't really leveraging a lot of structured data schemas or anything like that. I don't know, I just think I challenge that decision a little bit.
	- uploadVideo API -> video_table(RDB schema from above)
	- uploadVideo API -> BLOB storage(actual video files stored here)
- Follow up questions
	- What are the bottlenecks if we 10X traffic?
		- API endpoints should be behind CDN
		- Load balancers behind API endpoints
			- will help keep system highly available
			- will route traffic accordingly 
		- Use autoscaling group
		- Shard write RDB
			- like a load balancer for DBs
			- could shard by region


[Design FaceBook Messenger](https://www.youtube.com/watch?v=uzeJb7ZjoQ4&ab_channel=Exponent)

**Notes:
- Product goals:
	- real time messaging
	- groups
	- online status
	- media uploads
	- read receipts
	- notifications
- Technical goals:
	- low latency
	- high volume
	- reliable
	- secure
- Chat-Server API
	- connection to users
	- HTTP polling: repeatedly ask server if there is any new information (not right solution bc too many unnecessary requests and high latency)
	- long polling: server holds requests until data is available (better than above bc it helps with latency but we still need to maintain open connection), better for notifications
	- web sockets: (recommended) still maintain open connection (limits to how many connections port can have) thus need multiple servers and a load balancer in front
	- Distributed system
	- API servers need to talk to each other: pub-sub message queue 
	- Message service (message queue): each API server will publish messages to this queue and subscribe to updates 
	- How will we store and persist messages in the DB: need to focus on sharding and partition (not focussing on consistency). Use NoSQL DB bc it has built in replication and sharding (Cassandra)
	- How will we store and model data in DB
		- users table:
			- id int
			- username string
			- lastActive timeStamp
		- messages table:
			- id int
			- user int
			- conversation int
			- text string
			- media_url string
		- conversations
			- id int
			- name string
		- conversation_users
			- conversation int
			- user int
	- How can we scale: use cache 
	- How do we store media: object storage service like s3
		- We can use a CDN to cache
	- How do we notify offline users about messages
		- Message Service will notify Notification Service using some third party mail service API

[Basic System Design for Uber or Lyft](https://www.youtube.com/watch?v=R_agd5qZ26Y&ab_channel=InterviewPen)

**Notes**:
Requirements:
- Rider selects pickup point on map and can view ETA and price
- Rider pays to request a ride
- Drivers and riders are matched. Driver accepts rides.
- Rider receives info about driver, location in real time

- Use an event bus to make structure cleaner (kafken)
- Rider API and Driver API
	- problems:
		- handle large number of users
		- lots of realtime data (HTTP polling, web sockets)
	- we can use load balance and horizontally distribute api across multiple nodes
	- latency on rider side, websockets help here by maintaining persistent connection
- Database
	- Trip - id, driver id, rider id, price, pickup loc, destination, date, status, ride type
	- Driver - id, ride type, car info, loc, phone number, email, password, name
	- Rider - id, payment info, phone number, email, password, name
	- high traffic db so we need to shard by id (high cardinality, low frequency)
- Map
	- problems:
		- serve map images
		- convert street address to lat long
		- get directions
	- we can use map service like google maps
- Payment
	- use existing service like stripe alongside a webhook
- Pricing
	- problems:
		- pricing varies by demand
		- batch processing would be too slow
	- price API - store demand in redis, use spark to query (structured streaming)
- Rides
	- problems:
		- needs to efficiently find drivers in an area
		- need to calc ETA
	- ETA service
	- we didn't know exact loc of driver. need to find driver in some radius. Need to index of area. Like geospatial indexing. 
- Matching
	- when a rider requests a ride find closest driver using rides service. Tell driver API to accept or decline the ride. If accepted, then notify driver and driver gets directions to pickup location. If declines, repeat with next closest driver. 
