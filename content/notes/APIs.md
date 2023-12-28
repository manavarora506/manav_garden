---
title: APIs
date: 2023-12-25
tags:
  - seed
enableToc: true
---
# Rest API
**Sources:
https://dev.to/cassiocappellari/fundamentals-of-rest-api-2nag
https://www.redhat.com/en/topics/api/what-is-a-rest-api
https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm

![[rest_api.png]]

In REST, the implementation of the client and server can be done independently without knowing each other. The separation allows each component to grow independently. All REST systems are stateless meaning that the server does not need to know anything about the state of the client and vice versa. 

**Sending Requests**

REST requires that a client make a request to the server in order to retrieve or modify data on the server. A request generally consists of:

- an HTTP verb, which defines what kind of operation to perform
- a _header_, which allows the client to pass along information about the request
- a path to a resource
- an optional message body containing data

There are 4 basic HTTP verbs we use in requests to interact with resources in a REST system:

- GET — retrieve a specific resource (by id) or a collection of resources
- POST — create a new resource
- PUT — update a specific resource (by id)
- DELETE — remove a specific resource by id

In the header of the request, the client sends the type of content that it is able to receive from the server. This is called the `Accept` field, and it ensures that the server does not send data that cannot be understood or processed by the client. 

Requests must contain a path to a resource that the operation should be performed on. In RESTful APIs, paths should be designed to help the client know what is going on.

Conventionally, the first part of the path should be the plural form of the resource. This keeps nested paths simple to read and easy to understand.

**Sending Responses**

In cases where the server is sending a data payload to the client, the server must include a `content-type` in the header of the response. This `content-type` header field alerts the client to the type of data it is sending in the response body.

|Status code|Meaning|
|---|---|
|200 (OK)|This is the standard response for successful HTTP requests.|
|201 (CREATED)|This is the standard response for an HTTP request that resulted in an item being successfully created.|
|204 (NO CONTENT)|This is the standard response for successful HTTP requests, where nothing is being returned in the response body.|
|400 (BAD REQUEST)|The request cannot be processed because of bad request syntax, excessive size, or another client error.|
|403 (FORBIDDEN)|The client does not have permission to access this resource.|
|404 (NOT FOUND)|The resource could not be found at this time. It is possible it was deleted, or does not exist yet.|
|500 (INTERNAL SERVER ERROR)|The generic answer for an unexpected failure if there is no more specific information available.|

**Request Anatomy**:
* URL: URL means **U**niform **R**esource **L**ocator, which **is the address to not just identify a resource, but also to specify how to access it.** For example: `http://api.example.com`
* URL: URI means **U**niform **R**esource **I**dentifier, which is used in the URL to **specify which resource the Client would like to access in a request**. For example: `http://api.example.com/products`
		URL: `http://api.example.com/`  
		URI: `/products`
	Therefore, every URL is an URI, but not all URIs are URLs.
* Parameters: CRUD
* Body Params: **body of the request which contains all the data that the Server needs to successfully process the request.**
* Route Params: **parameters inserted in the URL with the information to identify a specific resource**
* Query Params: **parameters inserted in the URL, but with the main difference that it’s use cases are related to filter and search information about a resource, or even paginate and ordinate the results**
* Headers: **allows sending extra information in a request**

HTTP Methods:

GET - to retrieve data from a server
POST - to create a new resource in the server
PUT - to update a resource data in the server
PATCH - to update specific data in the server
DELETE - to delete a resource in the server

HATEOAS is an acronym for **H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate, it’s the concept that when sending information over a RESTful API the document received should contain everything the client needs in order to parse and use the data i.e they don’t have to contact any other endpoint not explicitly mentioned within the Document