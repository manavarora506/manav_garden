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
	- *comment to note: I don't feel as though there was enough thought put into the SQL vs NoSQL discussion. In my opinion based on the non functional requirements presented to us for a highly available system and 1 million active users per day, I think the data scalability should be a huge consideration in that decisions but wasn't even mentioned. Horizontally scaling a SQL database is possible with [[Sharding]] etc. but can be complicated to do. NoSQL can pretty trivially and cost effectively scale horizontally especially in cloud environments. And the data we are storing related to the videos is just small chunks, we aren't really leveraging a lot of structured data schemas or anything like that. I don't know, I just think I challenge that decision a little bit.
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

[Design Calendar Application](https://www.youtube.com/watch?v=39eAITqeu7g&ab_channel=Exponent)

requirements:
 - add events
 - reminders
  - timezone
  - invite users to events

entities:
- calendar
- users
- events
- notification
- notification channel

User table:
id->primary key, serial, uuid
name->string
email -> string

Calendar:
id->primary ket, serial, uuid
name -> string
description -> string
timezone -> timezone

Events:
id -> primary key, serial, uuid
name -> string
description -> string
Start -> Datetime
End -> Datetime
(midnight to midnight considered allDay)
AllDay -> Boolean

Notification:
id: primary key, serial, uuid
effective: DateTime
user: references user.id

Notification Channel
id: primary key, serial, uuid
method: string
value: string
user_id: references user.id

Relationships between these objects:

UserCalendar
user can have many calendars (one to many)
user_id: reference user_id
calendar_id: reference calendarId
why use join table bc you have extra information rather than using a foreign key

CalendarEvent join table
many to many relationship
calendar_id: reference calendar_id
event_id: reference 
role: string

EventNotification join table
event_id:reference event_id
notification_id: reference notification_id

How would design change if we wanted team calendar

Team Table
id

TeamUser

How will this system scale?


[Design Dropbox](https://www.youtube.com/watch?v=_UZ1ngy-kOI&t=1012s&ab_channel=HelloInterview-SWEInterviewPreparation)

Dropbox

Dropbox is a cloud based file storage service that allows users to store and share files. It provides a secure and reliable way to store and access files from anywhere, on any device.

 1) Requirements
	 1) Functional
		 - Upload a file 
		 - download a file
		 - automatically sync files across devices 
	2) Out of scope:
		1) Roll own blob storage
	3) Non_functional requirements
		1) availability >> consistency
		2) low latency upload and downloads (as low as possible)
		3) support large files as big 50gb
			1) resumable uploads 
		4) high data integrity (sync accuracy)
 1) Core-entities
	 1) File (raw bytes)
	 2) File Metadata
	 3) Users
 2) API
	 1) POST /files -> 200, body: File &  FileMetaData
	 3) GET files/:fileID -> File & FileMetadata
	 4) GET /changes?since={timestamp} -> fileIds[] 
 3) High-Level Design
	 1) Client -> LB & API -> UploadFile -> File Service -> upload file to blob storage (returns link to where file is) and after upload complete write metatdata to file Metadata table
	 2) File Metadata fields
		 1) FileId
		 2) File Name
		 3) MimeType (pdf, image, etc)
		 4) Size
		 5) OwnerId
		 6) s3 link
		 7) creation TimeStamp
		 8) update TimeStamp
	4) getFile(FileId) -> get file from s3, get metadata, then get link and download from s3
	5) Client side
		1) Client APP
		2) Local Folder
		3) local DB
	6) Remote changes
		1) Pull for changes
		2) download new file and replace
	7) Local change
		1) upload files to remote
		2) WindowsAPI
			1) FileSystemWatcher
		3) MacOS API:
			1) FSEvents
		4) Just call upload if any changes 
	8) Sync Service
		1) getChanges
		2)  returns metadata of any files that changed
 4) Deep-Dives
	 1) Upload large files: 50 gb
		 1) Web browsers have posy body size limits eg. 10 mb

[Design Ticketmaster](https://www.youtube.com/watch?v=fhdPyoO6aXI&ab_channel=HelloInterview-SWEInterviewPreparation)

Functional Requirements:
1) book tickets
2) view an event
3) search for events

NonFunctional Requirements:
1) Ask [[CAPTheorem]] Availability or Consistency 
1) Strong consistency for booking tickets and high availability for search and viewing events
2) read >> write
3) Scalability to handle surges from popular events 
4) Low-latency search

Out of scope:
- GDPR compliance
- fault tolerance
- etc

Ask interviewer if you need to reprioritize and tell them what you will prioritize

Core Entities (detail key fields and columns later)
- Event
- Venue
- Performer 
- Ticket

User-facing API's

GET /event/:eventID -> Event & Venue & Performer & Ticket[]
GET /search?terrm={term}&location={location}&type={type}&date={date} -> Partial<Event>[]

POST /booking/reserve body:{ticketId} -> don't put userId here as it could be security issue
instead use header: JWT | sessionToken (for user info)

PUT /booking/confirm body:{ticketId, paymentDetails (stripe)} -> don't put userId here as it could be security issue
instead use header: JWT | sessionToken (for user info)

High-Level Design

- Client
- API Gateway (take incoming API requests and route to correct server, authentication, rate-limiting, routing)
- To view event: Hit API gateway -> go to Event CRUD service -> Event Database (eventID, venueID, perfomerId, tickets[], name, description), Venue (id, location, seatMap), Performer (id), Ticket(id, eventId, seat, price, status: available, reserved, booked, reservedTimestamp)
-There are relationships, need consistency, use postgress SQL, but NoSQL would be fine too. In interview SQL vs NoSQL is old debate.
- Search Service: SELECT * From DB WHERE type in [] AND name like '%term%' but super slow b/c of wildcards
- Booking service: reserve(ticketID) -> update DB such that we update ticketId in ticket table's status field to reserved, available, booked. Then we make confirm(ticketId, paymentDetails) request. You can usually abstract Stripe/payment away. Stripe will handle request asynchronously. It will respond back with callback URL. Have some endpoint in booking service that it will callback too. If response is successful, then update ticket's status to booked. 
- Have a [[CRONJob]] that queries tickets db for tickets with reserved status and if timestamp > 10 min, set status back to available 
- Need something a bit more in real time. Get rid of chronjob and timestamp. Use ticket Lock (redis). When ticket is reserved, add to redis, {ticketId: true} TTL 10
- When user tries to reserve ticket, don't write to DB, instead lock ticket. Use distributed lock because there will be multiple instances of booking service. 
- What would happen if lock does down? 

Deep Dives

Goal is to find 1-3 places where you can go in depth. 
Reference Non-functional requirements and find what is missing. 

Improve search:
- Use Elastic Search: builds inverted index to make searching documents by terms really quickly
- ex. playoff: [event1, event2, event3], swift: [event5, event6]
- Also has support for geospatial queries 
- How do we get data into elastic search and keep consistent with primary DB
- Not best practice to use elastic search as primary data store b/c of durability concerns and cannot do complex transactions 
- If changes to primary db, changes need to get propagated to elastic search
- you can use change data capture (cdc). Changes to primary data store are put in stream and then change events can be consumed. You can abstract it away as cdc. 
- Writes to elastic search has limit. For systems with a lot of updates, do batching, queue, etc. For our case, events, venues, performers don't change a lot. Hardly ever added. Only up to 1000's a day. 
- Now search is pretty quick. 
- What about popular queries? caching
- No ranking or recommendations currently. 
- Aws opensearch (fully aws managed)
- Node query caching, caches top queries per shard
- You could add redis, memcache
- If system already has cdn for static images, cdn can cache api calls and response. But if a lot of users searching for same exact search term, results can be super quick. Less useful for more permutations for API request. 

Scalability to handle surges for popular events

- Try to make map real time so that seats are updated as they are made unavailable
- Simple solution: do long-polling where request is left open so server can keep giving responses. Cheap, does not require more infrastructure. 
- Another option could be using web sockets. 
- Another option could be server sent events, where there is persistent connection so server can send responses whenever it wants. Key difference between this and web-socket is that is it is not bi-directional. 
- Would have issues for super hot events, where screen would go black as all seats booked. Need to introduce a choke point. Could add a virtual waiting queue.
- Could use redis queue, sorted_set (priority queue based on time entered). Some event-driven logic to update queue. Simple but sophisticated solution. 


How would you scale:
- Use AWS gateway with load balances
- Scale services horizontally 
- For DB's you can shard
- Do back of envelope estimates at the end. Math is good but only do it if result effects solution. 
- Given read >> write. We could reduce read load on postrgess SQL, by caching events, venues, and performers. 

At the end make sure you satisfy all functional and non-functional requirements.

[Design Uber](https://www.youtube.com/watch?v=lsKU38RKQSo&ab_channel=HelloInterview-SWEInterviewPreparation)

Overview: Uber is a ride-sharing platform that connect passengers with drivers who offer transportation services in personal vehicles. It allows users to book rides on-demand from their smartphones, matching them with a nearby driver who will take them from their location to their desired destination

Functional requirements:
- Users should be able to input a start location and a destination and get an estimate fare
- Users should be able to request a ride based on estimate
- Drivers should be able to accept/deny a request and navigate to pickup/dropoff
- Focus in at most 3 features
Out-of-scope:
- multiple car types
- ratings for drivers and riders
- schedule a ride in advance
- message drivers
Non-Functional requirements:
- qualities of system
- low latency matching less than 1 min to match or failure
- consistency of matching. Ride to driver is 1:1
- highly available outside matching 
- handle high [[Throughput]], surges for peak hours or special events. 100s of thousands of requests/region
Out-of-scope:
- GDPR user privacy 
- resilience and handling system failures
- monitoring, logging, alerting etc
- CI/CD pipelines
I know a lot of candidates do back of envelope estimates now. But I request to do them during high-level design if there are some calculations I need for which the results will have direct impact on my design. 

Core Entities:
- Far too early to know db schema/modeling 
- Ride
- Driver
- Rider
- Location 

API:
POST /ride/fare-estimate -> Partial<Ride> 
{
   source,
   destination
}

PATCH /ride/request -> 200
{
  rideId
}

POST /location/update
{
  lat,
  long
}

PATCH /ride/driver/accept 
{
  rideId,
  true/false
}

PATCH /ride/driver/update -> lat/long | null
{
  rideId,
  status: 'pickedup' | 'droppedoff'
}

**User information from JWT or session which is why we don't see it in our API's. 

High-Level Design:

Rider (ios/android) Client -> AWS Managed API Gateway (load balancing, routing, authentication, ssl, termination, rate limiting) -> getFareEstimate() -> 

Ride Service (handle fare estimates):
- calls 3rd party mapping API call that given traffic gives eta to get from source to destination -> persist estimate in primary db. 
- make call to ride Matching service to get driver status

Ride object would have:
- id, 
- riderId, 
- DriverId ? 
- fare, 
- ETA, 
- source, 
- destination, 
- status: fareEstimated | matched

Rider
- id
- metadata
- ...

Driver:
- id 
- metadata
- status: in_ride | offline | available


Ride Matching Service:
- async process
- rider will make requestRide() call
- Matches drivers and riders
- Location DB
	- store location of drivers
	- getDriverLocations() within some x radius

Driver client:
- make calls to location service that is responsible for updating driver's' locations in location DB

Notification Service (APN, some push service)
- Send push notifs to driver device if they want to accept rider


Deep Dives:
- low latency matching less than 1 min to match or failure
	- BOE:
		- 6 M drivers
		- 3 M active
		- 3 M / 5 = 600k TPS
	- Postgres
		- range query to find drivers near you
		- not fast bc b-trees are optimal for 1-d queries not lat/long (2-d)
		- wider queries would be slow
		- can handle 2-4k transactions per second
		- not a good option
		- Geospatial index with quadtrees
	- Use queue to send requests in batches
		- adds a lot more latency
	- In order to handle high tps
		- use redis which can handle 100k-1M TPS
		- redis supports geohashing 
		- geohasing is cheap, easy to store, does not require additional data structures
		- quad tree is good if you have uneven distribution eg. yelp with big cities, not many frequent updates
		- geohashing is less good with uneven distribution of densities, but really good with high frequency of writes
		- 




