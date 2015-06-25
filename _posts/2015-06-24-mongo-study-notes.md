---
layout: post
title:  "Mongo Study Notes"
date:   2015-06-24
categories: GitHub
excerpt: git
comments: true
---

* content
{:toc}

### Start mongo server

~~~ shell
    mongod --config /usr/local/etc/mongod.conf
~~~

or have launchd start mongodb at login:

~~~ shell
    ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents
~~~

Then to load mongodb now:

~~~ shell
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
~~~

Or, if you don't want/need launchctl, you can just run:

~~~ shell
    mongod --config /usr/local/etc/mongod.conf
~~~

All MongoDB documents must have an _id field with a unique value. These operations do not explicitly specify a value for the _id field, so mongo creates a unique ObjectId value for the field before inserting it into the collection.

MongoDB stores data in the form of documents, which are JSON-like field and value pairs. Documents are analogous to structures in programming languages that associate keys with values (e.g. dictionaries, hashes, maps, and associative arrays)
Return Two Fields and Exclude _id

~~~ shell
db.records.find( { "user_id": { $lt: 42} }, { "_id": 0, "name": 1 , "email": 1 } )
~~~

### Data model
Data in MongoDB has a flexible schema. Unlike SQL databases, where you must determine and declare a table’s schema before inserting data, MongoDB’s collections do not enforce document structure. This flexibility facilitates the mapping of documents to an entity or an object. Each document can match the data fields of the represented entity, even if the data has substantial variation. In practice, however, the documents in a collection share a similar structure.
In general, use normalized data models:

when embedding would result in duplication of data but would not provide sufficient read performance advantages to outweigh the implications of the duplication.
to represent more complex many-to-many relationships.
to model large hierarchical data sets.

These factors are operational or address requirements that arise outside of the application but impact the performance of MongoDB based applications. When developing a data model, analyze all of your application’s read operations and write operations in conjunction with the following considerations.
As you create indexes, consider the following behaviors of indexes:

Each index requires at least 8KB of data space.

Adding an index has some negative performance impact for write operations. For collections with high write-to-read ratio, indexes are expensive since each insert must also update any indexes.
Collections with high read-to-write ratio often benefit from additional indexes. Indexes do not affect un-indexed read operations.
When active, each index consumes disk space and memory. This usage can be significant and should be tracked for capacity planning, especially for concerns over working set size.

To improve the performance of this query, add an ascending, or a descending, index to theinventory collection on the type field. [1] In the mongo shell, you can create indexes using thedb.collection.ensureIndex() method:

~~~ shell
db.inventory.ensureIndex( { type: 1 } )
~~~

The inequality operators $nin and $ne are not very selective, as they often match a large portion of the index. As a result, in most cases, a $nin or $ne query with an index may perform no better than a $nin or$ne query that must scan all documents in a collection

Queries that specify regular expressions, with inline JavaScript regular expressions or $regex operator expressions, cannot use an index with one exception. Queries that specify regular expression with anchorsat the beginning of a string can use an index.
the query is on a sharded collection and run against a primary.
any of the indexed fields in any of the documents in the collection includes an array. If an indexed field is an array, the index becomes a multi-key index index and cannot support a covered query.
any of the returned indexed fields are fields in subdocuments. For example, consider a collectionusers with documents of the following form:

~~~ json
{ _id: 1, user: { login: "tester" } }
~~~

The collection has the following index:

~~~ json
{ "user.login": 1 }
~~~

The { "user.login": 1 } index does not cover the following query:

~~~ shell
db.users.find( { "user.login": "tester" }, { "user.login": 1, _id: 0 } )
~~~

However, the query can use the { "user.login": 1 } index to find matching documents.

Read operations on sharded clusters are most efficient when directed to a specific shard. Queries to sharded collections should include the collection’s shard key. When a query includes a shard key, the mongos can use cluster metadata from the config database to route the queries to shards.

If a query does not include the shard key, the mongos must direct the query to all shards in the cluster. These scatter gather queries can be inefficient. On larger clusters, scatter gather queries are unfeasible for routine operations.

Read operations from secondary members of replica sets are not guaranteed to reflect the current state of the primary, and the state of secondaries will trail the primary by some amount of time. Often, applications don’t rely on this kind of strict consistency, but application developers should always consider the needs of their application before setting read preference.
 
If you specify the _id field, the value must be unique within the collection. For operations with write concern, if you try to create a document with a duplicate _id value, mongod returns a duplicate key exception.

The following diagram shows the same query in SQL:

The components of a SQL UPDATE statement.

EXAMPLE

~~~ mongo
db.users.update(
   { age: { $gt: 18 } },
   { $set: { status: "A" } },
   { multi: true }
)
~~~

he following update uses the $push operator with:

the $each modifier to append to the array 2 new elements,
the $sort modifier to order the elements by ascending (1) score, and
the $slice modifier to keep the last 3 elements of the ordered array.

db.students.update(
   { _id: 1 },
   {
     $push: {
        scores: {
           $each: [ { attempt: 3, score: 7 }, { attempt: 4, score: 4 } ],
           $sort: { score: 1 },
           $slice: -3
        }
     }
   }
)

[Video Link](https://www.youtube.com/watch?v=Mz320k-WREA)

* rich document --> many operations
* inheritance --> save unused attribute as schema
* one to many --> using embedded or referencing
* many to many --> referencing ids saving in one object:

categoryid has a list of prodctids 

update lock the whole document
size of array cannot be count/query, must store in another attribute

### spring data


repository build on top of it,

mongooperations push pop in array, inc, etc

transaction in one document

async write has least durability, not good

write --> when primary dies, a new primary is promoted from secondary. 

readPreference can be set --> add multiple set

* rs.initiate()
* rs.add(hostname)
* rs.status()
* addshard

take some time to reflect in status
