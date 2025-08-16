=
What's unbounded data?
- Ever growing, infinite data set
- Ex:
	- website clicks
	- temp sensors
	- gps locations
	- credit card purchases

What is streaming?
- Data processing engine for infinite or unbounded data
- ongoing data processing
- unbounded data processing 
- low latency, approx
- examples
	- trending instagram posts
	- popular music on spotify
	- trending search queries
	- top k movies

Lambda Architecture
- Run a streaming system alongside a batch system
- Both performs essentially the same calculation 
- Streaming system gives:
	- low latency, inaccurate results
	- approximate result
- Batch system runs in the future
	- corrects output
- Ideally, there shouldn't be any need for two different systems
- Streaming engines have evolved quite a bit

Different Times in Streaming

How to deal with Late Data

Different concepts of Time
- Event time
	- time at which events actually occurred
	- events come into system with a timestamp
	- {order)id:10, order_timestamp: 123232}
- Processing Time
	- Time at which events are observed in system
	- timestamp is assigned when event is processed
	- can be very different when the4 event actually happened
		- delayed data
		- out of order data
		- backfilling data

Why aren't they equal?
- In an ideal world they would be equal. Events would be processed the moment they occur
	- but external factors cause delay:
		- internet issues, especially for mobile events
		- slow processing engine
		- backfilling old data

What to use for computation?
It depends.

How much do you care about actual event time?
- Billing events
- User interactions
- Historical data

Maybe you don't care about actual time?
- Analytics data
- CPU usage

What to do with Late data?
Most common solution: ignore it

Problem is with windowing operations:
Ex:
- Sum of all events in last 60 minutes
- Average of scores in last 30 minutes
How do we know we have seen the last event of that window

Watermarks:
- Defines when to stop waiting for earlier events
- It's a threshold to specify how long the system waits for late events
- If an arriving event lies within the watermark it gets used. If it's older than watermark, it is dropped
- Example:
	- withWatermark(eventTime, delayThreshold)