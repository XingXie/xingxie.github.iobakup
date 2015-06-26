---
layout: post
title:  "Install Flume 1.6 in Ubuntu"
date:   2015-06-25
categories: Installation
excerpt: Flume
comments: true
---

Ubuntu 14.04, apache flume: 1.6.0

## Steps to Deploy Flume

0.get a machine, a user flumeuser, under home directory /vol/flumeuser

1.switch to flumeuser home directory, download the flume-1.6.0 file 

~~~ shell
wget http://mirrors.sonic.net/apache/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz
~~~

then run 

~~~ shell
tar -zxvf apache-flume-1.6.0-bin.tar.gz
~~~
then 

~~~ shell
mv apache-flume-1.6.0-bin flume
~~~

2.in home directory, edit .bashrc

~~~ shell
export CLICOLOR=R1
export LSCOLORS=ExExCxDxBxegedabagacad
export PS1='$USER@$(hostname):$(basename $PWD)$ '
alias ll='ls -alh'
alias pjava='ps -ef | grep java'
export JAVA_HOME=/usr/lib/jvm/java-7-oracle
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

export FLUME_HOME=/vol/flumeuser/flume
export FLUME_CONF_DIR=$FLUME_HOME/conf
export PATH=.:$PATH::$FLUME_HOME/bin
~~~

and source it.

3.go to flume/conf, create a example.conf, add the following code 

~~~ shell
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
~~~

4.in flume folder, run 

~~~ shell
bin/flume-ng agent --conf conf --conf-file conf/example.conf --name a1 -Dflume.root.logger=INFO,console
~~~

you will see some logs print out

5.in different terminal, log in to the same server, try

~~~ shell
telnet localhost 44444
~~~

then type your words, you will see outputs from two terminals.

### write to HDFS

1. the original flume bin does not contain hadoop-core file. You must download these two to put in flume/lib, by wget
 jar files from 

~~~ shell
http://central.maven.org/maven2/commons-configuration/commons-configuration/1.10/commons-configuration-1.10.jar
http://central.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-client-core/2.6.0/hadoop-mapreduce-client-core-2.6.0.jar
http://central.maven.org/maven2/org/apache/hadoop/hadoop-common/2.6.0/hadoop-common-2.6.0.jar
http://central.maven.org/maven2/org/apache/hadoop/hadoop-hdfs/2.6.0/hadoop-hdfs-2.6.0.jar
http://central.maven.org/maven2/org/apache/hadoop/hadoop-auth/2.6.0/hadoop-auth-2.6.0.jar
https://repository.cloudera.com/artifactory/repo/org/apache/hadoop/hadoop-core/2.6.0-mr1-cdh5.4.0/hadoop-core-2.6.0-mr1-cdh5.4.0-sources.jar
http://central.maven.org/maven2/org/htrace/htrace-core/3.0.4/htrace-core-3.0.4.jar
~~~

* to support kafka, download the following also

~~~ shell
http://repository.cloudera.com/cloudera/cloudera-repos/org/apache/zookeeper/zookeeper/3.4.5-cdh5.4.2/zookeeper-3.4.5-cdh5.4.2.jar
~~~

2.then start the service: 

~~~ shell
bin/flume-ng agent --conf conf --conf-file conf/hdfsexample.conf --name a1 -Dflume.root.logger=INFO,console  
~~~

the example is monitoring the input directory and write any new files to HDFS in directory /vol/hduser/test/flumeoutput.

*hdfsexamples.conf*

~~~ shell
#the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type=spooldir
a1.sources.r1.spoolDir=/vol/flumeuser/input
a1.sources.r1.channels=c1
a1.sources.r1.fileHeader = false

# Describe the sink
a1.sinks.k1.type=hdfs
a1.sinks.k1.hdfs.path=hdfs://name1.org:9000/user/hduser/test/flumeoutput
a1.sinks.k1.hdfs.fileType=DataStream
a1.sinks.k1.hdfs.writeFormat=TEXT
a1.sinks.k1.hdfs.rollInterval=4
a1.sinks.k1.channel=c1
a1.sinks.k1.hdfs.batchSize=1

# Use a channel which buffers events in memory
a1.channels.c1.type=file
a1.channels.c1.checkpointDir=/vol/flumeuser/test/checkpoint
a1.channels.c1.dataDirs=/vol/flumeuser/test/data
~~~

### kafka to hdfs example

when type in things in kafka, the flume will get it and re-write to HDFS

in flume server: run

~~~ shell
bin/flume-ng agent --conf conf --conf-file conf/kdexample.conf --name a1 -Dflume.root.logger=INFO,console
~~~

in kafka server, run

~~~ shell
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic cloudera.test.kafka
~~~

*kdexample.conf*

~~~ shell
#the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type=org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.zookeeperConnect= kafka1.org:2181
a1.sources.r1.topic=cloudera.test.kafka
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = timestamp
a1.sources.r1.grounId = group4
a1.sources.r1.channels=c1
a1.sources.r1.kafka.consumer.timeout.ms = 100

# Describe the sink
a1.sinks.k1.type=hdfs
a1.sinks.k1.hdfs.path=hdfs://name1.org:9000/user/hduser/test/kafkaoutput
a1.sinks.k1.hdfs.fileType=DataStream
a1.sinks.k1.hdfs.writeFormat=TEXT
a1.sinks.k1.hdfs.rollInterval=4
a1.sinks.k1.channel=c1
a1.sinks.k1.hdfs.batchSize=1

# Use a channel which buffers events in memory
a1.channels.c1.type=file
a1.channels.c1.checkpointDir=/vol/flumeuser/test/checkpoint
a1.channels.c1.dataDirs=/vol/flumeuser/test/data
~~~
