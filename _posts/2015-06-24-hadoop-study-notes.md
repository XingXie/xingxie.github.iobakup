---
layout: post
title:  "Hadoop Study Nodes"
date:   2015-06-24
categories: Hadoop
excerpt: Hadoop Notes
comments: true
---

* content
{:toc}

## Hadoop 2.0
Split up the two major functions of job tracker:
cluster resource management: application life-cycle management

map reduce becomes user library, or one of the application residing in Hadoop

Key concept: Application is a job; Container: basic unit of allocation, like 2GB 1 CPU

Resource Manager: Global resource scheduler, hierarchical queues

Node manager: per-machine agent, manages the life-cycle of container, container resource monitoring

Application Master: pre-application, manages application scheduling and task execution

HDFS federation in 2.0: possible many appMaster (name node). Resource is only but task execution managements can be isolated. 


## Installation
[2.3.0 homebrew link](http://glebche.appspot.com/static/hadoop-ecosystem/hadoop-hive-tutorial.html)

[2.2.0](http://importantfish.com/how-to-install-hadoop-on-mac-os-x/)

## Map-Reduce

1. Typically the compute nodes and the storage nodes are the same, that is, the Map/Reduce framework and the Distributed FileSystem are running on the same set of nodes. This configuration allows the framework to effectively schedule tasks on the nodes where data is already present, resulting in very high aggregate bandwidth across the cluster.
2. Applications typically implement the Mapper and Reducer interfaces to provide the map and reduce methods. These form the core of the job.

JobConf is typically used to specify the Mapper, combiner (if any), Partitioner, Reducer, InputFormat, OutputFormat and OutputCommitterimplementations. JobConf also indicates the set of input files (setInputPaths(JobConf, Path...) /addInputPath(JobConf, Path)) and (setInputPaths(JobConf, String)/addInputPaths(JobConf, String)) and where the output files should be written (setOutputPath(Path)).

when the replication factor is three, HDFS’s placement policy is to put one replica on one node in the local rack, another on a node in a different (remote) rack, and the last on a different node in the same remote rack. This policy cuts the inter-rack write traffic which generally improves write performance. The chance of rack failure is far less than that of node failure; this policy does not impact data reliability and availability guarantees. However, it does reduce the aggregate network bandwidth used when reading data since a block is placed in only two unique racks rather than three. With this policy, the replicas of a file do not evenly distribute across the racks. One third of replicas are on one node, two thirds of replicas are on one rack, and the other third are evenly distributed across the remaining racks. This policy improves write performance without compromising data reliability or read performance.

Secondary NameNode: performs periodic checkpoints of the namespace and helps keep the size of file containing log of HDFS modifications within certain limits at the NameNode.

3. preparing for mapreduce: loading files (64, 128MB), file system(native file system, hdfs, cloud), output(immutable), you define(input, map, reduce, output, use java or other languages, work with key-value pairs)
4. [A nice chinese blog](http://www.cnblogs.com/sharpxiajun/p/3151395.html) [Another Chinese blog] (http://dongxicheng.org/recommend/)
5. Jobtracker bottleneck: scalability bottleneck, cluster resource sharing and allocation flexibility (slot based approach --- big or small tasks are assigned to the same slot) [source](http://www.slideshare.net/martyhall/hadoop-tutorial-mapreduce-part-6-job-execution-on-yarn) 
6. 


## Execution, Configuration, etc

1. Everything after starting all daemons, run **jps** to check
2. You must start up the server before running something
3. You don't need to create an output folder, just specify it when you run the last command, duplicated one will return errors. And the execution process/result can be viewed in terminal as well as the local host. 
4. Some useful command:
sudo hadoop jar <jarFileName> <method> <fromDir> <toDir>
hadoop fs -cat file:///      local file system place!!!
or hdfs://localhost:9090/   hdfs space
key/value out , one list per local node, can implement local reducer (or combiner)

## Tools

1. unit testing mapreduce: MRUnit + asserts optionally using ApprovalITests
2. Mahout: Library with machine learning algorithms.

recommandation: likehood-- padora
classification: Know data and new data --- spam Id
clustering (new groups of similar data -- google news)

-- what are the outputs?
can be visualized

---- limitation
batch processing, not interactive; designed for a specific problem domain; map reduce programming paradigm not commonly understood (functional) / lack of trained support professionals / API or security model are moving targes

