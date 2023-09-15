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

