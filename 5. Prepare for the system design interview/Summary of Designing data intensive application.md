When preparing for the system design interview, I found this [book](http://shop.oreilly.com/product/0636920032175.do) to be a really good preparation. Here are some notes on the book, but I'll recommend you plan some time to skim through the book!

* The goal is to familiarize with complex data-intensive systems and the technology/algorithm behind every big system like Google file system, or Amazon online shop.
* Estimated Time: 2 days

# 1. Introduction

**Data, software and communication can be used for bad or for good, to create opportunities for everyone, and to avert disasters.**

To build a data system, you need a lot of tools, here is a not exhaustive list:

* Database: store data so they can be used again later
* Caches: Remember the result of an expensive operation that speeds up reads
* Search indexes: Allow user to search data by keywords
* Stream Processing: Send a message to another process, to be handled asynchronously
* Batch processing: Periodically crunch a large amount of accumulated data

We gonna look at:
* **Reliability**: the system should continue even in the face of adversity
* **Scalability**: there should be reasonable ways of dealing with the system grows (data volume, traffic volume)
* **Maintainability**: maintain the system to new use cases

### Reliability
A fault is not the same as a failure. A fault is when the component of the system deviates from its spec, whereas a failure is when the system as a whole stops providing the required service to the user.
There exist multiple types of faults:
* hardware faults. use redundancy to the individual hardware components. But now as the system needs more and more machines, there is a move towards systems that can tolerate the loss of entire machines, by using software fault tolerance techniques in a preference for hardware redundancy. 
* software errors. there is no quick solution, it requires careful thinking, process isolation, thorough testing, allowing processes to crash and restart, measuring, monitoring
* human errors: design interface that discourages doing the wrong thing. sufficiently open to avoid work around them. unit test, integration tests.

### Scalability
As already said, it describes a system's ability to cope with increased load. 

#### Measuring system abilities
* Latency is the duration that a request is waiting to be handled, and a response time is a time to process the request. They are the same.
* Median is a better metric to know how long users typically have to wait. 
* Percentile is another one. Usually, companies optimize the 99.7 percentile of responses time. Percentiles are also used in Service Level Objectives (SLO) and Service Level Agreements (SLA) contracts. They set expectations for clients of the service and allows customers to demand a refund if the SLA is not met. 

For infrastructure, you can scale up by moving to a more powerful machine, or scale out to move to distributing the load across multiple smaller machine.

### Maintainability
The majority of the cost of software is not in its initial development, but in its ongoing maintenance, fixing bugs. It is good to keep the system simple, removing the complexity as possible, and make it evolvable, and extensible. Small software projects can be composed of simple and expressive code, but as the project gets larger they often become very complex to understand.

Making a system simpler can be done by removing accidental complexity. __Complexity is accidental if the problem that the software solves does not arise only from the implementation.__

One of the best tools to remove accidental complexity is abstraction: for example, programming languages abstract the machine, CPU, and syscalls.

# 2. Data Models and Query Languages
Most applications are built by layering one data model on top of another. As an application developer, you look at the real world, and model it in terms of objects, those objects are stored in JSON, XML, tables. Some engineers have decided the way to represents JSON, XML in terms of bytes in memory, etc.

## SQL

SQL is the most popular and efficient implementation of the relation model, where data is organized into relations, each relation is an unordered collection of tuples. One drawback of SQL is the awkward translation between the object in the application and in the database.

One advantage of SQL is to provide better support for joints, many to one, and many to many relationships.

You can also store binary data in your SQL database. They are called `blob`, and depending on the database engine, they are stored separately and in the index. If you want to store images, for example, you can only reference the path to the image in another database, or use blob. Blob allows backup to be simpler to manage, and replicas and security come for free.

## NoSQL

NoSQL does not refer to any particular technology, but it was intended as a catchy Twitter hashtag for a meeting. A lot of databases fall into this hashtag.

* Key-value stores: data is stored in an array of key-value pairs like in Redis
* Document database: documents are saved, and grouped as collections. Each document can have a completely different structure.  Saving things in JSON with MongoDB is way simpler. The JSON representation has better locality than the multi-table schema. If you want to fetch object, you can perform multiple queries or a messy join, or you can represent the relevant information in one place, and automatically access it. 
* Graph database: It is composed of vertex and edges. It can be stored in a relational database with two relations: edges, and vertices. Any vertices can have an edge connection with any other vertex. By using different labels for different kinds of relationship, you can store multiple pieces of information in a single graph. You can extend the graph very easily. In a graph database, you can use declarative query languages such as SPARQL, Neo4J.

### Which data model leads to simpler application code?

* Document model if
  * the data in your application can be represented as a tree of one to many relationships where the entire tree is loaded at once
  * no relationship between records
  * data locality for queries, the document is usually stored as a continuous string such as JSON, XML
* Relational Model if
  * can access an item directly (in document, item are nested, and you need to retrieve the access path)
  * highly interconnected data (even though graph models are better)
  * start to support storing and querying with JSON and XML

The **CAP** Theorem: You can't build a distributed system that is

* **C**onsistent sequentially, all nodes see the same data at the same time
* **A**vailability: continuously available, every request get a response
* **P**artition tolerant (to any partition failures) system works even in case of message getting lost

All data models have a default configuration, which makes them more CA, CP, AP, but you can change the configuration to make them behave differently for your application.

* MongoDB is CP: consistent because of the Single Leader architecture, Partition Tolerant
* MySQL is AC: Leader, follower architecture. If Follower can't connect to Leader, they will create their own Leader
* Cassandra is AP: always available (nodes + replicas), and partition tolerant as all nodes are fully independent



# 3. Storage and Retrieval
## The data structures that power your database
The simplest database in the world:
```
#!/bin/bash
db_set () {
 echo "$1,$2" >> database
}
db_get () {
 grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```
This implementation has a very good complexity for adding a new entry in the database, but it's 0(n) for retrieving elements. To improve the complexity you can use a data structure like an `Index`. The general idea is to keep some additional metadata on the side, to help you locate the data you want. This structure usually only improve the performance of the queries.

### HashMap
For example, Bitcask is a storage system that has a very simple implementation: it uses a hash map to store key-value pairs mapping value to offset in a file.

Bitcask also keeps a log where all pairs are stored. If this log becomes too large, the database performs compaction on the data, by removing old pair that contained a key that has been updated. Furthermore, the log is appended only, which make it very fast to sequential write. The file format for the log is some sort of raw binary encoding.

* HashMaps have some limitations: you can't do range queries

### Indexes

It is a data structure that order values in a list. An index is usually represented as a list of pairs. In the list, it stores the value ordered, and also a pointer to the objects containing this value.

For MYSQL, a value is the value of a certain column (usually the id), and a pointer refers to the row in the table.

It also to retrieve quickly values, and do range operations.

Indexes accelerates read queries, but slow down the write query, and take more place in memory

### BTrees
* It's the most widely used indexing structure!

#### Structure
* B-trees keeps pairs of key-value sorted by key. B-trees break the database down into fixed size pages. Each page can be identified by using an address, which allows one page to refer to another. 
* B-trees are stored on disk
* There is one page that is designated as the root of the B-tree for whenever you want to look up a key in the index. The page contains k keys and k+1 references to child pages. Each child is responsible for a continuous range of keys 

```bash
root page:   [ref page 1, 100, ref page 2, 200, ...]
ref page 1:  [0, ref page 11, 20, ..., 80]
...
ref page 11: [1, val 1, 2, val 2, 3, val 3]
```

#### How to update values
* Search the leaf page containing the key, update the value, and write back to disk. If the key doesn't exist, add it to that page. If the page has reached is size, split it in two, and backtrack this information to the parent pages
* Algorithm ensure that the tree remains balances

###  Resiliency to crash

* Include an additional data structure on disk: a write ahead log. Append-only that store every B-tree modification, before applying them.

## Using database for data analytics
The basic access pattern of the database consists of a series of query and update, known as online transaction processing (OLTP). 

This pattern is completely different from the online analytic processing (OLAP). For this, big companies build a data warehouse that would store every OLTP database to make them available for analysis. This data warehouse is very good at answering analytic queries. Both OLAP, and OLTP uses MYSQL, but the underlying systems are optimized for different query patterns.

## Schema for analytics
Many data warehouses are used in a very formulaic style known as a star schema.
The star schema consists in a single read-only array of rows, called fact table, where each row represents a single event, and each column stored a value or a reference to the index of an OLTP database. For large companies, the number of rows can be extremely large.

### Data structure for analytics
* Optimizing the arrangement for queries: Instead of storing the fact tables by rows, where each row is stored next to each other, you will store them in columns. The reason is that analytic queries only look at some columns at a time, and doesn't require to load all the columns.
* Optimization of the space in memory: If the number of values is smaller compared to the number of rows, bitmap can be created to replace values
* In a column store, there is no order, because insertion of news rows is done in an append after way. 
* Data warehouse also store aggregate statistics of the value in the fact table. For this, a hypercube is created containing the sum/mean... where each dimension of a cube is a fact table column, and each row for a dimension is a unique value of that fact table column.

# 3. Encoding and evolution
Making sure that you can evolve the software, without having to down all the service when the updates are going on (rolling upgrade), but also make sure that user who hasn't updated their version can still connect using your service. For this, you need to maintain backward-forward compatibility.

## JSON, XML, and binary variants
* In XML and CSV you cannot distinguish between a number and a string
* JSON, and XML takes more space than binary formats. That's why it exists binary encodings for JSON, but these formats are still seen as niches.
* There exists schema in JSON, and XML, but are not widely used

## Protocol Buffers
* PB requires a schema to run. This schema can be compiled into objects for different languages. This generated code is useful for statically typed language, but become useless for languages like Python. 
  To maintain backward compatibility, after the first release, you cannot add new fields that are required to a protobuf, you can only make them optional, or set them to a default value. 
  You also have to make sure that you never removed a required field, but only one that is optional. 
  Furthermore, you should not change tag number for each field. You can change its name but not the field.

## Database
Schema evolution in the database can be handled by adding new columns. If code reads an old entry, the new column field will be set to NULL.

## Data flow through services
When you have processes that need to communicate over a network, there are different ways of arranging this communication. A classical approach is to have an API exposed over the network by a server and a client that connect to the server.

### REST
REST is not a protocol, but rather a design philosophy that builds upon the principles of HTTP. It uses simple data formats, such as JSON, and URLs for identifying resources. HTTP is used for cache control, authentication, content type negotiation. To cache control, Varnish can help. It is a web application accelerator that cache HTTP query/answer on top of any server so that similar queries get cached and not computer over and over.

### Remote Procedure Call (RPC)
Make a request to a remote network service that will look the same as calling a function in your programming language.
Sadly, network response is unpredictable, no efficiency because you can't use pointer 
An implementation in Java has been made, it is Remote Method Invocation RMI.

### Asynchronous message passing
A message is not sent via a direct network but goes to an intermediary message broker, that can act as a buffer in case of server overloading. One implementation is RabbitMQ. It uses queues where `Publisher` publishes data that `Consumers` can consume. For example, clients are the publishers and the worker are the consumer of this queries.

# 4. Distributed Data
Why distributing data:
* Scalability
* Fault tolerance
* Latency

## Scaling to higher load
There are two approaches to scaling:

* You can scale vertically which means buying better machine, 
* You can scale horizontally, which mean you run a lot of cheap machines, that you connect to a conventional network. You can use whatever machines at the best prices, and distribute the data across multiple geographic regions.

Scaling horizontally comes at a cost of more complex application code, and less expressiveness in the data model you can use.

# 5. Replication
Why replicating?
* keep data geographically close to your users
* allow the system to continue working even if some part fails

It is simple to replicate, you just need to copy the dataset. What is complicated is to handle changes to replicate data. 

## One Leader, Multiple Followers
The first strategy is to have a leader that receives the write and store them. The other replicas, the followers, look at the log of the leader and updates their local copy of the database. Then when a client wants to read, it can read from the leader or the followers.
This replication mode can be used as default in multiple data models: MongoDB, MYSQL
Replication can be synchronous: the leader needs to wait for one follower to confirm it has written the changes, before sending the OK message to the client. It is impractical to have all followers that are synchronous.
However, most of the implementation relies on asynchronous replication, because replicas might be far away, and the leader needs to continue processing writes.

### Potential failure
If one follower fails, it can keep a log of the data changes on a disk, so as to reconnect correctly with the leader and ask for the changes
If the leader fails, the infrastructure needs to realize that the system has failed, choose a new leader, and reconfigure the system so that the system can use the new leader.
There are a lot of possible conflicts and error that can happen in this system. One, for example, is a user that write to the leader, but read to a follower, and doesn't see his changes, because writes are asynchronous. To prevent this, some heuristics need to be implemented so as to always show to the user his latest changes, even if it requires a follower to contact the leader
Users can also read from different followers, that receive data in different order. To prevent this, you can add a write causality, and force follower to write data in order, queueing messages if fragment in between is missing.

## Multiple Leaders
In the multiple leader's infrastructures, you might have multiple data center across the world, each data center having a Leader. People can write to this Leader. The Leader is responsible to forward new write to each of its Follower and to all the other Leader. When the write message arrived at another Leader, it usually passes through a conflict resolver. 
Multiple Leader infrastructures bring tolerance to fault in a datacenter, increase in performance as client writes are sent to the closest Leader. 

An Extreme example: if you need to an application that continues to work even without internet, you can consider that every device is a Leader, and it becomes an asynchronous multi Leader replication process between the replicas of your calendar on all devices. 

Collaborative contributing (like Google Docs) can be seen as a database replication problem, where a document (local replication) needs to send their changes to the other replicates.

### Problem with this approach: concurrent writes
#### Definition: concurrent event
In distributed systems, concurrent things occur if they are both unaware of each other, regardless of the time they occurred at.

With a single Leader, it is possible to catch concurrent writes, as they arrived sequentially, and are treated sequentially. But when there are multiple Leaders, the error is detected after writes are validated locally and send to the other Leader asynchronously.    

There are multiple approaches to solve concurrent writes, generic one, and application specific one:

* generic one: have a universal timestamp and always keep the latest timestamp write, but it is prone to data loss.
* specific one: operational transformation, for example, is the underlying algorithm for Google Docs. There is a conflict manager in each replica that receives the changes from a server. Each replica remembers the previous operation and the last operation, and the server reorder this operation and send the changes to each replica. 
  To conclude it is always good to send new actions, and the state that leads to this action, so as to be able to merge concurrent actions in a certain order

## No Leaders
There exist implementation where there is no Leader, in this case, the client sends to multiple replicas at the same time.  


# 6. Partition/Sharding
Why Partitioning?
* Scalability: large dataset can be distributed across many disks

## Partitioning methods

* Horizontally partitioning: partition the values of each column into ranges
* Vertical partitioning: divide the data to store tables related to a specific feature
* Directory based partitioning: Query a service that knows where the data is. 

## Partition Criteria

### Random partionning

One simple idea is to partition the data randomly between nodes. The problem of such a solution is when you try to read a particular item, you have no way of knowing which node it is on, so you need to query all nodes in parallel

### Round Robin partitioning

Very simple strategy, with n partitions the `i` tuple, is assigned to partition `i % n`. The problem with this approach is that if the number of nodes changes over time, you need to move all the data to a new node.

### List partitionning

Heuristic to group values into certain partition, for example, affect all query in a certain geographic position to the same node

### Key or Hash partitioning

* Another idea is to partition by key range. This partition is very simple to implement, but keys are not necessarily uniformly spaced. Even if it simple, you need to think about which key to partition your data on partitioning by timestamp makes it easy to query a range of values, but becomes useless, as all the write queries go to the same node every day.
* You can also try to partition by the hash of a key. But with this technique in mind, you lose the property of key-range partitioning

## Problem with partitioning

### Joins over multiple databases

Performing joins result in contacting multiple databases, which becomes very inefficient. A workaround to this problem is to denormalize the database

###  Rebalancing partitions

Sometimes, in a cluster, you have machine failure, or you want to add new disks, RAMS, CPU in the cluster. If the data/computation is partitioned you want to rebalance the load fairly across all the new nodes, without having to shut down the services

**Consistent Hashing** is a technique to minimize the amount of data that needs to be moved if any node goes up or down. The technique uses a hash function that distributes values between two ranges. The boundaries can be joined to form a circle. We put node on this circle and save object to the closest node that has the hash just smaller than the object hash

#### Fixed number of partitions

A solution is to start with N way larger than the number of nodes. For example, if the number of nodes is 10, you create 1000 partitions, assign multiple ones to nodes, and when a new node is created, some node will give some of their partitions to new nodes.

####  Dynamic partitioning

Each partition can store between m to M values. If the number of objects becomes larger than M, the partition is split, in contrary, if there are too few objects, the partition is merged with another.



## Partitioning secondary index
Sometimes you can't only partition by a unique key because you want people to be able to query your database with more than a certain key. To do that you use a secondary index. For example, a selling car website might store an advertisement as a document, and partition document to different nodes. When you query for a blue car, you will contact a secondary index that store pair of color attribute and document index, so as to retrieve the document for a given attribute. There are two approaches to partition secondary index:
* Partition by the document: Every secondary index in every partition has all attributes possibles for every document in each partition. When a user wants to gather the results, he has to contact all the nodes
* Partition by term: Ever secondary index in every partition contain certain attributes, and no two partitions contain the same attributes. When looking for an attribute, you find the partition that store the document index for this given this attributes, and contact only the nodes that contain the document you want. 
### Comparison
* Partition by term: Faster Read
* Partition by document: Faster Write



# 7. Transactions
Transactions is a way for an application to group write and read into a logical unit. All the read an write are executed as one operation. A transaction can be committed, or be aborted, or rollback.
Transactions have been defined to be
* Atomic: Something that cannot be broken into smaller part, whether it is fully accepted, or whether it is rejected, but there is no intermediary state, even in the case of middle failure
* Consistency: Maintain a consistent state of the database after applying the transaction. A consistent state is ruled by statements that must always be true
* Isolated: Concurrent transactions are isolated from each other. If a transaction makes several writes, another concurrent transaction should see either the state before all the writes, or after all of them, but not a subset
* Durable: If a transaction is committed, any data it has written will not be forgotten. In theory, this is not possible to handle, but with replication, you can be sure it won't happen

The basic level of transaction is read committed:
* no dirty read: user read data that always has been committed
* no dirty write: update the same object at the same time before both transactions have committed

To prevent dirty write, you can ensure that whenever a transaction modify an object, it gets also a locker that it will hold until the transaction gets committed
To prevent dirty read, keep the old and new value, and update the old value only when the transaction is committed.

You also want to avoid write s=ke, which is when two transactions occur at the same time, and read a same value at the same time, and commit. The consequence is that the first transactions have invalidated the second one, but the second one is not aware of the first one, so both get accepted.   

# 8. The trouble with Distributed Systems 

So many things can go wrong in a distributed systems. In an Internet Distributed Systems, you often care about

* The service is always available
* Nodes are built with cheap hardware, that have high failure rate
* Always assume that something is broken
* Sometimes even if `TCP` acknowledges that a packet was delivered, the application may have crashed handling the request

## Unreliable Networks

In an asynchronous system, it is common that a sender can't tell whether a packet was delivered, because it might have never arrived, or the response was lost. Most of the system use `timeout` to request again when no response came.

The `timeout`is not an easy task to set. Low timeout tends to incorrectly detect dead node. A long timeout may force the user to wait. It is possible to do some statistics about the average daily or request handling, but in an asynchronous system, you never know if your message will be treated quick

## Networking congestion and queueing

The queue of incoming message of a node becomes too large so the queue must drop the latest messages, that would need to be resent afterward.

`TCP` performs flow control (congestion avoidance) in which a node limits its own rate of sending in order to avoid overloading a  network link, not the receiving node before the data even enters the network.

## Synchronous vs asynchronous network

An example of a synchronous network is the fixed line telephone network. Every call creates a channel between the two participants, with an allocated bandwidth and both parties at the end, knows when to receive messages, the quantities of messages. On the internet, it is impossible to predict the bandwidth requirement. If you want to transfer a file over a circuit you would have to guess a bandwidth allocation

## The truth is defined by the majority

Most of the times, in distributed systems, there is a single object that has the right to do one thing, for example writing to a database.  

When using a lock to protect access to some resource, we ensure that a node who is under a false belief of being the one cannot disrupt the rest of the system. A technique to do so is called fencing:

* A server is responsible to grant access to a database by providing a lock with an expiration date. To write to the database, a node must have the last lock released by the lock service. This check is done by the database that only accepts write a query with the latest lock seen so far.

# 9. Consistency and Consensus

## Linearizability

Make the system appear as if there was only one copy of the data, and all operations are atomic. In a linearizable system, there is a total order of the operations. 

Two events are ordered if they are causally ordered. If two events are incomparable, it means that they are concurrent. If such events exist in a system, this system is not linearizable.

It is very hard to enforce linearization for a system, but you can try to enforce a certain order:

* each node generates its own set of timestamp (`counter`, `nodeID`), each node keeps a queue for all the other about incoming messages
* attach a timestamp to every event, hoping that this will totally order operations when merging operation. In this case, a merging technique can be `last write wins`