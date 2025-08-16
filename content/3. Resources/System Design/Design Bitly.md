
What is a URL shortener -> converts long URLS into shorter manageable links

Functional Requirements:
- create a short url from a long url
- optionally support custom alias 
- optionally support expiration time 
- be redirected to original url from the short url

Non-Functional Requirements:
- Low latency on redirects (~200 ms)
- Scale to support 100M DAU and 1B urls
- Ensure uniqueness of short code 
- High availability, eventual consistency for url shortening 

BOTE
- scale
- latency 
- storage

Core entities:
- Original url
- Short url
- User

API or Interface
- Usually 1:1 mapping for requirement to API endpoint
- //shorten a url 
	- POST /urls -> shortURL
		{originalUrl,
		alias?,
		expirationTime?
		}
- //redirection
	- GET /{shortUrl} -> Redirect to OriginalUrl

High Level Design

- Start with API's
- Client -> primary server -> Database
	- will make request getShortUrl(longURL) -> primary server will create shorturl -> server will save url to DB
- Database
	- URL table
		- shortUrl/customAlias (pk)
		- longUrl
		- expirationTime?
		- creationTime
		- userId
	- User
		- userId
- Redirect(short) -> primary server will lookup long from short (hsort will be primary key) -> return 302 redirect (temporary) / 301 redirect (permanent so it can be cached)

Deep Dives
- Ensure uniqueness of short code
	- fast
	- unique 
	- short (5-7)
	- prefix of the long url? no b/c a ton of urls share same prefix
	- random number generator 10^9 characters (too many chars)
	- Base62 encoding, 0-9, A-Z, a-z
	- 62^6 = 56B Birthday Paradox 50% that two people in size 23 room share same bday
	- 880k collisions for 1 billion url
	- we just need to check for collisions first
	- Hash the long url
		- md5(longUrl) -> hash -> base62(hash)[:6]
		- same as above, still have chance of collisions
	- Counter 
		- incrementing a counter -> base62
		- predictability which is bad for security
		- warning, don't shorten private urls
		- rate limiting 
		- bijective function
			- squids.org
- Address low-latency request
	- pk ensures uniqueness and adds indexing using B-tree
	- Add a redis cache 
		- read through
		- LRU cache 
		- key: shortCode, value: longUrl
	- Could also use CDN as edge servers but then we don't hit primary server, which could be worse for analytics/logging

Scale to support 100M DAU and 1B urls
- 100M users -> 10^8 / 10^5 = 10^3 = 1000 rps
- Scale primary server horizontally
- More reads than writes
- Read service
- Write service
- API gateway can act as entrypoint
- You can now scale read service
- Splitting into two might be overkill
- If we use counter we need all write serves to be in sync, so we can use a global counter
- You could also replicate db's