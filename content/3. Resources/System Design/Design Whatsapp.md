
Whatsapp is a messaging service that allows users to send and receive encrypted messages and calls from their phones and computers. 

Requirements -> Core Entities -> API or Interface -> Data Flow -> High-Level Design -> Deep Dives

Functional Requirements:
- Start group chats 
- Send/Receive messages
- Send/Receive media
- Access messages after I've been offline
- Try to keep requirements short
Non-Functional Requirements:
- Delivered with low-latency  < 500 ms
- Guarantee delivery of messages
- Billions of users, high throughput
- Messages not stored unnecessarily 
- Fault-tolerant

Core Entities:
User
Chat
Messages
Client/Device 

Latency sensitive:
yes - keep going
no - simple polling

Frequent Bi-Directional Communication
yes - keep going
no - SSE

Peer to Peer? Audio/Video
yes - WebRTC
no - websocket

Commands Sent:
- createChar
- sendMessage
- createAttatchment
- modifyParticipants 
Commands Received: 
- newMessage
- chatUpdate

Make sure you let interviewer that is final design.

High-Level Design

- Build simple system first
- Client will make connection to chat server (one node for now)
- Chat Server will connect to DynamoDB (database)
	- Chat Table
		- id
		- name
		- metadata
	- Chat Participant
		- chatId
		- participantId
	- Attatchments
		- data
	- Messages
		- id
		- content
		- creatorId
		- timeStamp
	- Inbox
		- recipientId
		- messageId
	- Find all participants in a chat
	- Find all chats in which I am a participant
	- Hashtable to map clients to associated web sockets eg. clientA:WS1
- s3 Blob storage for media content that is connected to chat server, but chat server still having a lot of bandwidth
- Pre-signed URL's allowing client to directly upload from s3
- Send ack messages when messages received and delete entry from inbox
- When client connects show them messages from Inbox

Deep Dives
- How to handle billions of users
	- Load balancer for web servers is fine because it is stateless
	- But with websockets, we need to retain connection between client and server so we need layer 4 load balancer by creating a TCP connection. Load balancer will choose chat server with the least connections.
	- We need chat servers do be able to talk to each other
		- Use kafka
			- Have topics for each user
			- When clients connect to chat server, subscribe to that topic
			- But is not good solution b/c it is not made for this scale
		- Use consistent hash ring
			- Chat Registry connected to client
			- Zookeeper
				- So that we are aware where user is
				- Client will connect to chat registry to find which chat server they are connected to. Use information in zookeeper to find where user B is. 
		- Redis Pub/Sub