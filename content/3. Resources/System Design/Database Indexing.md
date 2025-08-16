
Need efficient data access:
- yes -> table size > 10k rows
	- yes -> what type of data are you querying
		- full text search -> inverted index
		- location data -> geospatial index (geohashing, quad-trees,r)
		- In-memory exact matches -> has index
		- Everything else -> B-tree
	- No -> Full table scan
- No -> Full Table scan

How to find a row in a db without an index?
- Pull a page into memory and search for the row 
- pull another page into memory and search for the row
- repeat until the row is found

How do indexes speed up queries?
- First load index into memory
- Use it to find out exactly which page has our row
- Then load just that page 

Hash index not used much in production b/c they do not support range queries like b-trees. Useful more for caching. 

B-trees good for prefix searches but bad for wildcards