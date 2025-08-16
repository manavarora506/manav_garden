**Google Drive/Dropbox System Design**

  

Cloud file system

  
Requirements:

1. Users can create and upload files
2. Files can be shared across multiple different users
3. Changes to files should be propagated to all devices with access
4. We should be able to see older versions of files

  

Less bottlenecked by speed but by how intense it is on local device. 

  

Capacity estimates:

1. 1 billion users
2. ~ 100 docs/user
3. Each doc ~100kb, some in mb
4. 1 billion x 100 x 100kb = 100 Tb of docs
5. Most docs shared with <10 people, some shared with many thousands 

  

Document permissions:  

- schema: userId, docId
- Indexing and partitioning by userId, which allows us to quickly see which documents a given user has access to

Document permissions DB:

- Replication?

- A single leader may be best for adds and subsequent removed
- User has permission, user then no longer has permission. Tombstones
- A CRDT could fix this.  
- Use MySQL

  
File Uploads:

- Split files into smaller chunks 
- Pros:

- Parallel uploading
- Only load modified chunks on file changes

- Cons:

- Added complexity on client to do file diffing, have to keep track of chunks in DB

  

Chunking Data schema:

- Chunks table: Id, version, hash, link
- Indexing by Id then version allows quick queries to determine all chunks for a given file version

  

Chunks DB considerations:

- Within a single leader, we need to be able to use locking to keep the proper version number. Cannot allow two users to modify the same version of a document at the same time. 
- In multi-leader approach, using last write wins, one file version gets clobbered, no history. However, if we needed the extra write throughput we could make it work.

  

Sibilngs:

- Idea: Use version vectors to determine concurrent writes and store both copies

  

Seems useful to use an ACID db so that all rows are updated atomically. 

- Serializability ensures that one version is finished before another can publish
- If trying to write a version that already exists DB rejects write

  

Uploading Actual File

- We want to upload as few chunks as possible 
- Upload file data to s3 and once complete, write meta data to chunks DB
- Have a cleanup job which polls s3 and checks if chunk lives somewhere in the DB

  

Pushing File changes to Users

- We want to ideally keep # of connected maintained by each client low. The best way to do this is to have a single node responsible for a given client. 

  

Hybrid Approach:

- For most documents < 10 users have access

- We can push these somewhere for the user, shard by userId

- For super popular documents, 100k have access

- Push changes to a popular document service, shard by documentId
- 3 connections is worse than 1, but better then having to send the popular files everywhere 

  

File Routing **Review

- Upon making changes to a file, we need to propagate changes 
- File is uploaded to s3 and chunks db. Kafka queue then shards file. We use permissions db. Then file is sent to unpopular files or popular files. . 

  

Metadata Reading

- Since we will have many consumers of each metadata change, something like kafkaesque makes sense here.
- Persistence ensures that clients that go down can re-read missed changes