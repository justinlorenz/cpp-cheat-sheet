## Livestreaming Application
### Video Data
- PUT Video API
  - Store video data in filesytem database (HDFS or Amazon S3)
  - Use RTMP for data upload (highly reliable because need data to be persisted when posted)
- GET Video API
  - getVideoData(id, device, offset) -> 10 second long frame
  - Send video data back to user using WebRTC 
  - MPEG-Dash, HLS are other data streaming options as well
- Post Comment
  - comment(id, data, authorId, videoId)
  - Client can use simple HTTP POST request
  - If comments table won't be queried much than can use NoSQL else for relational queries use MySQL
- Video Resolution
  - Take raw video gets broken into segments of 10 seconds. Provide segment to program which converts it to different resolution (4k, 1080p, 720p)
  - Use Map Reduce design pattern -> take one video and send to multiple servers which will all compress / convert the data (open server will take the request)
- CDN (Content Delivery Network) Solutions
  - Can be used to cache data

## Reservation System
- Need to deal with concurrent database accesses
  - Optimistic Locking - Optimistic Locking is a strategy where you read a record, take note of a version number and check that the version hasn’t changed before you write the record back. For a system like Hotel Booking where the Number of request/query/transactions per second might not be very high, this is a great option. When there are a lot of concurrent requests, let’s say like in the case of Flight Booking or a Popular movie this has big performance ramifications. Optimistic Lock performs badly if there’s a lot of contention at the same time because this leads to many transactions needing to be abandoned. If the system is already running at its maximum throughput, retrying a transaction could make performance worse. The system will eventually be able to process all transactions in order, but in the meantime, some may experience delays.
  - Pessimistic Locking = Pessimistic Locking is when you lock the record for your exclusive use until you have finished with it. It has much better integrity than optimistic locking but requires you to be careful with your application design to avoid Deadlocks. It is based on the principle that if anything might potentially go wrong, it’s better to wait until a situation is safe again before doing anything. 

## Database Design
### Caching
- Querying database and don't want to query database every time.
- Key would be the query pattern and value would be the response from database
- Common solution: Redis

### File Storage
- Designing system like Amazon where have various products (images / videos of the product) or like Netflix where need to store video.
- Want to use Blob storage (not a database because not something you query on).
- Common solution: S3
- Along with S3, most likely want to use CDN. Used to distributing the same image in a lot of locations. Example: want to distribute image across the globe to many different servers so people can query quicker instead of querying S3 itself.

### Text Search Engine
- Need to provide text searching capabilities on various products (Amazon product, searching video on Netflix, searching on Uber)
- Common implementation: Elastic Search, Solr (both built on Apache Lucene). These are search engines, not database. Not guaranteed to get data so could be lost (primary data source should be somewhere else)
- Fuzzy text search
    - Searching for "airport" but user types "ariport," want database to realize airport should be returned. Can provide level of fuziness that changes it back to correct word if has low edit distance.
 
### Timeseries Data
- Building Grafana, Prometheus, Graphite, metric tracking system (pushing metrics about CPU, latency)
- Extension of relational database. Only do sequential updates in an append only mode (not random updates). Read queries are bulk queries in a time range (query last few min / hours of data). Optimized for this kind of query / input pattern
- Common solutions: InfluxDB, openTSDB

### Datawarehouse / Big Data Store
- A lot of information and store various kinds of analytic requirements.
- Example: provide analytics on all the transactions (# orders, revenue, most popular item)
- Need datawarehouse for analytics on all data of company. Dump all data and provide queries on the data (generally used for offline reporting)
- Common solutions: Hadoop

### Relational vs. Non-Relational DB
- First thing that helps decide is the structure of the data. Very structured means use relational. Structured would be info you can easily model in forms of tables and tables would have rows and cols (user information on social network)
- If you have structured data, do we need atomicity or transactional guarantees (ACID)? Example: transferring money is two queries where one decreases money from person A and another adds money to person B. Database would need to wrap these in a boundary that says either both fail or both succeed. Common solutions: MySQL, Oracle SQL Server, PostgreSQL.
- Imagine building Amazon catalogue where have all items available on the platform (each item has certain attributes). If have a lot of attributes and wide variety of queries then need document DB (Mongo DB, CouchBase).
- If have ever increasing data + finite queries than use columnar DB (Cassandra, HBase)
- Combination of database structure can be powerful. For example, if you are Amazon, you can have an ACID RDBMS hold temp orders and non-fulfilled orders, and then move to a NoSQL DB for permanent storage of successful orders.  
  
