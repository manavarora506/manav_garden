
Following 5 step approach is immature
Has to be a discussion
Might skip some things b/c they might be too trivial 
Focus on right things

sys design that touches real world
focus more on software piece
knowing what you don't know is good signal


functional requirements:
- support some mobile app
- people can find available bikes
- rent bikes
- drop off bikes
- charging dock structure
- real time tracking of bike/battery 
- bay area base
- reserve bikes 

nonfunctional requirements:
- low latency (seconds to ms)
- app should be highly available
- night time usage might be lower
- db must be consistent to not allow two riders to rent the same bike

Bikes probably have chip that has heartbeat signals. Don't have good intuition on that, like what would be a good frequency. What would drain the battery? When bike is rented, more frequent heartbeats. You can get these signals from user's phone. 


 
Booking system is big part of the software problem. Lightweight tracking. How to represent bikes in a 2d world. 

Reservation system:
- status in a db, out of comission, rented, ready_to_be_rented, low_battery, etc
- rankings which bikes to show 

Review uber + booking system


Amazon Review Summary:
- Look at engagement like thumbs up/down
- behavioral signals (how much time are they spending, do they prefer A/B)
- Ask for direct feedback
- think about both behavioral and direct signals 
- After reading summary, do you see a drop in questions asked by customer
- Track does this end in purchase or not
- Tracking return 
- need to have summarization engine
- not very real time data
- summarize once a day
- Prompt
- batch 1,000 summaries at a time
- careful about the negative summaries 
- make sure aggregations happen with the same weight 
- 101 is very negative.



Design Question 

 Overview SF Bike Share offers a service that allows bikes to be rented from multiple stations that are placed across San Francisco. Bikes are tracked as they are rented and returned to stations. A full history of all bike rentals/returns is recorded, as well as the ability to see bikes that are currently rented. Each station is supported by many different sponsors who have their logos displayed on the stations. Each bike has a sensor on it that allows us to see its "vital signs" (brake quality, tire pressure, and other health indicators). We obtain the vital signs from each bike every 10 seconds. ## 

A mobile app or mobile web app used by customers that handles customer bike checkouts - A SaaS application (JavaScript/React) that serves as a reporting system for viewing live and historical data - A SaaS application for sponsors to see how many impressions their logo is getting 


Things to deeply understand:
- why optimistic locking
	- I’d prefer optimistic locking or conditional updates first because contention per individual bike should usually be low, and it avoids blocking. If metrics show hot-bike contention or too many failed retries at busy stations, I’d move the checkout path to a narrower pessimistic lock on the bike row, or use a reservation step.
	- It's also worth knowing about **optimistic concurrency control** (OCC), which takes a different approach entirely. Instead of locking rows upfront, you read data along with a version number (or timestamp), do your work, then at commit time check whether the version has changed. If it has, someone else modified the row first, so you retry. PostgreSQL doesn't have built-in OCC syntax, but you can implement it at the application level using a version column:
- What is redis sorted set
- How time series db work
	- Tags are crucial because they're indexed. Queries filtering by tags are fast. Fields are not indexed - they're the actual time-series data you're storing.
- Postgress has GIN extension for decent search capability. PostGIS for fast searching
- Row level locking is simplest and used For Update