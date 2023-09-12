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
