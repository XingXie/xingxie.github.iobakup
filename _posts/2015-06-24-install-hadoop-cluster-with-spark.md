---
layout: post
title:  "Install Hadoop Cluster with Spark Support"
date:   2015-06-25
categories: Installation
excerpt: hadoop
comments: true
---

* Hadoop: 2.6.0
* Spark: 1.3.1
* three nodes: 
* 10.0.0.1 name1.org
* 10.0.0.2 data1.org
* 10.0.0.3 data2.org

## Steps to Deploy Hadoop

0.Install java 8.

~~~ shell
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
$ sudo apt-get install oracle-java8-set-default
~~~

1.in the three machines, vi /etc/hostname 

in my case master is set to name1.org

the two slaves are set to data1.org and data2.org respectively

2.vi /etc/hosts

in the first line, change it as 

~~~ shell
127.0.0.1 localhost localhost.localdomain
~~~

and also add the following three lines

~~~~ shell
10.0.0.1 name1.org
10.0.0.2 data1.org
10.0.0.3 data2.org
~~~

3.create the user hduser, configure ssh between the three machines, in my case the home directory is /vol/hduser

if everything works fine, ssh between hduser in different machines should work

switch to hduser home directory, download the hadoop-2.6.0 file 

~~~ shell
wget http://apache.mirrors.tds.net/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
~~~

then run 

~~~ shell
tar -zxvf hadoop-2.6.0.tar.gz
~~~

4.go to hadoop-2.6.0/etc/hadoop, we need to config 7 files, in the three machines

4.1.hadoop-env.sh

~~~ shell
# The java implementation to use.
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
~~~

4.2.yarn-env.sh

~~~ shell
# some Java parameters
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
~~~

4.3.slaves 

~~~ shell
10.0.0.2 
10.0.0.3 
~~~

4.4.core-site.xml

~~~ xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://name1.org:9000/</value>
    </property>
    <property>
         <name>hadoop.tmp.dir</name>
         <value>file:/vol/hduser/hadoop-2.6.0/tmp</value>
    </property>
         <property>
          <name>fs.hdfs.impl</name>
          <value>org.apache.hadoop.hdfs.DistributedFileSystem</value>
     </property>
</configuration>
~~~

4.5.hdfs-site.xml

~~~ xml
<configuration>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>name1.org:9001</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/vol/hduser/hadoop-2.6.0/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/vol/hduser/hadoop-2.6.0/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
~~~
// be careful we use 2 replications here!

4.6.mapred-site.xml

~~~ xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
~~~

4.7.yarn-site.xml

~~~ xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>name1.org:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>name1.org:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>name1.org:8035</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>name1.org:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>name1.org:8088</value>
    </property>
</configuration>
~~~

5.in master node, in directory hadoop-2.6.0

~~~ shell
bin/hadoop namenode -format     
sbin/start-dfs.sh               
sbin/start-yarn.sh     
~~~

run jps command in three machines, 

~~~ shell
$ jps  #run on master
3407 SecondaryNameNode
3218 NameNode
3552 ResourceManager
3910 Jps

$ jps   #run on slaves
2072 NodeManager
2213 Jps
1962 DataNode         
~~~

you can also see hadoop management UI via http://name1.org:8088/

## Steps to Deploy SPARK
===============
1.We are using the same cluster to deploy spark. under sparkuser home directory, vi .bashrc

~~~ shell
export CLICOLOR=R1
export LSCOLORS=ExExCxDxBxegedabagacad
export PS1='$USER@$(hostname):$(basename $PWD)$ '
alias ll='ls -alh'
alias pjava='ps -ef | grep java'
alias dfs='$HADOOP_HOME/bin/hdfs dfs'
JAVA_HOME=/usr/lib/jvm/java-8-oracle
HADOOP_HOME=/vol/hduser/hadoop-2.6.0
SPARK_HOME=/vol/hduser/spark-1.3.1
~~~

then save and source .bashrc

2.in hduser home directory, download the spark-1.3.1 source code (can build several hadoop version)

~~~ shell
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.3.1.tgz
~~~

note that the link might be changed

3.
~~~ shell
tar zxvf spark-1.2.2.tgz. go to the spark-1.2.2 run dev/change-version-to-2.11.sh
~~~

4.run 

~~~ shell
sbt/sbt -Dscala-2.11=true -Pyarn -Phadoop-2.4 assembly (CRITICAL adding -Dscala-2.11=true) 
sbt/sbt -Dscala-2.11=true -Pyarn -Phadoop-2.4 -Phive -Phive-thriftserver assembly
~~~

5. in master server, edit conf/slave, add add the worker machine ips. edit conf/spark-env.sh, add

~~~ shell
export SPARK_MASTER_IP=10.0.0.1 (inner EC2 IP) 
export SPARK_SCALA_VERSION=2.11
~~~

in slave servers, edit conf/spark-env.sh, add

~~~ shell
export SPARK_SCALA_VERSION=2.11
~~~

6.in master machine, run 

~~~ shell
dfs -mkdir /spark_lib
~~~

go the diretory /vol/hduser/spark-1.3.1/assembly/target/scala-2.11,

run 

~~~ shell
dfs -copyFromLocal /spark-assembly-1.3.1-hadoop2.4.0.jar /spark_lib/
~~~

then in three machines, in spark-1.3.1/conf/spark-defaults.conf, add the following

~~~ shell
spark.yarn.jar hdfs://stg-newhdp-name1.fuhu.org:9000/spark_lib/spark-assembly-1.3.1-hadoop2.4.0.jar
~~~

7.in master spark-1.3.1/, sbin/start-all.sh. You can see process is running in master and slave machines.
When servers are up, 

~~~ shell
$ jps // master node
7949 Jps
7328 SecondaryNameNode
7805 Master
7137 NameNode
7475 ResourceManager

$jps // slave node
3132 DataNode
3759 Worker
3858 Jps
3231 NodeManager
~~~

you can check master spark ui
http://xxx.compute.amazonaws.com:8080/

you can do sbin/stop-all.sh to stop master and slaves. 

8.in master node, run a test case:

~~~ shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi 
--master yarn-cluster examples/target/scala-2.11/spark-examples*.jar 10
~~~

you can check the output in hadoop UI

### [References](http://wuchong.me/blog/2015/04/04/spark-on-yarn-cluster-deploy/)

