[Spotify](https://www.youtube.com/watch?v=_K-eupuDVEc&t=11s&ab_channel=IGotAnOffer%3AEngineering)

1) Clarifying Questions:
	- Spotify:
		- Songs/music
		- Playlists
		- Users
		- Artists
		- Podcasts
2) Use cases:
	- Finding and Playing Music
3) High Level (Metrics)
	1) Numbers:
		- Users: 1 Billions
		- Songs: 100 million
		- MP3 audio file - 5MB
		- Total audio: 500 TB
		- 3x replication -> 1.5PB
		- 100B per song metadata -> 10 GB songs
		- 1KB per user >> 1TB 
4) Design
	1) Spotify App
	2) Load Balancer
	3) Spotify web-server (multiple)
	4) Database
		- Song Audio DB (AWS s3) files are blob data, immutable. Scales linearly. Usually just read data. 
			- Song MP3
		- Metadata (users, songs, artists, ...) DB (AWS RDS)
			- Songs table:
				- sond_id
				- song_url (sharing)
				- artist
				- genre
				- link to album cover
				- audio link
5) Run through use case for finding music
	1) Use CDN as cache (AWS cloudfront)
6) Load balancing
	1) Make sure web servers are not overloaded (a lot of requests coming in, network bandwidth)
	2) Consider multiple metrics when applying load balancers
7) Replication
	1) For events where we have data outtages
	2) Place replicas close to users. Eg. BTS data close to Korea. Geo-aware strategy of data.