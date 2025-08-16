
Functional Requirements:
- Users should be able to search for businesses by name, location (lat/long), and category
- Users should be able to view businesses (and their reviews)
- Users should be able to leave reviews on businesses (mandatory 1-5 star rating and optional text)

Non-Functional Requirements:
- high availability >> strong consistency
- scale to 100 m dau and 10m businesses 
-  low latency search (<500 ms)

Core Entities:
- Business
- Review
- User

API Routes:
- Search -> GET /businesses?category={}&location={}&name -> Partial <Businesses[]&limit={}>
- View Business -> GET /business/:id -> Business GET /business/:id/reviews?page={}&limit={}-> Reviews[]
- Post a review (needs authentication)-> POST /business/:id/reviews {rating, text} -> Review | 200

High Level Design:
-  To search business: Client <-> API Gateway -> Business Service -> Business DB (id, name, category, description, address, lat/long, s3link, avgRating, numRatings) SELECT * FROM Business where category = ".."
- Review DB (id, userID, businessId, rating, text?)
- User DB (id)
- To view business details and reviews: SELECT * FROM Business where businessId = "id"
- How will users be able to leave reviews on businesses?
	- Client <-> API Gateway -> Review Service -> Primary DB (in this case we can just have one db b/c there is direct relationship between businesses and reviews)
	- API Gateway will do authorization

Deep Dive Questions:

How would you efficiently calculate and update the average rating for businesses to ensure it's readily available in search results and will be accurate up to a minute?
- Message Queue would be over kill
- CRON job
- Add the review, update the newRatings, calc newRating 

How would your system handle the scenario where multiple users submit reviews for the same business simultaneously? What mechanisms would you put in place to ensure data consistency?
- We can use row lockings, where we lock the business row at the start of the transaction and then release at the end
- We could also use optimistic concurrency control where version (avgRating, numRatings) you can check this at the beginning and end of transaction

How would you modify your system to ensure that a user can only leave one review per business?
- client side not good enough b/c we could bypass by making API request
- put constraint as close to db as possible
- put constraint unique on user id and business id respectively as composite key

How can you improve search to handle complex more queries more efficiently?
- Use elastic search and cdc but this might be overkill
- Postgres has extensions like PostGIS to create geo-spatial indexes on lat and long and also supports full text search 