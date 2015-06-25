---
layout: post
title:  "Netflix Architecture Presto"
date:   2015-06-24
categories: Framework
excerpt: 
comments: true
---

[Link](http://techblog.netflix.com/2013/01/hadoop-platform-as-service-in-cloud.html)
[Link](http://techblog.netflix.com/2014/10/using-presto-in-our-big-data-platform.html)

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345146/e0a065e6-1a9f-11e5-8547-a96729c53f00.png)

**S3 as the Cloud Data Warehouse**

We use S3 as the “source of truth” for our cloud-based data warehouse. Any dataset that is worth retaining is stored on S3. This includes data from billions of streaming events from (Netflix-enabled) televisions, laptops, and mobile devices every hour captured by our log data pipeline (called Ursula), plus dimension data from Cassandra supplied by our Aegisthus pipeline. So why do we use S3, and not HDFS as the source of truth? Firstly, S3 is designed for 99.999999999% durability and 99.99% availability of objects over a given year, and can sustain concurrent loss of data in two facilities. Secondly, S3 provides bucket versioning, which we use to protect against inadvertent data loss (e.g. if a developer errantly deletes some data, we can easily recover it). Thirdly, S3 is elastic, and provides practically “unlimited” size. We grew our data warehouse organically from a few hundred terabytes to petabytes without having to provision any storage resources in advance. Finally, our use of S3 as the data warehouse enables us to run multiple, highly dynamic clusters that are adaptable to failures and load, as we will show in the following sections. On the flip side, reading and writing from S3 can be slower than writing to HDFS. However, most queries and processes tend to be multi-stage MapReduce jobs, where mappers in the first stage read input data in parallel from S3, and reducers in the last stage write output data back to S3. HDFS and local storage are used for all intermediate and transient data, which reduces the performance overhead.

**What is Genie?**

Genie is a set of REST-ful services for job and resource management in the Hadoop ecosystem. Two key services are the Execution Service, which provides a REST-ful API to submit and manage Hadoop, Hive and Pig jobs, and the Configuration Service, which is a repository of available Hadoop resources, along with the metadata required to connect to and run jobs on these resources.
