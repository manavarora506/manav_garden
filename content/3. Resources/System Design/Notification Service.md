Functional requirements:
- createTopic(topicName)
- publish(topicName, message)
- subscribe(topicName, endpoint)
Non-functional requirements:
- Scalable (supports an arbitrarily large number of topics, publishers and subscribers)
- Highly available (survives hardware/network failures, no single point of failure)
- Highly performant (keep end-to-end latency as low as possible)
- Durable (messages must not be lost, every subscriber must receive evert message at least once)

High-Level architecture:
- All requests hit load balancer
- Metadata db, metadata service
- Store messages in temp storage
- Sender 

Frontend Service
- lightweight web service
- stateless service deployed across several data centers
- actions:
	- request validation
	- authentication/authorization
	- TLS (SSL) termination
	- Server-side encryption
	- caching
	- rate limiting
	- request dispatching
	- request deduplication
	- usage data collection
- request -> reverse-proxy (encryption, compression) -> Frontend service -> may use cache to reduce load