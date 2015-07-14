---
layout: post
title:  "Cassandra Practical and Study Notes"
date:   2015-06-24
categories: Study-Notes
excerpt:
comments: true
---

* content
{:toc}

### disk failure, and read/write in multiple DC

read quorum, find enough replicas from ANY DC is enough. Write quorum, must satisfy consistency level of all DCs.

We encountered a disk failure in spark node, then all write OP is failed. when a new disk is attached back to the spark node, the user/password does not work, then failure stills. We change the user with another, then it works. 



### Counter

if a table contains counter, all non-primary column should be in counter type.

Also counter table should use update statement only. can past a = a+ ? the ? can be passed in. 


### [data model](http://www.datastax.com/documentation/cql/3.1/cql/ddl/ddl_music_service_c.html)

With the schema as given so far, a query that includes the artist filter would require a sequential scan across the entire playlists dataset. Cassandra will reject such a query. If you first create an index on artist, Cassandra can efficiently pull out the records in question.
CREATE INDEX ON playlists(artist );

### keys

A compound primary key includes the partition key, which determines on which node data is stored, and one or more additional columns that determine clustering. Cassandra uses the first column name in the primary key definition as the partition key. For example, in the playlists table, id is the partition key. The remaining column, or columns that are not partition keys in the primary key definition are the clustering columns. In the case of the playlists table, the song_order is the clustering column. The data for each partition is clustered by the remaining column or columns of the primary key definition

Using a composite partition key

A composite partition key is a partition key consisting of multiple columns. You use an extra set of parentheses to enclose columns that make up the composite partition key. The columns within the primary key definition but outside the nested parentheses are clustering columns. These columns form logical sets inside a partition to facilitate retrieval.

~~~ mysql
CREATE TABLE Cats (
  block_id uuid,
  breed text,
  color text,
  short_hair boolean,
  PRIMARY KEY ((block_id, breed), color, short_hair)
);
~~~

For example, the composite partition key consists of block_id and breed. The clustering columns, color and short_hair, determine the clustering order of the data. Generally, Cassandra will store columns having the same block_id but a different breed on different nodes, and columns having the same block_id and breed on the same node.


### Collection columns

CQL introduces these collection types:

* set
* list
* map

When to use a collection

Use collections when you want to store or denormalize a small amount of data. Values of items in collections are limited to 64K. Other limitations also apply. Collections work well for storing data such as the phone numbers of a user and labels applied to an email. If the data you need to store has unbounded growth potential, such as all the messages sent by a user or events registered by a sensor, do not use collections. Instead, use a table having a compound primary key and store data in the clustering columns.

~~~ mysql
 SELECT * FROM playlists 
      WHERE album = 'Roll Away' AND title = 'Outside Woman Blues' 
      ALLOW FILTERING ;
~~~

When multiple occurrences of data match a condition in a WHERE clause, Cassandra selects the least-frequent occurrence of a condition for processing first for efficiency. For example, suppose data for Blind Joe Reynolds and Cream's versions of "Outside Woman Blues" were inserted into the playlists table. Cassandra queries on the album name first if there are fewer albums named Roll Away than there are songs called "Outside Woman Blues" in the database. When you attempt a potentially expensive query, such as searching a range of rows, Cassandra requires the ALLOW FILTERING directive.


The partitioning key is used to distribute data across different nodes, and if you want your nodes to be balanced (i.e. well distributed data across each node) then you want your partitioning key to be as random as possible. That is why the example you have uses UUIDs.
The clustering key is used for ordering so that querying columns with a particular clustering key can be more efficient. That is where you want your values to not be unique and where there would be a performance hit if unique rows were frequent.

The main difference from SQL is that Cassandra does not support joins or subqueries, except for batch analysis through Hive. Instead, Cassandra emphasizes denormalization through CQL features like collections and clustering specified at the schema level.

**Gossip**: A peer-to-peer communication protocol to discover and share location and state information about the other nodes in a Cassandra cluster.

Gossip information is also persisted locally by each node to use immediately when a node restarts. You may want to purge gossip history on node restart for various reasons, such as when the node's IP addresses has changed.

Partitioner: A partitioner determines how to distribute the data across the nodes in the cluster. Choosing a partitioner determines which node to place the first copy of data on.

You must set the partitioner type and assign the node a num_tokens value for each node. If not using virtual nodes (vnodes), use the initial_token setting instead.

Replica placement strategy: Cassandra stores copies (replicas) of data on multiple nodes to ensure reliability and fault tolerance. A replication strategy determines which nodes to place replicas on. The first replica of data is simply the first copy; it is not unique in any sense.

When you create a keyspace, you must define the replica placement strategy and the number of replicas you want.

**Snitch**: A snitch defines the topology information that the replication strategy uses to place replicas and route requests efficiently.
You need to configure a snitch when you create a cluster. The snitch is responsible for knowing the location of nodes within your network topology and distributing replicas by grouping machines into data centers and racks.
The cassandra.yaml file is the main configuration file for Cassandra. In this file, you set the initialization properties for a cluster, caching parameters for tables, properties for tuning and resource utilization, timeout settings, client connections, backups, and security.

Cassandra stores table properties in the system keyspace. You set storage configuration attributes on a per-keyspace or per-table basis programmatically or using a client application, such as CQL.

By default, a node is configured to store the data it manages in the /var/lib/cassandra directory. In a production cluster deployment, you change the commitlog-directory to a different disk drive from the data_file_directories.

Node failures can result from various causes such as hardware failures and network outages. Node outages are often transient but can last for extended intervals. A node outage rarely signifies a permanent departure from the cluster, and therefore does not automatically result in permanent removal of the node from the ring. Other nodes will periodically try to initiate gossip contact with failed nodes to see if they are back up. To permanently change a node's membership in a cluster, administrators must explicitly add or remove nodes from a Cassandra cluster using the nodetool utility.

When a node comes back online after an outage, it may have missed writes for the replica data it maintains. Once the failure detector marks a node as down, missed writes are stored by other replicas for a period of time providing hinted handoff is enabled. If a node is down for longer thanmax_hint_window_in_ms (3 hours by default), hints are no longer saved. Because nodes that die may have stored undelivered hints, you should run a repair after recovering a node that has been down for an extended period. Moreover, you should routinely run nodetool repair on all nodes to ensure they have consistent data.


When your create a cluster, you must specify the following:

* Virtual nodes: assigns data ownership to physical machines.
* Partitioner: partitions the data across the cluster.
* Replication strategy: determines the replicas for each row of data.
* Snitch: defines the topology information that the replication strategy uses to place replicas.

Virtual nodes

Overview of virtual nodes (vnodes).

Vnodes simplify many tasks in Cassandra:

You no longer have to calculate and assign tokens to each node.

Rebalancing a cluster is no longer necessary when adding or removing nodes. When a node joins the cluster, it assumes responsibility for an even portion of data from the other nodes in the cluster. If a node fails, the load is spread evenly across other nodes in the cluster.

Rebuilding a dead node is faster because it involves every other node in the cluster and because data is sent to the replacement node incrementally instead of waiting until the end of the validation phase.

Improves the use of heterogeneous machines in a cluster. You can assign a proportional number of vnodes to smaller and larger machines.

Starting in version 1.2, Cassandra changes this paradigm from one token and range per node to many tokens per node. The new paradigm is called virtual nodes (vnodes). Vnodes allow each node to own a large number of small partition ranges distributed throughout the cluster. Vnodes also use consistent hashing to distribute data but using them doesn't require token generation and assignment.

Data replication

Cassandra stores replicas on multiple nodes to ensure reliability and fault tolerance. A replication strategy determines the nodes where replicas are placed.

The total number of replicas across the cluster is referred to as the replication factor. A replication factor of 1 means that there is only one copy of each row on one node. A replication factor of 2 means two copies of each row, where each copy is on a different node. All replicas are equally important; there is no primary or master replica. As a general rule, the replication factor should not exceed the number of nodes in the cluster. However, you can increase the replication factor and then add the desired number of nodes later. When replication factor exceeds the number of nodes, writes are rejected, but reads are served as long as the desired consistency level can be met.

Write requests

The coordinator sends a write request to all replicas that own the row being written. As long as all replica nodes are up and available, they will get the write regardless of the consistency level specified by the client. The write consistency level determines how many replica nodes must respond with a success acknowledgment in order for the write to be considered successful. Success means that the data was written to the commit log and the memtable as described in About writes.

For example, in a single data center 10 node cluster with a replication factor of 3, an incoming write will go to all 3 nodes that own the requested row. If the write consistency level specified by the client is ONE, the first node to complete the write responds back to the coordinator, which then proxies the success message back to the client. A consistency level of ONE means that it is possible that 2 of the 3 replicas could miss the write if they happened to be down at the time the request was made. If a replica misses a write, Cassandra will make the row consistent later using one of its built-in repair mechanisms: hinted handoff, read repair, or anti-entropy node repair.

Multiple data center write requests

In multiple data center deployments, Cassandra optimizes write performance by choosing one coordinator node in each remote data center to handle the requests to replicas within that data center. The coordinator node contacted by the client application only needs to forward the write request to one node in each remote data center.
If using a consistency level of ONE or LOCAL_QUORUM, only the nodes in the same data center as the coordinator node must respond to the client request in order for the request to succeed. This way, geographical latency does not impact client request response times.

### design Cassandra DB

Summary

We’ve covered a few fundamental practices and walked through a detailed example to help you get started with Cassandra data model design. Here are the key takeaways:

* Don’t think of a relational table, but think of a nested sorted map data structure while designing Cassandra column families.
* Model column families around query patterns. But start your design with entities and relationships, if you can.
* De-normalize and duplicate for read performance. But don’t de-normalize if you don’t need to.
* Remember that there are many ways to model. The best way depends on your use case and query patterns.
* Storing values in column names is perfectly OK
* Leaving column values empty (“valueless” columns) is also OK.

It’s a common practice with Cassandra to store a value (actual data) in the column name (a.k.a. column key), and even to leave the column value field empty if there is nothing else to store. One motivation for this practice is that column names are stored physically sorted, but column values are not.

Make sure column key and row key are unique

Otherwise, data could get accidentally overwritten.

In Cassandra (a distributed database!), there is no unique constraint enforcement for row key or column key.

Also, there is no separate update operation (no in-place updates!). It’s always an upsert (mutate) in Cassandra. If you accidentally insert data with an existing row key and column key, the previous column value will be silently overwritten without any error (the change won’t be versioned; the data will be gone).

* writes to commitlogs, to memtable, flushing (bloom filter) to sstable(disk), multiple sstables perform compaction for deleting old.

Bloom filter (all keys in data file). A Bloom filter, is a space-efficient probabilistic data structure that is used to test whether an element is a member of a set. False positives are possible, but false negatives are not. Cassandra uses bloom filters to save IO when performing a key lookup: each SSTable has a bloom filter associated with it that Cassandra checks before doing any disk seeks, making queries for keys that don't exist almost free. Bloom filters are surprisingly simple: divide a memory area into buckets (one bit per bucket for a standard bloom filter; more -typically four - for a counting bloom filter). To insert a key, generate several hashes per key, and mark the buckets for each hash. To check if a key is present, check each bucket; if any bucket is empty, the key was never inserted in the filter. If all buckets are non-empty, though, the key is only probably inserted - other keys' hashes could have covered the same buckets. See All you ever wanted to know about writing bloom filters for details and in particular why getting a really good output distribution is important.


### Source

[architechture wiki](http://wiki.apache.org/cassandra/ArchitectureOverview)

[Link](https://wiki.apache.org/cassandra/MemtableSSTable)
