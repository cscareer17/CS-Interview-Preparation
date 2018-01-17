I was quite afraid of the system design interview until I realized that all the interviews were the same, as long as I followed a plan defined below. Look at the plan for the first examples. The last examples contain less information because it would be rewriting the same thing over and over.  

# Design a URL shortener 
1. Features:
   1. Functional features
      * Create new translation
      * Get forwarded to the correct URL
      * Delete an URL
      * Create its own url
      * Provide authentification


   2. Nonfunctional features
      * Make sure the system is always available, redirection should also be very quick
      * Do some analytics

3. Figures

   * Reads: 100M per day, 1000 query per sec
   * Writes: 1M per day, write an object of ~500 bytes -> 500GB per day of write

4. System API

   * `createURL(longURL, alias, username, expire_date)`
   * `deleteURL(shortURL, username)`
   * `forwardURL(shortURL)`

5. Data Models

   * store two tables for `User`, and `URL` 
   * `User`: username, email, password, number_api_call
   * `URL`: shortURL, long URL, userID, timestamp
   * Requirements of the database: store simple key, value pair, lot of reads, few write, availability, don't need to do join over the rows, so SQL might be useless
     * Hence use Cassandra

6. Schema

   ```
   clients (Browser) <-> HTTP server (nginx) <-> database (Cassandra)
   ```

7. Algorithm:

   1. Encoding URL
      1. Encoding with a HASH function, like SHA256, that has a certain base (base36 [a-z0-9]), change the base so as to have a new base of at max X characters.
      2. Should we keep the same short URL for same long URL, or provide a user-specific URL, discussion...
   2. If we need, to encode, it might become quite expensive to compute the encoding, so we might look for a generator of keys, but given the number of encoding per second required, ~10, it seems duable
   3. Redirecting URL using HTTP protocol (300 code)

8. Optimization

   1. Data Partitioning and Replication
      * For database write, we can't partition randomly because read will become very painful to perform. We can partition with a consistent hash.  We partition by URL across 3 nodes (1 Leader, 2 Follower) (30 M per month/3=10Million URL). Each partition gets replicated X times where X gives that the probability of at least one node serve `1 - (proba_failure)^X` is sufficiently large
   2. Caching
      * We might want to cache 20% of the daily request. 256 GB of data, Policy=LRU, cache get updated if the server contact the database
   3. Query queueing
      * Load balancing queries using a third party like OVH 

9. Telemetry

   If needed we can maintain a data warehouse where every read and writes (events) get appended to a database table with append-only mechanism

10. Authentification

  * Decide what user can see? some user might want their data to be private

# Create Pastebin
1. Features

   1. Functional features
      * User can past code
      * User can share code
      * User can delete code, and handle privacy code
   2. Nonfunctional features
      * Reliable system, no data should be lost
      * System should always be available
      * System should be fast

2. Figures

   * Writes: 5 write per sec, each object is 10KB in average -> 10GB/day
   * Reads: 10 read per sec -> 10Kb loaded -> bandewith = 100kb/sec

3. API

   * `saveDocument(user_api_token, document, metadata, shared_policy, alias_url)`
   * `getDocument(url, user_api_token)`and `deleteDocument`

4. Data Model

   * We can think of using an MYSQL database to store metadata content, and keep the documents separately in a block
   * Model needs to store/read quickly large string file. Could be could to store them consecutively and retrieve them quickly. 
   * No relational, no joint
     * Document database that would save two collections: documents, and users. Documents are indexed by a hash of their URL (better for partitioning)
   * MongoDB? or Cassandra? Discussion...

5. Schema

   ```
   clients <-> HTTP Server <-> document database
                         <-> metadata object
   ```

6. Algorithm

   * Use a hash function to retrieve the URL 

7. Optimization

   1. Replication
      * You want the data never to be forgotten. Make sure the data is not lost before being written to the database, for example by queueing message with logs. Use an architecture Leaders/Followers like in MongoDB
   2. Partition
      * Partition the data by the hash (consistent hash)
   3. Caching 
      * Caching query
   4. Availability
      * Load balancer between the client and the server, and between the server and the database. 

8. Analytics:
    * Using Elastic Search service to do analytics on Text? Discussion...

# Design Instagram
1. Features

   * User can upload photos
   * User can see photos of his friends
   * User can subscribe to friends
   * User can comment photos
   * User can control who can see his photos
   * User can use an app to take a picture and modify the picture

2. Figures

   * 20 new images per seconds: 180.000 writes of images on average per day. Each image is ~200KB. 360.GB day of new images 
     * Latency: 200MB incoming data/s + time to process
   * 1000 users required a pictures per sec -> 1.000 picture per sec -> 200MB of served data per sec -> 2TB per day
   * Number of users: 1000 new users per day -> 365.000 per year -> 3.6 M of users

3. API

   * `publish_photo(byte arr[] img, user_api_token, timestamp, message)`
   * `get_photos(user_api_token, timestamp)`
   * `comment_photos(user_api_token, comment, timestamp, img_id)`

4. Data Model

   * Few writes, but a lot of reads. Write and Reads are large. 
   * Images can't disappear and need to be saved. It is ok if users don't see picture instantly!
   * MYSQL to handle the info of a user, who follows who, and  Image's metadata. 
   * Use another storage like HDFS for images. If we save data in Hadoop, we can perform some operations asynchronously on the data. We store the path of each file in a metadata database. 

5. Schema

   Writes are synchronous. We only contact back the user when the data has been written or there has been an error

   ```
   Client -> HTTP Server -> Metadata DB
                   ------> Image DB
                   ------> Other Server
   ```

   Reads are asynchronous, the server doesn't wait for the user to ack it receives the data

   ```
   Client <- HTTP Server <--- Metadata DB
                       <--- Image DB
   ```

   You can make two servers for reading and write, that receives messages from a queueing system

6. Algorithm

   * Algorithm to distribute the photos to followers. We can have a separate server that computes the timeline for users. 
   * The server that computes timeline receives request of computing Timeline, subscribe to new incoming photos. Compute Timeline for the most frequent user. Or we can have a scoring system for each user of how likely they will come back in the next X minutes, and compute the timeline for this user. We can store the future timeline in Redis store where key is the user ID, and values are a list of PhotosID

7. Optimization

   1. Partitioning
      * It is possible to partition by user ID. There is a potential drawback with this methods: hot user, not balanced. Partition geographically
   2. Replication
      * Replication is not an option. Write must be replicated onto multiple nodes. Multiple replicas in production, and some backup plans. One node is responsible for receiving write if it goes down: election of a new Leader, followers are responsible to read what the Leader have saved
   3. Caching
      * Caching Timelines
   4. Load Balancer
      * Load balancer everywhere

8. Security

   * Making sure to access as authenticated. Protect images on the server

## Design Facebook Messenger

1. Features

   1. Functional Features
      * Send messages to another person
      * Messages are saved
      * Show if people are online or offline
   2. Non-Functional Features
      * Persistent data, never deleted
      * Serve messages as fast as possible
      * Synchronisation on every device
      * High availability
   3. Extended requirements
      * Push notification and group conversation

2. Figures:

   * Time: Number of people on the app every day: 100M users, each exchange 40 messages -> 200.000 new messages per second -> 4 billion messages/day
   * Storage: 1 Message has a size of ~1Kb -> 4 TB per day * (5 * 365) ~7 PB of data for five years
   * Bandwith: 200.000 Reads + 200.000 Writes = 400.000 * 1 Kb = 400Mo/sec

3. API design

   * `sendActivity(user_api_token)`

   * `sendMessage(user_api_token, recipient_api_token, message, metadata)`

   * `getMessages(user_api_token, recipient_api_token, max_num_utterances, search_info)`

     Create a REST API that would provide this service

4. Component Design on a single machine

   * Message Handling:
     * Receiving messages: Users can open Pull connection to the server waiting to get new messages. Whenever there is a new message, it gets sent to the user. To retrieve the open connection quickly, there might a key-value database where key contains the user id and value contains the connection object.
     * Distributing messages on a phone: server receives a message, save it, and until the message is delivered, the system keeps track of the message on another rapid database where key is the user ID, and value is a list of messages that need to be send
     * Distributing messages on a desktop: Click on a conversation ask the backend to retrieve the last messages.
   * Managing user status
     * Using the connection saved in the database. Status list is computed when a user connects, and also when a user sends a message to a specific user
   * Push notification
     * Each mobile client implement server where they push notifications to the device when asked too

5. Data Model

   * Fast retrieval of the latest messages
   * Client should paginate while retrieving data
   * Big table? Discussion...

6. Sharding

   * Partition by userID, with a consistent hash function to decide which database to put data on. History of messages can be retrieved by only looking at one cluster, which makes it very efficient to read and write

7. Replication

   * We need a system that has replicas. If we follow the architecture of Leader/Follower, where we only have Leader in production and Follower copying from the Leader. That would make it faster, than storing message on a multi Leader platform where data would end up in multiple different Leader, all of them might be very busy

8. Load Balancer

   * Same as other examples


## Designing Facebook New's feed

1. Features
   1. Functional Features
    * Send content to our network
    * Interact with the content of our network
    * Receive a summary of our network content
    * The content can be images, messages, activity, videos

2. Figures:

   * Trafic estimate: 
     * People connecting every second: 10000 persons -> 100.000.000 million active users per day 
     * Number of posts read per user per day: 500
   * Storage estimate: 
     * One post 1KB, 500 posts needs to be saved. It is 50 TB of data per day. 15PB of data in total for one year
     * To store such a thing per day. If a server store 2TB of data in disk, and 100 GB of RAM, we would need 500 servers to store all timelines

3. API
   * `getUserFeed(api_dev_key, user_id, number_of_posts, timestamp)`

4. High-level design
    * Web server to maintain connection with a user and transfer data
    * Metadata database and cache
    * Application server to execute the workflow of storing new posts in the database. Also, retrieve and push the newsfeed to the user
    * Post database
    * Video and Photo storage
    * Newsfeed generation service
  * Notification service

5. Data Model
    * `User`, `Entity` and `FeedItem`, `UserFollow`, `FeedMedia`, `Media` are the main object. Two relation `User-Entity`, and `FeedItem-Media` relation

6. Algorithm
    * Feed generation for all user-generated offline, otherwise it would be slow for the user. Maintain a history when the last feed was generated, and create a new one following a certain policy, like LRU.
        1. Retrieve id of all users connected with a person
        2. Retrieve popular content linking to these persons
        3. Rank the posts
        4. Store the feed in a cache 
        5. Send and paginate whenever asked
    * Feed publishing can be implemented using a mix of Fan-out-on-load model where data is stored in a database and pull from the server by the user when needed, and a Fan-out-on-write model where the data is sent to all the follower whenever a new post has arrived. Given the profile of a user, different policies can be implemented.

7. Sharding
    * Sharding based on userID
    * Sharding based on creation time. Storing all the top item on the same servers. However, this will not distribute the traffic. 

##  Designing Youtube
1. Features
   1. Functional feature
     * Upload videos
     * Watch videos
     * Get recommended videos
     * Interact with videos (like, comment)
   2. Non functionnal features
     * Availability
     * Reliable (no loss of data)
     * Real-time experience

2. Figures

   * Traffic estimate:
     * Write: 10 video uploaded every sec -> 36.000 per hour -> 470.000 videos uploaded per day -> 130.000.000 videos per year.  
     * Read: each sec, 10.000 people start video -> 10.000 * 90.000 = 900.000.000 millions views per day
   * Storage:
     * Each video: 10Mb -> we need a storage space of 1000Pb of data per day
   * Bandwith: 1000 * 10Mo = 10Gb/sec for read, 100Mo/sec for write

3. API

   * `uploadVideo(byte[] array, title, tags, user_api_credentials, metadata, encoding)`
   * `searchVideo(search_query, timestamp, duration, num_video_to_return=10, user_api_credentials)`
   * `get_home_page(user_api_credentials, num_suggestions)`
   * commenting api. Discussion...

4. System design

   ```
   client < ----- > web server <---------------> metadata database ^
                               <---------------> user database     |
                               <---------------> processing queue <-----> blob database
   ```

5. Data Model

   * `User`: username, password, api_key, age, registrationDate
   * `Video`: identifiant, title, tags, num_like, num_dislike, num_comments, metadata_key, storage_id
   * `Comment`: video_id, user_id, comment
   * `Views`: video_id, user_id, time_view (Maintain two indexes on for views and one for user_id)

   Use MYSQL

   Saving videos in HDFS, with replication. Architecture Leader/Follower. It is ok to have some delay before videos are seen on cluster

   We have to think about access pattern and grouping, so we can think about designing in BigTable. 

6. Partitioning

   Partitioning video metadata using a consistent hashing technique on the video id

7. Video duplication: split video into blocks, if blocks are matching, do not duplicate the copy

8. Load Balancing: consistent hashing among cache server

9. Caching

   * 20/80% rules. We can try to cache 20% of the video: 180 millions of videos per day is around 200Gb of metadata to store. Use a LRU policy

## Design Twitter Search
1. Features
   1. Functional features
     * Searching for the status of users
   2. Nonfunctional features
     * Quick, and available
2. Figures:
   * Traffic estimation: 10.000 queries/sec -> 900.000.000  millions per day. 
   * Storage estimation: 
     * Query size: each tweet as an average of 3 words -> 20 * 16 = 320 bytes * 1 B = 320GB of query / per day
     * Query response size: 10 status: 10ko * 1B = 10TB of data send, 100Mo/sec
3. API
   * `search(words, timestamp, user_api_key, num_results)`
4. Data model
   * Store Status in MYSQL table `StatusID, StatusText`. Have an index for each possible word in the English vocab and map it to list of statusID. 
5. Sharding
   * Sharding based on a word. What if one word is popular.
   * Sharding based on status ID: multiple index server aggregates a list of status ID and returns the potential ID. Aggregate Server rank the results
6. Replication
   * Master/Slave logic. If both servers go down, we need a way to rebuild this server, we can have a key-value storage that keeps all the statusID saved on a certain database
7. Ranking
   * Ranking on each index server, then merged sorted result


## Design UBER backend
1. Features
   1. Functional features
     * People can order and pay taxi 
     * People can see where their taxi are as soon as they are matched
     * System should provide a path from point A to point B
   2. Nonfunctional features
     * Always available
     * Secure 
     * Real-time
2. Capacity
   * Traffic estimation: 1M daily users. 10 new user per second. An update is done every 3 seconds. 
   * One query: 30bytes for one update on the position -> 30Mb of send data about the position. 
3. API
   * `search_driver(current_position, destination, user_api_key, type_of_car, starting_at)`
   * `get_position_driver(user_api_token)`
   * `update_position(current_position, driver_api_token)`
4. Data Model
   * MYSQL to store the information about a User, a Driver, and a Trip
5. Algorithm
   * QuadTree works when places do not move frequently. 1M driver updating their position every three seconds. Around 10.000 drivers per city at max. It means 3.000 updates on the tree per seconds. We can use a QuadTree and store in a Redis the DriverID:  old position, grid, and check if the new position is still on the grid, otherwise we update the QuadTree. 
   * This update of position is around 30 bytes as said so that 30Mb of data, which can be stored in a Redis.
   * Everyone can stay on one server if it was only about memory usage, but we will need to think about availability and reliability.
   * When a user connects, it received a list of drivers to subscribe to. Their position will be updated on the list. A subscription service can be set on the server using long pooling or push model


## Design a Web Crawler
1. Features
    1. Functional features
        * Test web pages and valid syntax
        * Monitor sites to see when content change
        * Maintain mirror sites for popular Web sites
        * Build a special purpose index, like Google
    2. Nonfunctional features
        * Scalability
        * Extensibility: modular way, new functionality can be added

2. Capacity
  * Only HTML page, HTTP protocol.
  * 15 billion pages crawled in 4 weeks ~= 6000 pages/sec
  * 15B * 100Kb = 1.5 petabyte

3. High-level design
  * Pick a URL
  * Determine the URL
  * Connect and retrieve the content of the page
  * Parse the document, and look for new URL, or do a path-ascending on the current URL
  * Add the new URL to be processed
  * Save the document

4. Detailed component design
**URL frontier**: The URL frontier is the service that will keep track of the URL that needs to be scrapped. It can keep them in a FIFO. We can distribute the computation across multiple clusters and have an aggregate cluster.

**The fetcher module**: given a URL, retrieve the document

**Document deduplicate stream**: Store a checksum of every file, to not process documents if they have already been processed. Using a hash function like MD5

**URL filter**: Filtering URLs manually using a blacklist

**Domain name resolution**: This is used by the fetcher module to retrieve the IP. Domain name is a bottleneck.

**Checkpointing**: saving to disk to restore if failure  

5. Algorithm
* Crawler trap, spam site, and cloaked content. Look for the AOPIC algorithm. 


## Design an API Rate Limiter
1. Features

   * Limit the number of requests an entity can send to an API: Rate limiting means defining speed of answer, and throttling is the process of controlling the usage of the APIs
   * Always available, no extra latency

2. System design

   ```
   client <---> HTTP Server <-----> API Rate Limiter
                  <----------------> App server
   ```

3. Data Model

   Key: userID or IPID, value: [all timestamp connection] or only the counter. Move the windows when new connection. Or schedule a Mysql routine

4. Techniques of caches write: write back-> directly save to the cache, and continue the transaction. Quicker, but nothing is definitely saved on a disk.


## Designing TypeAhead Suggestion for Google
1. Features
    1. Functional feature
        * As user is typing their query, it should suggest finishing what the user is typing
    2. Nonfunctionnal feature
        * Should be quick, and always working
    3. Extended requirements
        * Filtering hating term or expression

2. Storage
    * [(node, num_child), (node, num_child), (node, num_child)....]

3. Partition
   * split by the first character, can lead to unbalance tree
   * split by range of word, if trie is too large, split in two
   * split by hash of query, faster
4. Caching
5. Optimization on the client side
   * do not send if typing, wait at least 50 sec