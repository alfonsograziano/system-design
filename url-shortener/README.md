# URL Shortener System Design

![Alt text](images/System%20design%20v0.1.PNG "System design v0.1")

## Introduction
In this system design my goal is to create an url shortener service. 

We have no contraints on user requirements.
We want that the system is capable to scale up to millions of users all over the world.

## Basic features
First of all we have to understand what the user can do with our system.

There are two main features: 
- Create a new short url (from a long one)
- Read a short url

From an authentication point of view, we choose to allow just authenticated users to create short urls. **Short urls could be read from anyone**.

In our system design we are not interested in the auth sub-system so we can view it as a blackbox with those features: 
- Login and signup with username and password
- Login and signup with social (e.g. Google and Facebook)
- Password recovery


## The shortener service

Our service should support those API: 
- Create short URL from a long URL
- Update the short URL
- Delete the short URL 
- See the list of all my URLs
- Enter in each short URL to check the Analytics


To create the short URL from a long one we need first of all to understand the **allowed characters**.

To improve readability, we allow just a-z and 0-9 characters, so for example mydomain.com/thisstring01 and mydomain.com/ThisSTRING01 will be recognized as the same short url.

This give us an alphabeth with 36 chars. 

<br />
We want to users love our product, so we well allow **vanity url**.

When an user try to shorten an url, he will find a random-generated one. The user can try to **override** the random-generated url and if what he want is available, than he can make the override effective. 

<br /> 

### How to store data

We will store all the url data in a NoSQL database (NoSQL is choosen for scalability purposes).

Our database "schema" could be just this: 
- id
- createdAt
- userId
- shortUrl
- longUrl

Data will be written in the main database and will be read from read replicas. 

To make our service global, we want to have database replicas in different regions.

Unfortunately, read data from database at each request from our users if not a **scalable** strategy. We will use a Cache (like Redis or Memcached) to save data. As eviction strategy, LRU should be fine.

## The redirect service
This is a really "simple" service, but it is the core of our app. We want to redirect the user (using a 302 http response code) to the long url. 

How to do this? From our server, we could read the short-ulr code. Then we would lookup in the cache. If we have a cache hit, so we can just return to the user a redirect at the long url.

If there is a cache miss, so we have to read the element from the database, put it into the cache, and then retreive to the user.

### Why 302 and not 301?
With a 301 redirect we could cache into the user browser the long url and this is great from a load point of view. The tradeoffs are two: 
- What if the user change the redirect using the update URL api? 
- What if the user want to monitor the clicks through analytics? 

This could be a problem, so in this implementation we choose to use 302 redirect.

### Infrastructure 
Since our goal is to create a reliable, available and scalable system, we want to have this service in multiple availability zones across the globe. 

We will use a geographic route to handle the traffic and a load balancer with an auto-scaling group in each region.

If for example an entire region goes down, we could use another region as temporary **failover**.

Another problem could be the cache cold start. What if the entire cache crashes? Then for few seconds (or minutes) we will have a peak on the database that could lead to a downtime. We could use some techniques to mitigate this situation (like load at the start the most frequently used urls).

## Improvements
### Add A/B testing
### Add analytics capabilities
### Allow custom redirects for our customers


