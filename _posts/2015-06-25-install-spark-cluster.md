---
layout: post
title:  "Install Spark Cluster in Ubuntu Cluster"
date:   2015-06-25
categories: Installation
excerpt: Install Spark Cluster
comments: true
---


## Steps to Deploy SPARK

Ubuntu 14.04, Scala 2.11, spark 1.2.2 (for using cassandra connector).

1. Install java 8.

~~~ shell
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
$ sudo apt-get install oracle-java8-set-default
~~~

2. create the user, and ssh within the cluster machines.

* switch user su -l sparkuser, download the zip file 

~~~ shell
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.2.2.tgz
~~~ 

note that the link might be changed

3. create .bash_profile and source it after put the following lines:

~~~ shell
export CLICOLOR=R1
export LSCOLORS=ExExCxDxBxegedabagacad
export PS1='$USER@$(hostname):$(basename $PWD)$ '
alias ll='ls -alh'
alias pjava='ps -ef | grep java'
export SPARK_HOME='/vol/sparkuser/spark-1.2.2'
export JAVA_HOME='/usr/lib/jvm/java-8-oracle'
~~~

**SPARK_HOME** will be the place of the spark directory you install.

4. tar -zxvf spark-1.2.2.tgz. you might need to install git before running the following command. 
go to the spark-1.2.2, then run 

~~~ shell
sbt/sbt -Dscala-2.11=true -Pyarn -Phadoop-2.3 assembly (CRITICAL adding -Dscala-2.11=true) 
sbt/sbt -Dscala-2.11=true -Pyarn -Phadoop-2.3 -Phive -Phive-thriftserver assembly
~~~

[Build with SBT](http://people.apache.org/~pwendell/spark-1.2.2-rc1-docs/building-spark.html#building-with-sbt) 

5. in master server, edit conf/slave, add add the worker machine ips. edit conf/spark-env.sh, add

~~~ shell
export SPARK_MASTER_IP=10.22.4.83 (inner EC2 IP) 
export SPARK_SCALA_VERSION=2.11
~~~

edit conf/log4j.properties, change all INFO TO ERROR
and you need to put slave machine in slaves.sh

in slave servers, edit conf/spark-env.sh, add

~~~ shell
export SPARK_SCALA_VERSION=2.11
~~~

6. in master spark-1.2.2/, sbin/start-all.sh. You can see process is running in master and slave machines.
When servers are up, you can check ui
http://xxxx.compute.amazonaws.com:8080/

you can do sbin/stop-all.sh to stop master and slaves. 

7. upload your jar file from local to master and slave remote servers.
Script uploadToServer.sh (Prefer)

~~~ shell
!/bin/bash
# stage 3 servers (test)
CLUSTER_HOSTS="x1.compute.amazonaws.com x2.compute.amazonaws.com x3.compute.amazonaws.com"
for h in $CLUSTER_HOSTS; do
 scp SparkStatistics-assembly-1.0.jar xing.xie@$h:/raid0/sparkuser
done
# one spark server
scp SparkStatistics-assembly-1.0.jar xing.xie@x1.compute.amazonaws.com:/vol/sparkuser
~~~


## RUN SPARK JOBS

1. run on a single mode

In single node spark job, you can control the output report in your directory. 

..1. In sparkuser home directory, create directory report and debug

..2. You can manually run the daily and weekly jobs. You can see the sys out. stage/prod is upon your server choice.
  
~~~ shell  
  $SPARK_HOME/bin/spark-submit \
  --class "xxx.WeeklyStatistics" \
  	--master spark://10.22.4.83:7077 \
  SparkStatistics-assembly-1.0.jar stage
  
   $SPARK_HOME/bin/spark-submit \
    --class "xxx.DailyStatistics" \
	--master spark://10.22.4.83:7077 \
    SparkStatistics-assembly-1.0.jar stage
~~~

2. run on cluster mode

..1. The job will redirect to one of worker nodes. the report output is in work directory, 

~~~ shell
/vol/sparkuser/spark-1.2.2/work/driver-20150424010710-0004/report/WeeklyReport-2015-04-23T18:07:13.115-07:00.txt
~~~

You can use

~~~ shell
find / -type f -iname "Daily*" 
~~~

to find the reports in the worker server.

ui http://ec2-54-70-223-152.us-west-2.compute.amazonaws.com:8080/ can tell more about your job status.

..2. You can manually run the daily and weekly jobs. You CANNOT see the system out. stage/prod is upon your server choice.
	
~~~ shell	
	$SPARK_HOME/bin/spark-submit   --class "xxx.WeeklyStatistics"  \
	--master spark://10.0.0.0:7077 \
	--deploy-mode cluster \
	  SparkStatistics-assembly-1.0.jar stage
	  
  	$SPARK_HOME/bin/spark-submit   --class "xxx.DailyStatistics"  \
  	--master spark://10.0.0.0:7077 \
	--deploy-mode cluster \
  	  SparkStatistics-assembly-1.0.jar stage
~~~

3. Run using a crontab job, the scala-daily-stage.sh and scala-weekly-stage.sh are the files to 

~~~ shell
#!/bin/bash

# script to run spark
# requires JAVA_HOME and FUHU_HOME variables.
fi

if [[ -z "$JAVA_HOME" ]]; then
        echo "JAVA_HOME is not set.";
        exit 1;
fi

if [[ -z "$SPARK_HOME" ]]; then
        echo "JAVA_HOME is not set.";
        exit 1;
fi
(
  # Wait for lock on /var/lock/.scala.exclusivelock (fd 200) for 10 seconds
  flock -x -w 10 200 || exit 1
  # Do stuff
JVM_PARAMS="-Xms2g -Xmx4g"
EXECUTORMEM="4G"
MAIN="xxx.DailyStatistics"
MASTER="spark://10.0.0.0:7077"
SERVER="stage"
DATE="`date +%Y-%m-%dT%H:%M:%S`"
SCALAJAR="SparkStatistics-assembly-1.0.jar"
REPORTTYPE="Daily"
eval DATE="$DATE"
# --driver-memory $EXECUTORMEM is critical when run this job in standalone one spark node
$SPARK_HOME/bin/spark-submit  --class $MAIN --master $MASTER --driver-memory $EXECUTORMEM --executor-memory $EXECUTORMEM $SCALAJAR $SERVER > /vol/sparkuser/debug/$SERVER-$REPORTTYPE-$DATE.txt 2>&1 &
 ) 200>/var/lock/.scala.exclusivelock
 ~~~


In this case, you need to create /var/lock/.scala.exclusivelock. The following is the production crontab

~~~ shell
SPARK_HOME='/vol/sparkuser/spark-1.2.2'
JAVA_HOME=/usr/lib/jvm/java-8-oracle
FUHU_HOME=/vol/sparkuser/wings-spark
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# m h  dom mon dow   command
30 3 * * * /vol/sparkuser/scala-daily-prod.sh > /tmp/testsparkdailyprod.txt # txt is to check if your cronb job running status
30 5 * * * /vol/sparkuser/scala-weekly-prod.sh > /tmp/testsparkweeklyprod.txt # txt is to check if your cronb job running status
~~~
