Some information about Memcached, Redis, MongoDB, Cassandra, HBase, Kafka

### Redis

In memory key data structure store. It is a cache to store object like string, list, set, sorted set, hash, binary file

- You can enforce persistence to the disk by dumping the database, or dump all the operations done so far
- You can enforce replication (implemented as a Leader/Follower)
- You can store binary file up to 512MB for each key
- Here are some operations:
  - SET key value
  - GET key
  - INCR key (atomic incrementation)
  - EXPIRE key 120 (sec)
  - Push to list (left or right) RPUSH and LPUSH and get with LRANGE key index_depart index_fin
  - Union of list SUNION
  - Push to set, or an ordered set: this is useful to maintain a list of ordered objects
  - You can also store object. Each object is represented as a list of key-value pair. You can retrieve one attribute of this object or all of them.
  - You can PUBLISH a channel PUBLISH channel data (str, formated str), and another client can SUBSCRIBE to channels, and retrieve in real time values
  - If you want an overview of your database, you can look at keys, as you would look for tables in MySQL
- Data type: List and Set: Up to 4 billion value per list.

#### Comparison with Memcached

- Same latency and throughput
- Redis is more powerful and flexible. Redis has been built after Memcached

# NOSQL

Neither MongoDB nor Cassandra is good if you need an ACID database, with consistency and normalization

### MongoDB

In MongoDB, objects are stored as collections. Objects are anything that can be stored as JSON.

* DIfferents fields in every document if you want
* A lot of flexibility


* In the CAP triangle, MongoDB offers Consistency and Partition Tolerance overs Availability

#### Replicas

* Single Leader and Followers
* Election if Leader goes down

##### Disadvantages

Replicas are here to address durability not scalability

A majority of the server must agree to elect the Leader. You can't have replication and an even number of nodes

#### Partitioning

You can use `mongos` which are routing service that processes queries from the application layer and determines the location of the data on some shared cluster. Note that the shared cluster might contain multiple Leaders that will always be affected a range of values

`mongos` also provides a load balancer to rebalance range of values across the different nodes

####  Commands

#### Create a collection

You create a collection with ```db.createCollections(name_collection)```

#### Insert values

You add an element in the collection using MongoDB ```db.name_collection.insert([{name: "Paul"}, {name: "Paulhino", "surname": "Polo"}])```

It doesn't matter if one object does not contain the same attribute as the other one in the collection.

#### Update values

```db.name_collection.update(json_query, json_new_values)```

The ```json_query```  might look like ```{"name": "Someone"}```, and the JSON new value will be the new attributes for the matched objects. When updating all the matched objects, get replaced by the new attributes, and if new attributes are not set again, they disappear.

To remedy this, you can only append/update attributes to matched objects with ```db.name_collection.update(json_query, j${set:{attribute: value}})```

Apart from `set`, you can also do `inc:{attribute:incr_value}`, or `unset:{attr:1}`. 

#### FInd values

Find used an underlined index data structure  (like BTree). You can specify the usage of a certain index if you want

```db.name_collection.find({attr1.attr2.attr3: "value"})``` for values

```db.name_collection.find({list:"val"})``` if attr is list, it will search for occurence of val in the list

##### Querying with MapReduce in MongoDB

Map reduce is a programming model for processing large amounts of data. Some NoSQL including MongoDB supports read-only queries across many documents. I

However, the uses are quite restricted: 

- Before using map reduce, you need to filter documents that you want to process, 
- Then you map each document to a pair of key, value, and use the reduce to operate some transformation/calculation on the data. 

For MongoDB, you can use Javascript to write these two functions.

### Cassandra

It is a distributed database, consisting of nodes that don't share anything. Each node is responsible for a certain portion of the dataset. If you had more node, you get a linear relation throughput **for writing**. 

#### Writes

Cassandra writes on a single node are **very** fast. It comes to a node, gets logged, appends to a buffer, and acknowledge to be processed by the system back to the user. When the buffer becomes too large, it is saved in a file. Regularly Cassandra jobs perform compaction to remove duplicates, and out of date entries.

##### Partition

Data get partitioned across different nodes using a consistent hashing function. 

##### Replication

All node is responsible to store the data from the range of another node, so as to replicate the written data. Write on the Leader and the Replicas are synchronous

#### Reads

For reading in the distributed cluster, all nodes are aware (using a consistent hash) of where the data might be and ask to all leaders and replicas about the key. The node receives multiple responses, and look for similar ones (this is a consistency parameter to set) before sending the messages.

In Cassandra, you can turn down and up to any node without affecting the cluster.

#### Data Model

* key-value architecture
* data are grouped by partitions which consist of rows sharing the same primary key. In Cassandra, it is good to aim at minimizing this partition query
* you can duplicate and denormalize data
* spread the data evenly around the cluster, but cluster data by queries

#### Implementation

* Insert key-values, only keys as to always be present, new columns can be inserted, or cannot be filled. You can specify a TTL for when the row should disappear

### HBase

Hbase is stored on HDFS filesystem and based on the Google BigTable. It is a column family oriented database. Data can be retrieved with MapReduce

If the immediate question is about how to access this data, and how to group it, rather than how to describe the entity and the interaction with others, then HBase is a good tool to work with.

You have to think about access pattern

#### Data model

Data is stored as a column based database. 

* Instead of storing by rows, HBase stored by columns. 
* It can handle missing values for certain column. For this, it just doesn't store anything, not even a NULL. 
* Columns are retrieving using a key value
* Columns values don't have to of the same type
* It groups multiple columns into a column family. Column family are not stored in the same place
*  It can store past value of a certain entry
* Getting this primary key right requires time and effort. You only have one index or primary key. Multiple pieces of data are usually in the row key

#### Partition

* Tables can be broken into smaller pieces called Regions.
* BigTable divided table by their row identifier across multiple RegionServer

#### Read and Write

* Client talks directly to a RegionServer for reading and writing

#### Streaming

Streaming let you publish data in real time. 

How to get data from many different sources into the clusters

### Kafka

It is a publish/subscribe messaging system. Store message coming from `Publisher` and send them whoever has subscribed to it and want to consume it. Kafta consumers subscribe to `topics`. 

A topic can have many different consumers with their **own position in the stream** maintained.

It is not only for `Hadoop`.

#### Architecture

```
              <-------------> Connectors
              |
Producers -> Kafka cluster -> Consumers
              |
              <-------------> Stream Processors
```

Connectors can be plugin module that can publish or subscribe and save. You can set up a database to publish and subscribe



## HTTP Server

### NGINX [n-ginix]

Server HTTP dedicated that is very efficient at scaling up and down given the number of requests

### Apache Server

Easy to configure and setup, can use `.htaccess` to rewrite URL received and send them to the server. This is not something possible in NGINX, you have to modify the configuration of your server